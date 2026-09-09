# Comprehensive Guide to Motor Desyncs in FPV Multirotors

A **motor desynchronization (desync)** occurs when an Electronic Speed Controller (ESC) loses track of the brushless motor rotor's magnetic and electrical angular position. When the sensorless Back-EMF (Electromotive Force) zero-crossing detection fails, the ESC inverter bridge fires switching MOSFETs out of phase. This causes instantaneous motor stuttering, aggressive uncommanded yaw spins ("death rolls"), extreme current spikes, or total loss of flight control.

---

## 1. Physics & Root Causes of Desyncs

```
                      [ Root Mechanisms of Desyncs ]
                                    │
    ┌───────────────────────────────┼───────────────────────────────┐
    ▼                               ▼                               ▼
[ Back-EMF Signal Loss ]    [ Commutation Lag / Advance ]   [ Demagnetization Collapse ]
  • High PWM frequency        • Rapid throttle steps (0-100%)  • Inductive energy in un-driven
    blurs zero-crossing.        overwhelms "Auto" timing.        winding not fully decayed
  • Ultra-low RPM produces    • Mismatched timing for stator     before next step fires.
    sub-millivolt BEMF.         inductance / pole count.       • Short-circuit current spike.
```

1. **Back-EMF Zero-Crossing Collapse:** Sensorless ESCs rely on measuring induced voltage across the un-driven floating phase. If PWM frequency is excessively high (e.g., 96kHz/128kHz on micro stators) or motor speed drops too low, electrical noise drowns out the zero-crossing signal.
2. **Timing Lag During Transients:** When throttle increases rapidly, the rotor acceleration rate can lag behind the commanded commutation frequency. If timing is set to `Auto`, the predictive algorithm can miscalculate, causing the magnetic stator field to slip out of synchronization.
3. **Demagnetization Energy Spike:** When a phase shuts off, its inductive magnetic field collapses, creating flyback current. If the ESC energizes the next phase before this current dissipates, a high-current short occurs across the bridge, blinding the BEMF comparator.

---

## 2. Diagnostic Flowchart

```
                          [ Motor Desync Occurs ]
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
[ Low Throttle / Arming / Spoolup ]             [ High Throttle Punch / Snap Turn / Yaw Spin ]
         │                                                       │
  • Raise Betaflight Dynamic Idle                         • Set static Motor Timing (20°–23°)
    (or Idle Throttle %)                                  • Increase Demag Compensation to HIGH
  • Drop PWM Frequency (48kHz or 24kHz)                   • Lower Rampup Power / Accel Limit
  • Lower / Tune Startup Power (Bluejay)                  • Check for Motor Screw Stator Shorts
  • Disable Sinusoidal Start (AM32)                       • Verify Low-ESR Capacitor & TVS Diode
```

---

## 3. Master Settings Matrix Across All ESC Firmwares

| Configuration Parameter | [BLHeli_32](blheli32.md) | [Bluejay](bluejay.md) | [AM32](am32.md) | [ESCape32](escape32.md) | Recommended Setting for Desync Prevention |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Motor Timing** | `Auto` or `1°–31°` | `Auto`, `Medium`, `Med-High`, `High` | `Auto` or `15°–26°` | `15°–26°` | **Disable `Auto`.** Set static **`20°–23°`** for standard 5" builds. Set **`15°–18°`** for large high-inductance stators (7"+). |
| **Demag Compensation** | `Off`, `Low`, `High` | `Off`, `Low`, `High` | `Off`, `Low`, `High` / Demag Timing | Configurable Demag | Set to **`High`** on any setup experiencing punchout or snap-turn desyncs. (Use `Low`/`Off` only for dedicated high-Kv racing on clean builds). |
| **PWM Switching Frequency** | `16kHz`–`128kHz` / Variable | `24kHz`, `48kHz`, `96kHz` | `16kHz`–`128kHz` / By-RPM | `16kHz`–`96kHz` | **Drop PWM frequency.** Use **`48kHz`** for Whoops/micros (never 96kHz). Use **`24kHz` or `24kHz–48kHz Variable`** for 5" freestyle/racing and heavy lifters. |
| **Startup / Rampup Power** | `Rampup Power: 10%–150%` | `Startup Min/Max Power` | `Startup Power` / Current Limit | `Spoolup Accel` | Lower Rampup Power to **`30%–50%`** to prevent over-acceleration slip. On Bluejay, set Min: `1100`, Max: `1200–1250` (never max out). |
| **Sine Mode / FOC Startup** | `Sine Mode: ON/OFF` | N/A | `Sinusoidal Startup` | `Sine Spoolup` | **Disable Sinusoidal Start** if arming stalls or low-RPM jitter occur on high-Kv multirotors. |
| **Betaflight Dynamic Idle** | `idle_min_rpm` / `idle_percent` | `idle_min_rpm` / `idle_percent` | `idle_min_rpm` / `idle_percent` | `idle_min_rpm` / `idle_percent` | Increase `idle_min_rpm` (or digital idle throttle %) so motor RPM never decelerates into the zero-crossing blind zone. |

---

## 4. Specific Sizing Rules: Propeller, Motor Stator & LiPo Cell Count

### 1. TinyWhoops & Toothpicks (1S–2S | 0702 to 1202.5 | 31mm–2" Props)
* **Motor Characteristics:** Ultra-low stator inductance, sub-millivolt BEMF signal at low RPM, extreme Kv (18,000Kv–30,000Kv).
* **Firmware:** [Bluejay](bluejay.md) or [AM32](am32.md).
* **Desync Prevention Setup:**
  * **PWM Frequency:** Set strictly to **48kHz**. *Avoid 96kHz*—it collapses the BEMF zero-crossing detection window and causes motor stalls during aggressive maneuvers.
  * **Startup Power (Bluejay):** `Startup Min Power = 1100`, `Startup Max Power = 1200–1250`. Maxing out startup power causes coil saturation and immediate desync.
  * **Flight Controller Idle:** Set static idle to **9%–14%** or Betaflight Dynamic Idle to `4500–6500 RPM`.

### 2. Micro & Cinewhoops (3S–6S | 1404 to 2004 | 2.5"–4" Props)
* **Motor Characteristics:** High disc loading, turbulent ducted airflow, aggressive PID responses to propwash.
* **Firmware:** [AM32](am32.md), [BLHeli_32](blheli32.md), or [Bluejay](bluejay.md).
* **Desync Prevention Setup:**
  * **Motor Timing:** Fixed **18°–22.5°** (avoid `Auto` on 4S/6S high-Kv micros).
  * **Demag Compensation:** **High** (essential for heavy multi-blade ducted props).
  * **PWM Frequency:** **48kHz** or **Variable PWM (24kHz–48kHz)**.
  * **Dynamic Idle:** Set `idle_min_rpm` to `3500–4200 RPM`.

### 3. 5-Inch Freestyle & High-Performance (4S–6S | 2207 / 2306 | 1750–2550Kv)
* **Motor Characteristics:** High inertia, extreme throttle punchouts, high-G inverted yaw maneuvers, turtle mode (crash flip).
* **Firmware:** [AM32](am32.md), [BLHeli_32](blheli32.md), [ESCape32](escape32.md).
* **Desync Prevention Setup:**
  * **Motor Timing:** Fixed **22°–23°**.
  * **Demag Compensation:** **High** for freestyle.
  * **PWM Frequency:** **24kHz–48kHz Variable** or **PWM By-RPM** in AM32.
  * **Dynamic Idle:** Set `idle_min_rpm` to `3000–3500 RPM` (`set idle_min_rpm = 32`).

### 4. 5-Inch Racing Optimization (6S | 2207 / 2208 | 2100–2150Kv)
* **High-Speed Demag Trade-off:** On ultra-clean racing builds, setting Demag to **Low** or **Off** recovers ~7,000 RPM (up to 15–20% top-end power) because the ESC does not artificially delay phase transitions.
* **Warning:** Only use `Low`/`Off` on vibration-free frames with fresh low-ESR capacitors; otherwise, aggressive corner braking will trigger desyncs.

### 5. Long Range / Heavy / 7"–10" Macroquads & Cinelifters (6S–12S | 2806.5 to 3115+)
* **Motor Characteristics:** Massive rotor inertia, high stator inductance, huge inductive flyback energy during active braking.
* **Firmware:** [AM32](am32.md), [BLHeli_32](blheli32.md), [ESCape32](escape32.md).
* **Desync Prevention Setup:**
  * **Motor Timing:** **15°–18°** (low timing prevents core saturation and commutation failure on large stators).
  * **Demag Compensation:** Strictly **High**.
  * **Rampup Power:** Lower rampup / acceleration rate to avoid exceeding magnetic lock during throttle punches.
  * **PWM Frequency:** Fixed **24kHz** or **16kHz–24kHz** for maximum low-end torque.
  * **Capacitors:** Minimum 1000µF Low-ESR capacitor + TVS spike diode soldered directly to the ESC battery pads.

---

## 5. Hardware & Electrical Root Causes of Desyncs

1. **Motor Screw Contacting Stator Windings:**
   * If a motor mount screw is even 0.5mm too long, it presses into the enamel wire of the stator coils. Under motor vibration or frame flex, it creates an intermittent ground short, causing instantaneous desync and FET destruction.
2. **Missing or Inadequate Low-ESR Capacitor:**
   * Active braking (damped light) injects massive voltage spikes back onto the DC rail. Without a low-ESR capacitor (Panasonic FR/Rubycon ZLH) and TVS diode directly across the ESC battery pads, voltage ripple corrupts the gate driver charge pumps and resets the ESC MCU in flight.
3. **Cold Solder Joints / High Resistance Phase Wires:**
   * High electrical resistance on one phase wire leads to asymmetric phase current, causing the zero-crossing comparator to lose tracking under high current draw.
4. **Bent Motor Bell / Cracked Magnets:**
   * Asymmetric magnetic air gaps produce irregular Back-EMF waveforms that confuse the zero-crossing timing algorithm.

---

## 6. Step-by-Step Desync Elimination Workflow

```
[ Step 1: Physical Inspection ]
  • Inspect motor screws under magnification (ensure clear gap from windings).
  • Verify low-ESR capacitor is securely soldered directly to ESC battery pads.
  • Check phase wire solder joints for smooth, shiny, complete wetting.

[ Step 2: Flight Controller Configuration ]
  • In Betaflight, verify Bidirectional DShot is enabled and packet error rate is 0%.
  • Set Dynamic Idle `idle_min_rpm` appropriately (3200 for 5", 5000 for Whoop).

[ Step 3: ESC Configuration ]
  • Set Motor Timing to a static value (22°–23° for 5", 16°–18° for 7"+).
  • Set Demag Compensation to HIGH.
  • Set PWM Frequency to 24kHz–48kHz (avoid 96kHz/128kHz).
  • Lower Rampup Power / Slew Rate to 30%–50%.

[ Step 4: Test Flight & Blackbox Verification ]
  • Perform punchout tests while recording Blackbox log at 2kHz gyro/PID rate.
  • Inspect motor trace lines: if one motor trace jumps to 100% while gyro shows uncommanded roll/pitch/yaw spin, that specific motor suffered a desync.
```

---

*Related Knowledge Base Modules:*
* [Theory of Operation & Hardware](theory-operation-hardware.md)
* [PWM Frequency & Switching Guide](pwm-frequency-heat.md)
* [Protocols & Telemetry](protocols-telemetry.md)
* [Betaflight ESC Integration](betaflight-esc-tuning.md)
* [BLHeli_32 Guide](blheli32.md) | [Bluejay Guide](bluejay.md) | [AM32 Guide](am32.md) | [ESCape32 Guide](escape32.md)
* [Back to Knowledge Base Index](README.md)
