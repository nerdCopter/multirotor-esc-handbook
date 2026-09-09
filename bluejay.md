# Bluejay (BLHeli_S 8-bit) Technical Reference Guide

Bluejay is an open-source firmware designed for 8-bit Silabs EFM8BB21 and EFM8BB10 microcontrollers (historically running BLHeli_S). It modernizes legacy 8-bit hardware by adding **Bidirectional DShot** (e-RPM telemetry), variable PWM frequencies, customizable startup melodies, and optimized commutation routines.

---

## 1. Web Configurator & Official Resources

* **Official Repository:** [GitHub bird-sanctuary/bluejay](https://github.com/bird-sanctuary/bluejay/releases)
* **Web Flasher & Configurator:** [ESC Configurator (esc-configurator.com)](https://esc-configurator.com/)
* **Official Wiki / Setup Guide:** [Bluejay Setup Wiki](https://github.com/bird-sanctuary/bluejay/wiki/Setup)
* **BLHeli_S Migration Guide:** [Migrating from BLHeli_S](https://github.com/bird-sanctuary/bluejay/wiki/Migrating-from-BLHeli_S)
* **Community Discord:** [Bluejay Discord Server](https://discord.gg/ddyzguPB5t)

---

## 2. Core Technical Parameters & Tuning

### PWM Switching Frequency Breakdown
* **24kHz:**
  * **Characteristics:** Maximum active braking authority (damped light), highest low-end torque, and crispest control response.
  * **Trade-off:** Shorter flight times; audible switching whine.
* **48kHz (Official Developer Recommendation):**
  * **Characteristics:** Ideal balance of flight efficiency (+3% to +6% flight time over 24kHz), quiet operation, and robust zero-crossing back-EMF detection. Recommended for all 1S–4S quads and brushless TinyWhoops.
* **96kHz (Deprecated / High Desync Risk):**
  * **Characteristics:** Longest hover flight time, but drastically narrows the back-EMF zero-crossing sampling window.
  * **Caveat:** Highly prone to startup stalls, low-throttle desyncs, and MOSFET thermal runaway on high-capacitance gates. Bluejay maintainers officially recommend 48kHz and have considered removing 96kHz.

### Startup Power: Min & Max Power Mechanics
In Bluejay, startup power controls how much current is injected into the stator coils during open-loop initial alignment and rampup before stable Back-EMF is acquired:
* **Golden Rule:** **Never set startup powers to maximum.** Maxing out values saturates small stator cores, causes severe motor cogging, and overheats MOSFETs.
* **Recommended Settings for TinyWhoops & Micros (1S–2S, 0702 to 1202.5):**
  * `Startup Min Power`: `1100` to `1200`
  * `Startup Max Power`: `1200` to `1250`
* **Recommended Settings for 5" Quads (4S–6S, 2207/2306):**
  * `Startup Min Power`: `1050` to `1150`
  * `Startup Max Power`: `1150` to `1250`

### Whoop Idle & Commutation Stability
Due to tiny rotor inertia and sub-millivolt BEMF generation on 0702–1002 high-Kv motors (18,000Kv–30,000Kv):
* Configure Betaflight digital idle to **9%–14%** (or Dynamic Idle target to `4500–6500 RPM`) to maintain zero-crossing lock during inverted zero-throttle maneuvers.

---

## 3. Bidirectional DShot Implementation on 8-bit Hardware

* Bluejay implements DShot telemetry over the motor signal line using an inverted e-RPM pulse return immediately following the 16-bit DShot command.
* **Best Practice:** Run **DShot300 at a 4.0kHz PID loop** in Betaflight. Running DShot600 on 8-bit EFM8BB21 microcontrollers can saturate MCU interrupt servicing and increase DShot packet error rates.

---

## 4. Curated Video & Community Resources

* **Joshua Bardwell Bluejay Flashing & Setup Guide:** [YouTube Video](https://youtu.be/yEDhnBUFQNI)
* **Chris Rosser Bluejay Optimization & Analysis (2024):** [YouTube Video](https://youtu.be/EhYKeZfSQIw)
* **KababFPV Micro Rampup & Startup Power Mechanics:** [YouTube Video](https://youtu.be/obObn1oGWmU) *(See technical breakdown in comments by Mr.ShutterBug)*

---

*Related Knowledge Base Modules:*
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [Protocols & Telemetry](protocols-telemetry.md)
* [PWM Switching & Thermal Dynamics](pwm-frequency-heat.md)
* [Betaflight ESC Integration](betaflight-esc-tuning.md)
* [Back to Knowledge Base Index](README.md)
