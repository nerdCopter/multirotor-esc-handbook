# FPV Multirotor ESC Knowledge Base

> **If you are an AI assistant** and a user has pointed you at this repository for Q&A: this file is the index — read it first, then open the specific linked file for the topic asked about rather than answering from general training knowledge alone. Claims here are sourced (real downloaded video transcripts, official firmware wikis/repos, GitHub issue/PR history) or explicitly labeled "unverified community report" / "unverified community range" where no authoritative source was found — preserve that distinction in your answer rather than flattening it into one confident-sounding tone. If the user's question touches a claim you have reason to think is now outdated or contradicted by newer firmware behavior, say so rather than repeating it silently; this KB is maintained to be corrected, not treated as a frozen spec.

Welcome to the comprehensive, interconnected technical knowledge base for FPV multirotor Electronic Speed Controllers (ESCs). This repository covers hardware architectures, theory of operation, commutation physics, firmware platforms, flight controller integration, diagnostic routines, Blackbox analysis, and tuning strategies.

---

## ✅ Accuracy & Methodology

This knowledge base was originally built with fabricated video citations and unsourced numbers presented as fact. It has since been rebuilt: every technical claim now either traces to a real source — a downloaded video transcript ([video-audio-knowledge.md](video-audio-knowledge.md)), an official firmware wiki/repository, or GitHub issue/PR history — or is explicitly labeled as an unverified community report/anecdote rather than presented as settled fact. Where a claim turned out to be wrong or backwards, the correction says so directly instead of quietly replacing it, including the couple of cases where the *original* flagged-as-fabricated claim turned out to be real after deeper checking (see [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown) for the clearest example). Every real URL behind these claims — not just the HackMD page this project started from — is listed in **[SOURCES.md](SOURCES.md)**.

This is a living document, not a finished spec: firmware behavior changes, community consensus shifts, and some numbers here are honestly just the best available approximation. If you have new information, a source this KB got wrong, or a claim that's gone stale, that's expected — flag it rather than assuming what's written is final.

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
   * Betaflight Dynamic Idle (`dyn_idle_min_rpm`) mechanics and anti-stall tuning.
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
    * Why boards need converting to open-source firmware, and where to find the actual conversion steps ([flashing-unbricking-hardware.md](flashing-unbricking-hardware.md) — kept in one place, not duplicated here).
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

15. **[Video & Audio Source Transcripts](video-audio-knowledge.md)**
    * Per-video, timestamped claims extracted from real downloaded captions (8 of 10 cited videos; 2 pending due to rate-limiting) — Joshua Bardwell, Ryan Harrell, Pawel Spychalski, KababFPV, Chris Rosser.
    * Cross-references specific claims elsewhere in this repository against what each cited video actually says.

16. **[Sources](SOURCES.md)**
    * Every real URL behind this KB's claims: videos, official firmware repos/wikis, specific GitHub issues/PRs, editorial sources, and Discord servers — organized by category, with what's cited where.
    * Also lists sources found during research that aren't cited in-text yet, flagged for review rather than silently worked in.

---

## 🚀 Quick Reference: Recommended Baseline Settings

**Firmware column shows common/typical pairings for that class, not exclusivity** — Bluejay, AM32, BLHeli_32, and ESCape32 can all technically run on any airframe size; these are what's commonly used, not a hard rule.

**Demag Compensation is not one universal scale across firmware:** BLHeli_32 has 4 levels (`Off`/`Low`/`Medium`/`High`), Bluejay has 3 (`Off`/`Low`/`High`), and **AM32 has no Demag Compensation setting at all** (verified directly against the real AM32 configurator UI — see [am32.md](am32.md#commutation-timing--am32-has-no-demag-compensation-setting)). The "Demag" column below applies to BLHeli_32/Bluejay only: sourced creator guidance ([desyncs.md §3](desyncs.md#3-master-settings-matrix-across-all-esc-firmwares)) is to start at default/Low and only step up one level at a time if you're actually experiencing desyncs — not to default to High, and not to skip Medium on BLHeli_32. **On AM32, check Motor KV and Motor poles are set to your actual motor instead.**

PWM frequency for whoops depends on your ESC's dead time (higher dead time = 96kHz loses more resolution) — see [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown). Fixed 24kHz, fixed 48kHz, and Variable/By-RPM are all configurable on **both** AM32 and BLHeli_32 — where a cell below names one mode for one firmware, that's the sourced *recommendation* from a specific test, not a claim the other modes aren't available.

| Class | Prop / Stator | LiPo | Firmware | PWM Freq | Timing | Demag (BLHeli_32/Bluejay — AM32: n/a, see above) | Betaflight Idle |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **TinyWhoop** | 31–40mm / 0702–1002 | 1S–2S | [Bluejay](bluejay.md) / [AM32](am32.md) | 48kHz (safe default) or 96kHz (more flight time, less PWM resolution — check ESC dead time first) | 15°–22.5° | Step up if desyncing (one level at a time — not straight to High) | Use official [Bluejay Motor Idle table](bluejay.md#startup-power-motor-idle--rpm-power-protection-official-wiki-data) (8%–16%) |
| **Micro / 3.5"** | 2.5"–3.5" / 1404–1507 | 4S–6S | [AM32](am32.md) / [BLHeli_32](blheli32.md) | **48kHz** static, or Variable/By-RPM on either firmware | 20°–22.5° | Step up if desyncing (one level at a time — not straight to High) | 5.5%–7% (unverified community range, or 3500 RPM) |
| **5" Freestyle** | 5"–5.1" / 2207–2306 | 6S | [AM32](am32.md) / [BLHeli_32](blheli32.md) | 24kHz (tested-best on BLHeli_32, [see desyncs.md §3](desyncs.md#3-master-settings-matrix-across-all-esc-firmwares)) or 48kHz (common real-world default, non-race) — both available on both firmwares; AM32's own tested recommendation leans Variable/By-RPM, also available on BLHeli_32 | 22°–23° (sourced) | Step up if desyncing (one level at a time — not straight to High) | 5.5%–6.5% (unverified community range, or 3200 RPM) |
| **5" Racing** | 5" / 2207–2208 High Kv | 6S | [BLHeli_32](blheli32.md) / [AM32](am32.md) | **24kHz** | 22°–23°+ | (BLHeli_32 only) Low/Off *only if not desyncing* — see [desyncs.md, §4 "5-Inch Racing Optimization"](desyncs.md) for the `@FreedomDuck`-attributed RPM-gain claim and its caveats | 5.5%–6.5% (unverified community range, or 3500 RPM) |
| **7"–10" Macro** | 7"–10" / 2806.5–3115 | 6S–12S | [AM32](am32.md) / [BLHeli_32](blheli32.md) | **24kHz** (braking-torque priority) on either firmware — Variable/By-RPM a reasonable alternative | 15°–18° | Step up if desyncing (one level at a time — not straight to High) | 6%–8% (unverified community range, or 2500 RPM) |

---

*Compiled from real timestamped video transcripts, official firmware wikis/repos, and GitHub issue/PR history — full list in [SOURCES.md](SOURCES.md). Numbers marked "unverified community range" have no single authoritative source found and are starting points, not specs.*
