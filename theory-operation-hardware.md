# ESC Theory of Operation, Electrical Architecture & Hardware Design

Electronic Speed Controllers (ESCs) convert DC electrical power from a LiPo battery into precisely timed 3-phase AC trapezoidal or sinusoidal waveforms to drive sensorless Brushless DC (BLDC) and Permanent Magnet Synchronous Motors (PMSM).

---

## 1. Electrical Inverter Architecture

A typical 3-phase brushless inverter consists of **6 N-Channel MOSFETs** configured in three half-bridge pairs (Phases A, B, and C).

```
          (+) Battery Rail (V_bat)
              │         │         │
           [High A]  [High B]  [High C]  (High-Side N-FETs)
              │         │         │
   Phase A ───┴─── Phase B ─── Phase C ─── (Motor Phase Terminals)
              │         │         │
           [Low A]   [Low B]   [Low C]   (Low-Side N-FETs)
              │         │         │
          (-) Ground Rail (GND) via Current Shunt Resistor
```

### Key Subsystems:
1. **Gate Drivers & Charge Pumps (Bootstrap Circuit):** Because high-side MOSFETs require a gate voltage ($V_{gs}$) higher than the battery rail ($V_{bat} + 10\text{V}$) to fully saturate, bootstrap capacitors and integrated gate drivers provide high-current switching pulses.
2. **Current Sense Shunt Resistor:** Low-resistance precision shunt (typically $0.5\text{m}\Omega$ to $1\text{m}\Omega$) measuring total inverter return current for hardware overcurrent protection (OCP) and analog/DShot telemetry.
3. **Low-Pass Filter & Voltage Dividers:** Attenuate phase voltages down to MCU ADC / comparator voltage ranges ($0\text{V}–3.3\text{V}$) for Back-EMF sensing.

---

## 2. Sensorless Commutation & Back-EMF Zero-Crossing Detection

In a standard 6-step trapezoidal commutation cycle, two phases are energized at any given moment (one high, one low), while the **third phase floats un-driven**.

```
Step 1: Phase A High (+), Phase B Low (-), Phase C Floating (Sensing Zero-Crossing)
Step 2: Phase A High (+), Phase C Low (-), Phase B Floating
Step 3: Phase B High (+), Phase C Low (-), Phase A Floating
Step 4: Phase B High (+), Phase A Low (-), Phase C Floating
Step 5: Phase C High (+), Phase B Low (-), Phase A Floating
Step 6: Phase C High (+), Phase A Low (-), Phase B Floating
```

### Zero-Crossing Detection (ZCD) Physics:
* As the permanent magnets in the rotor sweep past the un-energized stator tooth, they induce a voltage called **Back-Electromotive Force (BEMF)**.
* When the BEMF waveform crosses the virtual motor neutral point ($V_{bat} / 2$), a **Zero-Crossing** event is registered.
* **Commutation Advance Angle:** The ESC timer waits an electrical delay corresponding to $30^\circ$ minus the configured **Motor Timing Advance** before switching to the next step.

```
Induced BEMF Voltage
       ▲
       │        / Phase Floating
+Vbat  │       /
       │      /
Neutral┼─────* Zero-Crossing Point (BEMF = Vbat / 2)
(Vbat/2)    /
       │   /
 GND   │  /
       └────────────────────────► Time
              ◄── 30° Delay ──►[Next Commutation Step]
```

---

## 3. Active Braking / Damped Light (Regenerative Damping)

Traditional legacy ESCs allowed un-driven motor phases to freewheel through MOSFET body diodes during deceleration (passive freewheeling). Modern firmware utilizes **Active Braking (Complementary PWM / Damped Light)**:
* When PWM is low, the complementary low-side MOSFET is actively switched ON.
* **Benefits:** 
  * Near-instantaneous motor deceleration response.
  * Eliminates prop-wash oscillations during rapid pitch/roll axis transitions.
  * Recovers kinetic energy back into the DC bus (regenerative braking).
* **Electrical Challenge:** Regenerative braking injects massive inductive flyback voltage spikes into the DC rail, requiring low-ESR filtering capacitors.

---

## 4. Power Filtering: Capacitors, TVS Diodes & ESC Longevity

Active braking and fast MOSFET switching generate sharp voltage ripple spikes ($V = L \cdot \frac{di}{dt}$) that can easily exceed the voltage breakdown rating of the MOSFETs (e.g. 35V spikes on a 6S 25.2V pack).

### Required Filtering Protection:
1. **Low-ESR Electrolytic Capacitors (Panasonic FR / Rubycon ZLH):**
   * *4S Builds:* $35\text{V}, 470\mu\text{F}–1000\mu\text{F}$
   * *6S Builds:* $35\text{V}–50\text{V}, 470\mu\text{F}–1000\mu\text{F}$
   * *8S–12S Heavy Lifters:* $50\text{V}–63\text{V}, 1000\mu\text{F}–2200\mu\text{F}$
2. **TVS (Transient Voltage Suppressor) Diodes:** Absorbs sub-microsecond overvoltage spikes before they reach the ESC logic gate drivers.
3. **Placement Rule:** Capacitors must be soldered **directly to the ESC battery pads** with lead lengths under 5mm for maximum attenuation.

---

*Related Documentation:*
* [Protocols & Telemetry](protocols-telemetry.md)
* [PWM Frequency & Switching Guide](pwm-frequency-heat.md)
* [Desync Troubleshooting Guide](desync-troubleshooting.md)
* [Back to Knowledge Base Index](README.md)
