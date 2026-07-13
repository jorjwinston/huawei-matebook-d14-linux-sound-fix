# Fixing Everest ES8336 Audio on Huawei MateBook (AMD Ryzen) under Linux

This repository provides a complete guide and kernel patches to fix the ES8336 (ESSX8336) audio codec on Huawei MateBook laptops with AMD Ryzen processors running Linux.

---

## Why this exists

I spent almost **4 months** trying to fix the audio on my Huawei MateBook under Linux.

After many failed attempts, I finally identified the real kernel-level issues and created this repository so others won't have to repeat the same journey.

---

## Acknowledgements

I don't consider myself a Linux kernel developer.

The kernel patches in this repository were developed with extensive help from AI assistants, especially **Google Antigravity CLI** and **ChatGPT**.

My role was to investigate the hardware, collect logs, test ideas, verify results, and iterate until the correct solution was found.

**AI accelerated the development, but every change was validated on real hardware.**

---

## Tested Hardware

- Laptop: Huawei MateBook D14 (NBM-WXX9)
- CPU: AMD Ryzen 7 5700U
- Audio Codec: Everest ES8336 (ESSX8336)
- Distribution: CachyOS (Arch Linux)

---

## The Problem

Many AMD-based laptops using the ES8336 codec suffer from one or more of these issues:

- Dummy Output
- No Sound
- Media freezes at 00:00
- Speakers not detected
- Static noise
- Broken machine driver detection

---

## Root Cause Analysis

The real causes were:

- Missing DMI entry in the AMD ACP configuration driver
- Incorrect machine driver selection
- Wrong MCLK configuration
- Incorrect DAI format
- GPIO amplifier configuration
- ALSA mixer volume scaling

---

## Solution

This repository includes:

- Kernel patches
- Build instructions
- Installation guide
- ALSA mixer configuration
- PipeWire/WirePlumber configuration
- Troubleshooting notes

---

## Repository Structure

```
patches/
README.md
screenshots/
```

---

## Results

✅ Internal speakers working

✅ Headphone detection working

✅ ES8336 codec initialized correctly

✅ PipeWire detects the sound card

✅ No more Dummy Output

---

## License

MIT License
