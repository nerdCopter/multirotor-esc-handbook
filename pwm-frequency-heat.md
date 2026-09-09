# PWM Switching Frequency, Braking & Thermal Dynamics

The Pulse Width Modulation (PWM) switching frequency of an ESC determines how often the inverter MOSFETs switch on and off to regulate voltage and current through the motor windings.

---

## 1. Trade-Off Analysis: Low vs High PWM Frequency

| Feature | Low Frequency (24kHz) | Moderate (48kHz) | High (96kHz–128kHz) |
| :--- | :--- | :--- | :--- |
| **Throttle Smoothness** | Crisp / aggressive | Very smooth | Ultra smooth / soft |
| **Dynamic Braking Authority** | Maximum | Balanced | Reduced |
| **Low-End Torque & Grip** | High | Balanced | Lower perceived torque |
| **Flight Time / Hover Efficiency** | Baseline | +3% to +6% | +5% to +10% |
| **MOSFET Switching Heat** | Minimal | Moderate | **High to Extreme** |
| **Motor Stator Heating** | Moderate | Lower | Low |
| **Desync Risk at Low RPM** | **Lowest** (Clean back-EMF) | Moderate / Safe | **High** (Noisy zero-crossing) |

---

## 2. Thermal Management & High Frequency Risks

* **MOSFET Switching Losses ($P_{sw}$):** Every time a MOSFET transitions from cutoff to saturation (and vice versa), it passes through its linear resistive region, dissipating power as heat ($P_{sw} \propto f_{pwm} \times V_{in} \times I \times t_{transition}$).
* **128kHz / 96kHz Thermal Runaway:** On ESCs with high gate capacitance ($Q_g$) or slow gate drivers, running at 96kHz or 128kHz can cause MOSFET temperatures to exceed 100°C–120°C in flight, leading to gate breakdown or solder reflow.
* **Motor Heat Paradox:** While higher PWM frequencies reduce eddy current core losses in the motor stator iron (making the motors run cooler), they transfer that thermal burden directly onto the ESC MOSFETs.

---

## 3. Dynamic & Variable PWM Methods

1. **Static PWM:** The ESC remains at one frequency (e.g. 24kHz or 48kHz) regardless of throttle.
2. **Variable PWM (By Throttle):** 
   * *Low Throttle:* Switches at 24kHz for crisp low-end handling and maximum active braking response.
   * *High Throttle:* Transitions to 48kHz or 96kHz for aerodynamic efficiency and smoother top-end control.
3. **Variable PWM "By RPM" (AM32):**
   * Automatically scales switching frequency based on measured motor angular velocity (RPM), mitigating resonance and keeping commutation clean.

---

*Related Documentation:*
* [Desync Troubleshooting Guide](desync-troubleshooting.md)
* [BLHeli_32 Technical Guide](blheli32.md)
* [Bluejay Technical Guide](bluejay.md)
* [AM32 Technical Guide](am32.md)
* [Back to Knowledge Base Index](README.md)
