# Flight Controller & ESC Interaction / Tuning Guide

The interaction between the Flight Controller (FC) PID loop and the Electronic Speed Controller (ESC) directly determines flight stability, propwash suppression, tracking authority, and thermal performance.

---

## 1. Bi-directional DShot & PID Loop Synchronization

To prevent packet jitter and timing artifacts, the FC PID loop frequency must match the DShot rate capabilities:

| FC Loop Frequency | Recommended DShot Protocol |
| :--- | :--- |
| **8.0 kHz** | **DShot600** |
| **4.0 kHz** (or 3.2 kHz on BMI270 gyros) | **DShot300** |
| **2.0 kHz** | **DShot150** |

Official Betaflight guidance (with RPM filtering / bidirectional DShot enabled): 2K/DShot150, 4K/DShot300, 8K/DShot600 — pushing bidirectional telemetry over a faster protocol than the loop needs tends to raise packet error rate rather than help. Treat any specific MCU-to-protocol mapping as build-dependent, not a fixed rule.

> [!IMPORTANT]
> When enabling **Bidirectional DShot** on 8-bit [Bluejay](bluejay.md) ESCs, running **DShot300 at 4kHz PID loop** is the gold standard. Attempting DShot600 on 8-bit hardware can cause packet error rate spikes due to MCU interrupt latency.

---

## 2. Dynamic Idle (`dyn_idle_min_rpm`)

In traditional flight controllers, `motor_idle_throttle` was set as a static percentage (e.g. 5.5%). Under high-G maneuvers or zero-throttle drops, aerodynamic airflow can slow down or stall propellers, triggering severe desyncs and loss of PID authority.

### How Dynamic Idle Works:
* With Bidirectional DShot enabled, Betaflight monitors the instantaneous RPM of every motor.
* If a motor's speed drops toward the configured `dyn_idle_min_rpm` floor, the FC dynamically raises the idle throttle set-point (`dshot_idle_value`) to prevent the motor from stalling or dropping into zero-crossing noise territory.
* CLI unit: the value is RPM ÷ 100, e.g. `set dyn_idle_min_rpm = 32` targets ~3,200 RPM.

### `dyn_idle_min_rpm` Starting Points:
Betaflight's own guidance gives **30–40 (3,000–4,000 RPM)** as a starting point for a typical 5" quad. The per-class numbers below are community-tuned starting points, not an official spec — verify per motor/prop/cell combo with Blackbox RPM traces before trusting them:
* **TinyWhoops (1S/2S, 0702–1002):** for Bluejay ESCs, use the official [per-motor Motor Idle table](bluejay.md#startup-power-motor-idle--rpm-power-protection-official-wiki-data) (`8%`–`16%` static idle depending on deadtime) rather than a flat range.
* **3"–3.5" Micros (1404–1507):** `~35–42` (3,500–4,200 RPM, unverified community range).
* **5" Freestyle / Racing (2207/2306):** `30–40` (3,000–4,000 RPM) — matches [official Betaflight Dynamic Idle guidance](https://betaflight.com/docs/wiki/guides/current/Dynamic-Idle).
* **7"–10" Macroquads:** `~20–26` (2,000–2,600 RPM, unverified community range).

> [!WARNING]
> **1S/2S Dynamic Idle risk (Bluejay-sourced, [official wiki](https://github.com/bird-sanctuary/bluejay/wiki/Setup)):** prior to Betaflight 4.5, Dynamic Idle on 1S/2S Bluejay builds made motors harder to start, leading pilots to raise ESC Startup Min/Max Power to compensate — which silently disabled the ESC's own startup overcurrent protection (BLHeli_S-family ESCs have no phase-current feedback, so the startup limit is the only safeguard against a stalled-motor overcurrent event). [Betaflight PR #12432](https://github.com/betaflight/betaflight/pull/12432) addressed this. On Betaflight 4.5+, seed Dynamic Idle with the Motor Idle table value above; on older versions with a 1S/2S Bluejay build, use static idle instead.

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
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [PWM Frequency & Switching Guide](pwm-frequency-heat.md)
* [Back to Knowledge Base Index](README.md)
