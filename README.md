# FPV Multirotor ESC Knowledge Base

Welcome to the comprehensive, interconnected technical knowledge base for FPV multirotor Electronic Speed Controllers (ESCs). This repository covers hardware architectures, theory of operation, commutation physics, firmware platforms, flight controller integration, diagnostic routines, Blackbox analysis, and tuning strategies.

---

## 📚 Table of Contents & Knowledge Modules

1. **[Desyncs & Commutation Troubleshooting](desyncs.md)**
   * Single master guide on motor desynchronization root causes, physics, and diagnostics.
   * Cross-firmware settings matrix: Motor Timing, Demag Compensation, PWM Frequency, Rampup Power, Dynamic Idle.
   * Sizing rules for 1S Whoops, 3"–4" Micros, 5" Freestyle/Racing, and 7"–10" Macroquads.
   * Hardware-induced desync analysis (shorted screws, cracked pads, cold solder joints, missing low-ESR caps).

2. **[Theory of Operation & Hardware Architecture](theory-operation-hardware.md)**
   * Inverter bridge anatomy (6 N-FETs, half-bridges, high-side charge pumps, current shunt resistors).
   * Sensorless 6-step trapezoidal commutation and Back-EMF Zero-Crossing Detection (ZCD).
   * Active Braking (Damped Light) physics and power filtering (Low-ESR capacitors & TVS diodes).

3. **[Field Oriented Control (FOC) vs 6-Step Trapezoidal Commutation](foc-vs-trapezoidal.md)**
   * Theoretical and architectural differences: Block commutation vs vector space modulation.
   * Why trapezoidal commutation dominates multirotor flight (transient acceleration & low latency).
   * Hybrid approaches: Sinusoidal startup (FOC-lite) in AM32.

4. **[Communication Protocols, DShot & Telemetry](protocols-telemetry.md)**
   * Analog protocols (PWM, OneShot, MultiShot) vs digital DShot (DShot150 to DShot1200).
   * DShot 16-bit packet structure, CRC verification, and special command codes.
   * Bidirectional DShot, EDT (Extended DShot Telemetry), and dynamic RPM filtering.

5. **[Flight Controller & ESC Integration (Betaflight Tuning)](betaflight-esc-tuning.md)**
   * FC loop frequency synchronization with DShot rates (4kHz / 8kHz).
   * Betaflight Dynamic Idle (`idle_min_rpm`) mechanics and anti-stall tuning.
   * Motor direction configuration (Props-In vs Props-Out) and DShot lost-model beeper.

6. **[ESC Diagnostics & Blackbox Log Analysis](blackbox-esc-diagnostics.md)**
   * Betaflight Blackbox configuration for logging ESC/DShot telemetry at 2kHz/4kHz rates.
   * Visual anatomy and trace signatures of desync death rolls vs gyro noise vs D-term oscillations.
   * Diagnosing DShot packet error rates (`dshot_err`) and electrical ground bounce.

7. **[PWM Switching Frequency, Braking & Thermal Dynamics](pwm-frequency-heat.md)**
   * Low (24kHz) vs High (48kHz / 96kHz / 128kHz) performance, torque, and braking trade-offs.
   * MOSFET switching losses ($P_{sw}$), thermal runaway risks, and motor stator heating dynamics.
   * Dynamic PWM strategies (Fixed, Variable by Throttle, and Variable "By RPM").

8. **[Advanced ESC Features: 3D Flight, Turtle Mode & Custom Tones](advanced-esc-features.md)**
   * Flip Over After Crash (Turtle Mode) mechanics and current protection.
   * 3D aerobatics bidirectional motor control and power bus flyback dynamics.
   * RTTTL custom startup tone editing and hardware thermal throttling.

9. **[ESC Firmware Comparison Matrix](firmware-comparison.md)**
   * 8-bit vs 32-bit architectural breakdown (ARM Cortex-M0/M4, Silabs EFM8BB21, RISC-V).
   * Cross-comparison across BLHeli_32, Bluejay, AM32, and ESCape32.

10. **[BLHeli_32 Technical Reference Guide](blheli32.md)**
    * Parameter tuning (Timing, Demag, Variable PWM, Rampup Power).
    * ST-Link conversion workflows for unlocking boards to run open-source firmware.
    * Curated video and tutorial references.

11. **[Bluejay (BLHeli_S 8-bit) Technical Guide](bluejay.md)**
    * Web configurator setup ([esc-configurator.com](https://esc-configurator.com/)) and migration from legacy BLHeli_S.
    * Bidirectional DShot on 8-bit hardware, Startup Min/Max Power, and Whoop idle configuration.

12. **[AM32 32-Bit Multi-Rotor Guide](am32.md)**
    * Open-source ARM/RISC-V firmware architecture and features.
    * Dynamic PWM "By RPM", Sinusoidal Startup (FOC-lite), and stall protection.
    * Web configurator ([am32.ca](https://am32.ca/)) and wiki reference links.

13. **[ESCape32 Technical Guide](escape32.md)**
    * Modern deterministic 32-bit architecture for STM32 and AT32 microcontrollers.
    * Commutation timing algorithms, bidirectional telemetry, and firmware targets.

14. **[ESC Flashing, ST-Link Unbricking & Hardware Modding](flashing-unbricking-hardware.md)**
    * FC passthrough flashing vs direct SWD flashing.
    * Unlocking Readout Protection (RDP Level 1 -> 0) with STM32_Programmer_CLI and OpenOCD.
    * Hardware target mapping (gate drivers, current sense shunts, phase sense comparators).

15. **[Video, Audio & Community Source Knowledge Transcripts](video-audio-knowledge.md)**
    * Deep technical extractions from Joshua Bardwell, Ryan Harrell, Pawel Spychalski, KababFPV, Chris Rosser, and High Energy Failures.

---

## 🚀 Quick Reference: Recommended Baseline Settings

| Class | Prop / Stator | LiPo | Firmware | PWM Freq | Timing | Demag | Betaflight Idle |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **TinyWhoop** | 31–40mm / 0702–1002 | 1S–2S | [Bluejay](bluejay.md) / [AM32](am32.md) | **48kHz** | Auto / Med-High | High | 9%–14% (or 4500+ RPM) |
| **Micro / 3.5"** | 2.5"–3.5" / 1404–1507 | 4S–6S | [AM32](am32.md) / [BLHeli_32](blheli32.md) | **48kHz** / Variable | 20°–22.5° | High | 5.5%–7% (or 3500 RPM) |
| **5" Freestyle** | 5"–5.1" / 2207–2306 | 6S | [AM32](am32.md) / [BLHeli_32](blheli32.md) | **24kHz–48kHz** | 22°–23° | High | 5.5%–6.5% (or 3200 RPM) |
| **5" Racing** | 5" / 2207–2208 High Kv | 6S | [BLHeli_32](blheli32.md) / [AM32](am32.md) | **24kHz** | 23°–26° | Low / Off | 5.5%–6.5% (or 3500 RPM) |
| **7"–10" Macro** | 7"–10" / 2806.5–3115 | 6S–12S | [AM32](am32.md) / [BLHeli_32](blheli32.md) | **24kHz** | 15°–18° | High | 6%–8% (or 2500 RPM) |

---

*Compiled from [HackMD ESC Resources](https://hackmd.io/6meEOax2T-KuzpujxHHSMw) and community engineering sources.*
