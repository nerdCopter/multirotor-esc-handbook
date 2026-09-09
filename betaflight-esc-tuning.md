# Flight Controller & ESC Interaction / Tuning Guide

The interaction between the Flight Controller (FC) PID loop and the Electronic Speed Controller (ESC) directly determines flight stability, propwash suppression, tracking authority, and thermal performance.

---

## 1. Bi-directional DShot & PID Loop Synchronization

To prevent packet jitter and timing artifacts, the FC PID loop frequency must match the DShot rate capabilities:

| FC Loop Frequency | Recommended DShot Protocol | Hardware Compatibility |
| :--- | :--- | :--- |
| **8.0 kHz** | **DShot600** (or DShot300) | STM32F7 / STM32H7 / G4 with 32-bit ESCs |
| **4.0 kHz** | **DShot300** (or DShot600) | STM32F405 / F411 / Bluejay 8-bit & 32-bit ESCs |
| **2.0 kHz** | **DShot150** / **DShot300** | Ultra-budget / High-load MCUs |

> [!IMPORTANT]
> When enabling **Bidirectional DShot** on 8-bit [Bluejay](bluejay.md) ESCs, running **DShot300 at 4kHz PID loop** is the gold standard. Attempting DShot600 on 8-bit hardware can cause packet error rate spikes due to MCU interrupt latency.

---

## 2. Dynamic Idle (`idle_min_rpm`)

In traditional flight controllers, `motor_idle_throttle` was set as a static percentage (e.g. 5.5%). Under high-G maneuvers or zero-throttle drops, aerodynamic airflow can slow down or stall propellers, triggering severe desyncs and loss of PID authority.

### How Dynamic Idle Works:
* With Bidirectional DShot enabled, Betaflight monitors the instantaneous RPM of every motor.
* If a motor's speed drops toward the configured `idle_min_rpm`, the FC dynamically increases digital throttle to prevent the motor from stalling or dropping into zero-crossing noise territory.

### Recommended `idle_min_rpm` Settings:
* **TinyWhoops (1S/2S, 0702–1002):** `4500` to `6500 RPM` (or static 9%–14% idle).
* **3"–3.5" Micros (1404–1507):** `3500` to `4200 RPM`.
* **5" Freestyle / Racing (2207/2306):** `3000` to `3500 RPM` (Value in CLI: `set idle_min_rpm = 32`).
* **7"–10" Macroquads:** `2000` to `2600 RPM`.

---

## 3. D-Shot Beacon & Motor Direction Configuration

### Motor Direction Management:
* Standard quad setups spin **Props-In** (inward towards the camera).
* Many freestyle and race pilots configure **Props-Out** (Reverse direction) to push debris and grass away from the FPV camera lens and reduce propwash over the fuselage.
* Motor direction can be flipped without resoldering:
  1. In Betaflight Motor Tab (using DShot command).
  2. In ESC configurators ([AM32 Configurator](https://am32.ca/), [ESC-Configurator](https://esc-configurator.com/), or BLHeli Suite).

### Lost Quad DShot Beacon:
* DShot allows the FC to command the ESC to oscillate the motor coils at audible frequencies (1kHz–4kHz), acting as a loud buzzer when the quad is lost in tall grass.
* Configured in Betaflight *Beeper Configuration* and ESC *Beacon Strength / Delay* parameters.

---

*Related Documentation:*
* [Protocols & Telemetry](protocols-telemetry.md)
* [Desync Troubleshooting Guide](desync-troubleshooting.md)
* [PWM Frequency & Switching Guide](pwm-frequency-heat.md)
* [Back to Knowledge Base Index](README.md)
