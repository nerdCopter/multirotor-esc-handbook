# Field Oriented Control (FOC) vs 6-Step Trapezoidal Commutation

FPV multirotors and electric drive systems utilize two primary brushless motor commutation strategies: **6-Step Trapezoidal Commutation** (standard across BLHeli_32, Bluejay, and baseline AM32) and **Field Oriented Control (FOC / Sinusoidal Commutation)**.

---

## 1. Architectural & Theoretical Differences

```
    [ 6-Step Trapezoidal Commutation ]                 [ Field Oriented Control (FOC) ]
  • Two phases driven at any moment                  • All three phases continuously energized
    (1 High, 1 Low, 1 Floating)                        with smooth, 120° phase-offset sine waves.
  • Discrete 60° commutation steps.                  • Continuous Clarke & Park transforms calculate
  • Torque ripple present at step transitions.         direct (Id) and quadrature (Iq) current vectors.
  • Simple Back-EMF Zero Crossing Detection.         • Maximum torque per ampere (MTPA), zero ripple.
```

### Direct Comparison Matrix

| Parameter / Metric | 6-Step Trapezoidal Commutation | Field Oriented Control (FOC) |
| :--- | :--- | :--- |
| **Phase Energization** | 2 Phases ON, 1 Phase Floating | **All 3 Phases Actively Driven** |
| **Current Waveform** | Block / Trapezoidal | Pure Sinusoidal |
| **Torque Ripple** | High (commonly cited ~15% at commutation transitions) | **Low (commonly cited <5%, not literally zero)** |
| **Acoustic Noise** | Distinct high-frequency whine & harmonics | **Virtually Silent** |
| **Low-End Efficiency (<1000 RPM)** | Poor (High cogging, prone to stall) | **Extreme** (Smooth rotation down to 1 RPM) |
| **Top-End RPM Limit & Acceleration** | **Maximum High-RPM Slew Rate & Punch** | Computationally heavy; slightly lower top-end on low-power MCUs |
| **Sensorless Position Sensing** | Direct Back-EMF Zero-Crossing (ZCD) | Sliding Mode Observer (SMO) / High-Frequency Injection (HFI) |
| **MCU Computational Load** | Very Low (Runs on 8-bit 24MHz MCUs) | **High** (Requires 32-bit Cortex-M4/M7 with FPU) |

---

## 2. Why 6-Step Trapezoidal Dominates FPV Drones

1. **Extreme Transient Throttle Acceleration:** FPV freestyle and racing quadcopters demand instant motor acceleration from 2,000 RPM to 35,000 RPM in less than 30 milliseconds. Trapezoidal commutation delivers maximum raw phase voltage instantly without recalculating complex vector transformations.
2. **Simplified Zero-Crossing Detection:** Floating one un-driven phase every 60° electrical provides a clear analog window to measure Back-EMF crossing $V_{bat}/2$, even during violent aerodynamic gusts and crash recoveries.
3. **Firmware Efficiency:** 6-step commutation math is lightweight enough to execute at 48kHz+ switching frequencies on inexpensive microcontrollers.

---

## 3. Voltage-Domain Drive Strategies: Block, Sinusoidal & Space Vector Modulation (SVM)

Within the trapezoidal/sinusoidal families above, the ESC's drive-voltage waveform itself can take three distinct shapes. Sourced (Chris Rosser, citing back-EMF measurement data from FETtec's Felix, [M4NOIwdSBOc](https://youtu.be/M4NOIwdSBOc)):

* **Block commutation:** each phase is driven fully on/off whenever the motor's ideal (back-EMF-shaped) driving current is above/below 50%. Simplest to implement — no switching while a phase is held at zero or full battery voltage — but has three real drawbacks: (1) rapid on/off switching of the coil field causes demagnetization current spikes; (2) roughly a third of each electrical cycle drives no phase at all, spent instead sensing the back-EMF zero-crossing; (3) the drive-current shape doesn't match the motor's ideal sinusoidal back-EMF, under-driving and over-driving the motor at different points in the cycle.
* **Pure sinusoidal commutation:** all three phases are driven with smooth PWM-synthesized sine waves matching the back-EMF shape — most efficient and smoothest torque delivery, but with two drawbacks: (1) constant switching (versus block's mostly-static phases) increases switching losses and ESC heat; (2) peak phase-to-phase voltage tops out around 87% of battery voltage (the two sinusoids are never maximally separated at the same instant), reducing available torque/power versus block commutation's full battery voltage at full throttle.
* **Space Vector Modulation (SVM):** subtracts the instantaneous minimum of the three phase voltages from all three phases. This recovers full battery voltage across each phase at full throttle (restoring block commutation's top-end power), while each phase still spends part of its cycle at zero volts (reducing switching losses versus pure sinusoidal), and the resulting phase *current* remains a clean sinusoid despite the drive *voltage* no longer looking sinusoidal. SVM is the standard drive waveform behind "sinusoidal start" implementations in modern 32-bit firmware.

All three still need a brief zero-drive window per phase to sense the back-EMF zero-crossing for rotor position, which slightly distorts the otherwise-ideal current waveform — see § 4 below for an approach that removes this window entirely.

## 4. FETtec SFOC — Continuous Back-EMF Sensing (Prototype Hardware)

FETtec has prototyped an ESC approach ("SFOC," Simple Field Oriented Control) that removes the zero-drive sensing window described above. Rather than sensing rotor position only during that window (three known points per electrical cycle), it uses **two shunt resistors per motor** — instead of the single ESC-wide battery-lead shunt typical of other FPV ESCs — to continuously measure both current and voltage on each phase, calculating back-EMF and rotor position continuously rather than at three fixed points. This removes the zero-drive-window distortion from the drive waveform, giving a cleaner sinusoidal current, finer commutation-timing resolution, and full-time phase driving (more available torque). Sourced (Chris Rosser, [M4NOIwdSBOc](https://youtu.be/M4NOIwdSBOc)).

Independent bench testing of an early SFOC prototype (firmware v1.01, 2024-03) against BLHeli_32, AM32, and Bluejay ESCs on the same motor/prop found the SFOC prototype reached 90% of a commanded throttle step in 40ms, versus ~44ms for the AM32 and Bluejay units tested and ~52ms for the BLHeli_32 unit tested, and produced more flywheel-test torque than any other unit in the batch. Sourced (Chris Rosser, [QENbsI3swCI](https://youtu.be/QENbsI3swCI)).

**This result does not hold up in a later, larger test batch.** In a 40-ESC comparison two years later (2026-03), a FETtec SFOC unit disarmed and refused to complete both the step-response test and the flywheel torque test, producing no usable data in either. Sourced (Chris Rosser, [pH4K3ErugW4](https://youtu.be/pH4K3ErugW4)). The two results aren't necessarily contradictory — different prototype hardware/firmware revisions two years apart, tested in different batches — but this KB is not treating the 2024 result as a settled characteristic of FETtec SFOC. **This was pre-release prototype hardware/firmware as of both sourced tests; current release/availability status has not been independently verified for this KB.**

## 5. Hybrid Implementations in Modern Firmware

### 1. AM32 Sinusoidal Startup (FOC-Lite)
* **How it works:** When starting from a dead stop, AM32's Sine Start mode drives all three phases with open-loop sinusoidal waveforms; since BEMF sensing is unavailable in this mode, throttle directly controls rotation speed rather than duty cycle. There is no fixed universal RPM handoff figure — AM32 ramps the sine-mode speed to match the minimum speed required at changeover, and the `prot_stall` (stall-protection ERPM threshold) setting can be raised to give the motor a longer sinusoidal run-up before handing off to trapezoidal commutation. See the [AM32 ESC Settings Explained wiki](https://github.com/AlkaMotors/AM32-MultiRotor-ESC-firmware/wiki/ESC-Settings-Explained) for the authoritative behavior.
* Once handed off, the firmware runs standard 6-step trapezoidal commutation using Back-EMF zero-crossing detection.
* **Benefit:** Reduces low-speed cogging and startup jitter versus a pure open-loop trapezoidal start, while preserving full top-end throttle punch and active braking power once running in trap mode.
* **Caveat:** Fixed-speed and fixed-duty-cycle throttle modes are not compatible with sinusoidal startup.

---

*Related Documentation:*
* [Theory of Operation & Hardware](theory-operation-hardware.md)
* [AM32 Technical Guide](am32.md)
* [PWM Switching & Thermal Dynamics](pwm-frequency-heat.md)
* [Back to Knowledge Base Index](README.md)
