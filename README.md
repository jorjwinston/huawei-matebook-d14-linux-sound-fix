Fixing Everest ES8336 Audio on Huawei MateBook (AMD Ryzen) under Linux
This repository provides a comprehensive, professional guide and the required kernel patches to fix the notorious Everest Semiconductor ES8336 (ESSX8336) audio issues on AMD Ryzen-based laptops (specifically tested on HUAWEI MateBook NBM-WXX9 with AMD Ryzen 7 5700U under Arch Linux / CachyOS, but applicable to any modern Linux distribution).

Repository Structure
huawei-matebook-d14-linux-sound-fix/
├── README.md
├── LICENSE
└── patches/
    ├── acp-config.patch
    └── acp3x-es83xx.patch


The Problem
Many AMD-based laptops using the Everest ES8336 codec suffer from one or more of the following issues on Linux:
1.	Dummy Output (No Sound Card Detected): The kernel audio coprocessor driver loads, but fails to instantiate or register the ALSA sound card.
2.	Media Stream Hang (00:00 Lockup): The media player freezes at 00:00 because of clock mismatches (I2S BCLK/LRCK sync errors).
3.	No Sound / Output Mute: The driver loads successfully, but no physical sound comes out of the speakers/headphones due to incorrect GPIO configurations or hidden ALSA mixer gain levels.

Root Cause Analysis
Our deep debugging sessions revealed the following roots of the issue:
1.	DMI Table Mismatch: The default AMD ACP (Audio Coprocessor) config driver lacks matching rules for newer Huawei MateBook models. As a result, the legacy machine driver is bypassed or fails to load.
2.	Clock and Master/Slave Desync: Forcing soc_mclk = true or SND_SOC_DAIFMT_CBC_CFC (CPU Master / Codec Slave) breaks the DMA stream, causing the audio system to freeze. The codec must run as Codec Master / Frame Provider (SND_SOC_DAIFMT_CBP_CFP) with a physical 48 MHz Master Clock (ES83XX_48_MHZ_MCLK quirk).
3.	Amplifier Power Polarity: The onboard speaker amplifier is configured as Active High (active_low = false). Setting it to Active Low completely mutes the physical outputs.
4.	ALSA Volume Scale Bug: Using generic volume adjustment commands like amixer -c 1 sset 'Headphone Mixer' 70% forces the volume down to 1 (on a scale of 0-11), causing total silence that mimics a hardware or driver failure.

Step-by-Step Fix Guide
Step 1: Clone your Kernel Source
Make sure you have cloned your active kernel source repository (e.g., CachyOS kernel source or Arch Linux kernel source).
Step 2: Apply the patches
Apply the patches included in this repository to your kernel sound subsystem:
# Navigate to your kernel source directory
cd /path/to/your/kernel/source/

# Apply configuration and driver patches
git apply /path/to/huawei-matebook-d14-linux-sound-fix/patches/acp-config.patch
git apply /path/to/huawei-matebook-d14-linux-sound-fix/patches/acp3x-es83xx.patch


Step 3: Compiling and Installing Modules
Once patched, compile the sound drivers for your active kernel (using LLVM/Clang or GCC toolchains):
# Compile the ALSA AMD SoC drivers
make -C /lib/modules/$(uname -r)/build M=$(pwd)/sound/soc/amd LLVM=1 modules

# Compress the newly compiled legacy machine card driver
zstd -f sound/soc/amd/acp/snd-acp-legacy-mach.ko

# Copy files to the active system modules directory
sudo cp sound/soc/amd/acp/snd-acp-legacy-mach.ko.zst /lib/modules/$(uname -r)/kernel/sound/soc/amd/acp/

# Update module dependencies
sudo depmod -a

# Rebuild the system initramfs boot images (crucial for loading drivers early at boot)
sudo mkinitcpio -P   # On Arch/CachyOS (Or 'sudo update-initramfs -u -k all' on Ubuntu/Debian)

# Reboot your laptop
reboot


Step 4: Crucial ALSA Mixer Adjustments
Even with correct drivers loaded, you must adjust the ALSA mixer levels. Avoid setting percentage-based values on small volume ranges:
# 1. Force Headphone Mixer volume to maximum (11 out of 11)
amixer -c 1 sset 'Headphone Mixer' 11

# 2. Make sure the Headphone switch is ON
amixer -c 1 sset 'Headphone' on

# 3. Disable double-sampling frequency mismatch tweak (if active)
amixer -c 1 sset 'DAC Double Fs' off


Step 5: WirePlumber/PipeWire Setup
To prevent any runtime audio glitches or bad bit-shifting on the ES8336 codec, configure WirePlumber to force a safe format (16-bit, 48000Hz) for your node:
Create the file ~/.config/wireplumber/wireplumber.conf.d/51-alsa-es8316.conf:
monitor.alsa.rules = [
  {
    matches = [
      {
        node.name = "~alsa_input.pci.*es83xx.*"
      },
      {
        node.name = "~alsa_output.pci.*es83xx.*"
      }
    ]
    actions = {
      update-props = {
        audio.format = "S16LE"
        audio.rate = 48000
        api.alsa.period-size = 1024
        api.alsa.headroom = 1024
        session.suspend-timeout-seconds = 0
      }
    }
  }
]

Restart audio services:
systemctl --user restart pipewire.service wireplumber.service

Test your output:
speaker-test -c 2 -l 1


Role of Google Antigravity CLI (AGY)
This solution was researched, developed, and validated interactively using Google Antigravity CLI (AGY).
AGY acted as an autonomous Linux kernel debugger, enabling:
·	Real-time ACPI table parsing and DMI data extraction.
·	Automatic patching of kernel files with LLVM/Clang modules validation.
·	Deep analysis of past communication logs to trace sound cancellation bugs.
·	Seamless diagnostic testing alongside the user.
Special thanks to the Google DeepMind team and the AGY CLI engine!
