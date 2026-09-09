# BLHeli_32 Technical Reference Guide

BLHeli_32 is a third-generation, 32-bit closed-source ESC firmware designed for ARM Cortex-M0/M4 (STM32F051, STM32G071, AT32F421) microcontrollers. It pioneered bidirectional DShot, variable PWM switching rates, integrated current/temperature telemetry, and programmable commutation timing.

---

## 1. Core Configuration Parameters & Engineering Context

### Motor Timing Advance
* **Parameter Range:** `Auto` or `1°` to `31°` (Default: `Auto` or `16°`).
* **Physics:** Advances the phase commutation angle ahead of the magnetic neutral point to compensate for stator coil current rise time ($L/R$ time constant).
* **Tuning Recommendations:**
  * **Auto:** Dynamic algorithm adjusts timing based on estimated load. While efficient in steady state, aggressive throttle steps (0% to 100% punches) can outpace the algorithm, causing desyncs.
  * **Static 22°–23°:** The gold standard for 5" freestyle and racing builds. Completely eliminates tracking lag during violent punchouts.
  * **Static 15°–18°:** Essential for large, high-inductance stators (2806.5 to 3115 on 7"–10" builds) to avoid magnetic core saturation.

### Demagnetization Compensation (Demag)
* **Parameter Range:** `Off`, `Low`, `High`.
* **Physics:** When a phase is turned off, its magnetic field collapses, driving inductive flyback current. Demag compensation delays energizing the subsequent phase until this current decays below a safe threshold.
* **Tuning Recommendations:**
  * **High:** Required for motors prone to desyncs (e.g. ducted cinewhoops, aggressive pitch propellers, heavy payloads).
  * **Low:** Standard factory default providing balanced protection.
  * **Off / Low (Racing Max Power):** Recovers ~**7,000 RPM (15–20% power output)** on 6S high-Kv (2100–2150Kv) racing motors by eliminating artificial phase switching delays.

### PWM Switching Frequency & Variable PWM
* **Range:** `16kHz` to `128kHz` (or Low/High variable bounds, e.g. `24kHz–48kHz`).
* **Variable PWM Behavior:** Continuously increases switching frequency proportionally with throttle stick position.
  * *Low Throttle:* Switches at 24kHz for crisp low-end handling and maximum active braking authority.
  * *High Throttle:* Steps up to 48kHz or 96kHz for smoother top-end response and reduced motor stator heating.

### Rampup Power & Acceleration Limits
* **Parameter Range:** `10%` to `150%` (Default: `50%` or `100%`).
* **Physics:** Restricts maximum instantaneous current slew rate during throttle transients.
* **Tuning Recommendation:** Lowering rampup power to **`30%–50%`** prevents over-current tripping and mechanical slip on heavy propellers.

---

## 2. Converting & Unlocking BLHeli_32 Boards to Open-Source (AM32 / ESCape32)

Following the cessation of BLHeli_32 licensing and development, hardware can be converted to open-source [AM32](am32.md) or [ESCape32](escape32.md):
1. **Hardware Interface:** Solder wires to SWD test pads (`SWDIO`, `SWCLK`, `GND`) on the ESC PCB.
2. **Unlock Chip:** Connect an ST-Link v2 / DAPLink programmer and clear Readout Protection (RDP Level 1 -> Level 0) via `STM32_Programmer_CLI -c port=SWD mode=UR -rdu`.
3. **Flash Bootloader & Firmware:** Flash the corresponding AM32/ESCape32 target hex matching the gate driver and pin mapping. Full details in [ESC Flashing & Unbricking Guide](flashing-unbricking-hardware.md).

---

## 3. Curated Video & Audio Resources

* **Joshua Bardwell & Ryan Harrell BLHeli_32 Deep Series:** [YouTube Playlist](https://www.youtube.com/playlist?list=PLwoDb7WF6c8kXOyPdBog1wtRcxnXMasUb)
* **Bardwell / Harrell Desync & Demag Deep Dive (@21:17):** [YouTube Video](https://youtu.be/oKcyXR7Yx64)
* **Pawel Spychalski Demag-Compensation Technical Explanation:** [YouTube Video](https://youtu.be/c94e9TCCP8Y)
* **Joshua Bardwell Variable PWM Testing & Benchmarks:** [YouTube Video](https://youtu.be/xuQeJA4EGr8)
* **Chris Rosser BLHeli_32 Performance & Tuning (2021):** [YouTube Video](https://youtu.be/7WeHTb7aBrE)
* **Chris Rosser BLHeli_32 & State of ESCs (2024):** [YouTube Video](https://youtu.be/6gv0_jTEYZM)

---

*Related Knowledge Base Modules:*
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [Theory of Operation & Hardware](theory-operation-hardware.md)
* [PWM Switching & Thermal Dynamics](pwm-frequency-heat.md)
* [ESC Flashing & Unbricking Guide](flashing-unbricking-hardware.md)
* [Back to Knowledge Base Index](README.md)
