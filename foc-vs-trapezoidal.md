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
| **Torque Ripple** | High (~14% theoretical ripple at transitions) | **Near Zero** |
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

## 3. Hybrid Implementations in Modern Firmware

### 1. AM32 Sinusoidal Startup (FOC-Lite)
* **How it works:** When starting from a dead stop (0 RPM), AM32 generates open-loop sinusoidal waveforms (Space Vector Modulation) to align the rotor magnets and smoothly accelerate the motor up to ~300–500 RPM.
* Once Back-EMF amplitude reaches readable levels, the firmware seamlessly hands off control to high-performance 6-step trapezoidal commutation.
* **Benefit:** Completely eliminates low-speed cogging and startup jitter while preserving full top-end throttle punch and active braking power.

---

*Related Documentation:*
* [Theory of Operation & Hardware](theory-operation-hardware.md)
* [AM32 Technical Guide](am32.md)
* [PWM Switching & Thermal Dynamics](pwm-frequency-heat.md)
* [Back to Knowledge Base Index](README.md)
