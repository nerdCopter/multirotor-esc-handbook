# PWM Switching Frequency, Braking & Thermal Dynamics

The Pulse Width Modulation (PWM) switching frequency of an ESC determines how often the inverter MOSFETs switch on and off to regulate voltage and current through the motor windings.

---

## 1. Trade-Off Analysis: Low vs High PWM Frequency

| Feature | Low Frequency (24kHz) | Moderate (48kHz) | High (96kHz–128kHz) |
| :--- | :--- | :--- | :--- |
| **Throttle Smoothness** | Crisp / aggressive | Very smooth | Ultra smooth / soft |
| **Dynamic Braking Authority** | Maximum | Balanced | Reduced |
| **Low-End Torque & Grip** | High | Balanced | Lower perceived torque |
| **Flight Time / Hover Efficiency** | Baseline | Commonly reported modest gain | Commonly reported larger gain, with diminishing returns |
| **MOSFET Switching Heat** | Minimal | Moderate | **High to Extreme** |
| **Motor Stator Heating** | Moderate | Lower | Low |

> **On "Desync Risk at Low RPM":** a desync-risk gradient from 24kHz (lowest) to 96kHz (highest) does not hold up as a general rule. Ryan Harrell ([oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @25:19) states directly that he has seen raising *and* lowering PWM frequency each fix the same desync symptom on different builds — it's build-dependent, not a one-directional dial. The real, better-sourced 96kHz tradeoff (found in Bluejay's own GitHub history — see [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown)) is **duty-cycle resolution**: 96kHz gives only 256 PWM steps in the best case, and higher ESC dead time reduces that further — this is the actual documented reason Bluejay's maintainers almost removed 96kHz entirely, before restoring it after users reported real flight-time gains. Treat PWM frequency primarily as a torque/efficiency/resolution/braking tradeoff (the rows above and the dead-time note in [theory-operation-hardware.md](theory-operation-hardware.md#5-dead-time--shoot-through-protection)), and treat any desync at a given frequency as build-specific evidence to test around, not a property of the frequency itself. See [desyncs.md](desyncs.md#1-physics--root-causes-of-desyncs).

---

## 2. Thermal Management & High Frequency Risks

* **MOSFET Switching Losses ($P_{sw}$):** Every time a MOSFET transitions from cutoff to saturation (and vice versa), it passes through its linear resistive region, dissipating power as heat. The standard hard-switching approximation is $P_{sw} \approx 0.5 \times V_{in} \times I \times (t_{rise} + t_{fall}) \times f_{pwm}$ — switching loss scales linearly with switching frequency, bus voltage, and current.
* **128kHz / 96kHz Thermal Runaway:** On ESCs with high gate capacitance ($Q_g$) or slow gate drivers, running at 96kHz or 128kHz can cause MOSFET temperatures to exceed 100°C–120°C in flight, leading to gate breakdown or solder reflow.
* **Motor Heat Paradox:** While higher PWM frequencies reduce eddy current core losses in the motor stator iron (making the motors run cooler), they transfer that thermal burden directly onto the ESC MOSFETs.

---

## 3. Dynamic & Variable PWM Methods

1. **Static PWM:** The ESC remains at one frequency (e.g. 24kHz or 48kHz) regardless of throttle.
2. **Variable PWM (By Throttle):** 
   * *Low Throttle:* Switches at 24kHz for crisp low-end handling and maximum active braking response.
   * *High Throttle:* Transitions to 48kHz or 96kHz for aerodynamic efficiency and smoother top-end control.
3. **Variable PWM "By RPM" (AM32):**
   * Automatically scales switching frequency based on measured motor angular velocity (RPM) rather than throttle position, mitigating the mid-throttle mechanical/electrical resonance ("jello") that fixed-by-throttle variable PWM can produce.
   * Sourced mechanism, in the AM32 creator's own words (interview with Joshua Bardwell, [yOeVj6P9PSU](https://youtu.be/yOeVj6P9PSU)): fixed at **24kHz** until commutation frequency reaches roughly **11.5kHz**, then ramps linearly up to **48kHz**.
   * BLHeli_32 shipped an equivalent By-RPM feature starting in **firmware v32.8.3** (Joshua Bardwell, [xuQeJA4EGr8](https://youtu.be/xuQeJA4EGr8)).
   * See [am32.md](am32.md#dynamic-pwm-modes--pwm-by-rpm) for a real-world caveat: independent user reports of desyncs/noise from AM32's Variable and By-RPM modes, resolved by switching to fixed frequency. Not a universal upgrade — test it on your build.

---

*Related Documentation:*
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [BLHeli_32 Technical Guide](blheli32.md)
* [Bluejay Technical Guide](bluejay.md)
* [AM32 Technical Guide](am32.md)
* [Back to Knowledge Base Index](README.md)
