# ESC Firmware Comparison Matrix

This document provides an architectural and functional comparison between all modern multirotor ESC firmwares: **BLHeli_32**, **Bluejay**, **AM32**, and **ESCape32**.

---

## 1. High-Level Comparison Table

| Feature / Metric | [BLHeli_32](blheli32.md) | [Bluejay](bluejay.md) | [AM32](am32.md) | [ESCape32](escape32.md) |
| :--- | :--- | :--- | :--- | :--- |
| **Architecture** | 32-bit (ARM Cortex-M0/M4) | 8-bit (Silabs EFM8BB21) | 32-bit (ARM / RISC-V) | 32-bit (STM32 / AT32) |
| **Project Status** | Closed Source (Discontinued) | Open Source (Active) | Open Source (Active) | Open Source (Active) |
| **Primary Target** | 5" Freestyle/Race, Macroquads | TinyWhoops, Micros, Budget 5" | 5" Freestyle/Race, Whoops, Crawlers | High Performance 5", Micros |
| **Bi-directional DShot** | Yes (DShot300/600/1200) | Yes (DShot300/600) | Yes (DShot300/600/1200) | Yes (DShot300/600/1200) |
| **PWM Switching Rates** | 16kHz to 128kHz | 24kHz, 48kHz, 96kHz | 16kHz to 128kHz | Configurable (typically 24kHz–48kHz; exact ceiling is target-dependent) |
| **Dynamic PWM Modes** | Variable by Throttle | Static per flash | Throttle & **By RPM** | Variable by Throttle |
| **Sinusoidal / FOC Mode** | Sine Start (Limited) | No | Yes (Sinusoidal Start) | Yes (Sine Spoolup) |
| **Configurator Interface** | BLHeli32 Suite (App / Chrome) | [ESC Configurator (Web)](https://esc-configurator.com/) | [AM32 Web App](https://am32.ca/) | ESCape32 Configurator / CLI |

---

## 2. Choosing the Right Firmware

* **For 1S/2S TinyWhoops:** [Bluejay](bluejay.md) at **48kHz** (or [AM32](am32.md) if board has 32-bit MCU).
* **For 5" Freestyle & Racing:** [AM32](am32.md) or [BLHeli_32](blheli32.md) with static timing (22°–23°) and PWM at 24kHz–48kHz.
* **For 7"–10"+ Long Range & Heavy Lifters:** [AM32](am32.md) / [BLHeli_32](blheli32.md) with low timing (15°–18°) and 24kHz PWM. On BLHeli_32, step Demag toward High if experiencing desyncs — AM32 has no Demag Compensation setting; verify Motor KV/Motor poles instead.
* **For Micro Crawlers & Low-Speed Robotics:** [AM32](am32.md) with Sinusoidal Startup enabled.

---

## 3. Independent Bench-Test Findings (Firmware-Level, Hardware Held Constant)

Testing that held motor/prop/PCB hardware constant and varied only firmware found real firmware-level performance differences, distinct from ordinary board-to-board hardware variance:

* **Torque-curve shape:** on identical PCB hardware, BLHeli_32 firmware produced a torque curve peaking later (higher RPM) and higher in magnitude than AM32 firmware on the same board. Sourced (Chris Rosser, [pH4K3ErugW4](https://youtu.be/pH4K3ErugW4), [mdM4jEWOqx4](https://youtu.be/mdM4jEWOqx4)).
* **Low-throttle efficiency:** BLHeli_32 measured less efficient than AM32 at low throttle in the same test batch, but comparable to or better than AM32 at full throttle, where efficiency is dominated by FET on-resistance rather than firmware. Sourced (Chris Rosser, [mdM4jEWOqx4](https://youtu.be/mdM4jEWOqx4)).
* **Step-response latency:** in one bench comparison, a BLHeli_32 unit reached 90% of a commanded throttle step slower than the AM32 and Bluejay units in the same batch (on the order of 10ms slower). Sourced (Chris Rosser, [QENbsI3swCI](https://youtu.be/QENbsI3swCI)) — see [foc-vs-trapezoidal.md §4](foc-vs-trapezoidal.md) for the same test batch's FETtec SFOC prototype comparison.
* **Current rating alone is not a reliable predictor of thermal resilience** — PCB copper thickness and the presence of an aluminum heat spreader across the MOSFETs correlated better with time-to-overheat than the ESC's advertised amp rating, across boards from multiple manufacturers. Sourced (Chris Rosser, [pH4K3ErugW4](https://youtu.be/pH4K3ErugW4), [mdM4jEWOqx4](https://youtu.be/mdM4jEWOqx4)).

These results come from specific tested units, not a survey of every ESC on the market — treat as firmware/design-level tendencies, not a guarantee for any specific board.

---

*Related Documentation:*
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [BLHeli_32 Guide](blheli32.md)
* [Bluejay Guide](bluejay.md)
* [AM32 Guide](am32.md)
* [ESCape32 Guide](escape32.md)
* [Back to Knowledge Base Index](README.md)
