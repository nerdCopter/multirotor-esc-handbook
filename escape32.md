# ESCape32 Technical Reference Guide

ESCape32 is a high-performance open-source 32-bit ESC firmware created by Roman Bapst (neoxic) specifically for STM32 and AT32 microcontrollers (STM32G4, STM32G0, AT32F421). It emphasizes clean, deterministic architecture, advanced commutation algorithms, and bidirectional telemetry.

---

## 1. Documentation & Community

* **Official Repository:** [GitHub neoxic/ESCape32](https://github.com/neoxic/ESCape32)
* **Community Discord:** [ESCape32 Discord Server](https://discord.gg/aed6xdSM5Y)

---

## 2. Key Architecture & Features

* **Advanced Commutation Timing:** Employs precise timing estimators with adaptive zero-crossing filtering to maintain lock under extreme motor deceleration.
* **Full DShot Support:** DShot150, DShot300, DShot600, DShot1200 with bidirectional RPM telemetry (e-RPM reporting for Betaflight RPM filtering).
* **Hardware Portability:** Native support for STM32G4xx, STM32G0xx, and AT32F421 chips, providing an easy migration target for unlocked BLHeli_32 boards.
* **Thermal & Voltage Protection:** Integrated sensor telemetry over DShot and analog ADC.

---

*Related Documentation:*
* [Firmware Comparison](firmware-comparison.md)
* [AM32 Technical Guide](am32.md)
* [Desync Troubleshooting Guide](desync-troubleshooting.md)
* [Back to Knowledge Base Index](README.md)
