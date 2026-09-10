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
1. **Gate Drivers & Charge Pumps (Bootstrap Circuit):** A high-side N-FET needs $V_{gs}$ (gate-to-source, not gate-to-ground) roughly $10\text{V}$–$15\text{V}$ above its source. Because the high-side source floats at $V_{bat}$ once that FET is on, its gate must be driven to roughly $V_{bat} + 10\text{V}$–$15\text{V}$ referenced to ground. A bootstrap capacitor charged during the low-side on-time supplies this floating rail; the integrated gate driver switches it onto the gate.
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

Active braking and fast MOSFET switching generate sharp voltage ripple spikes ($V = L \cdot \frac{di}{dt}$). Direct oscilloscope measurement on a 4S/6S 2207 test motor with no capacitor or TVS diode fitted showed spikes reaching roughly double the average battery voltage on the battery leads, and higher still on individual motor phases during active braking (e.g. 44.5V battery-lead spike on a 6S pack averaging 23.8V; up to 35V on a motor phase at 4S). Sourced (Chris Rosser, [VgHOHcWu7-U](https://youtu.be/VgHOHcWu7-U)).

### Capacitor Construction & Failure Modes
Two capacitor types are used on FPV ESCs, with different tradeoffs. Sourced (Chris Rosser, [ANq7a2S0Gik](https://youtu.be/ANq7a2S0Gik)):
* **Wet electrolytic:** charge moves as ions through a liquid electrolyte — higher ESR, but the electrolyte can regrow the dielectric layer after a moderate overvoltage event ("self-healing"). Loses roughly 20-50% of its rated capacitance at 24-48kHz switching frequency (rated at 120Hz) because ions can't fully penetrate the foil's etched surface area at high frequency.
* **Solid polymer:** charge moves as electrons through a conductive polymer — ESR roughly 1,000-10,000x lower, and retains capacitance much better at switching frequency. Cannot self-heal from an overvoltage event: dielectric breakdown melts the polymer and the capacitor fails short/explosively. More fragile against voltage spikes at the same voltage rating; needs a TVS diode on the ESC to keep spikes within its rating.
* **Hybrid polymer:** a solid-polymer cathode plus a small amount of liquid electrolyte for partial self-healing — combines most of solid polymer's low ESR with wet electrolytic's spike durability, at higher cost.

**Lead material matters as much as the capacitor's own spec.** Most capacitor leads are copper-clad steel, not solid copper — graded by IACS (International Annealed Copper Standard) conductivity rating relative to pure copper. Cheap/unbranded capacitors commonly use ~20% IACS leads; branded low-ESR capacitors use higher grades (30-40% IACS). At ESC switching currents, low-IACS leads heat resistively (I²R), and rising temperature raises resistance further — a thermal-runaway loop that can melt the lead off the capacitor body. Once a lead melts, filtering is gone and the resulting spike can destroy the FETs, TVS diode, or voltage regulator. IACS grade can't be judged visually or by touch; it was only confirmed in testing via thermal-camera imaging under sustained load. Sourced (Chris Rosser, [ANq7a2S0Gik](https://youtu.be/ANq7a2S0Gik)).

### Sizing Guidance
Voltage rating: **1.5x to 2x maximum pack voltage** (1.5x for smaller/cruising builds, 2x for heavier/aggressive builds or extra crash-spike margin). Capacitance scales with prop size more than battery voltage. Sourced (Chris Rosser, [ANq7a2S0Gik](https://youtu.be/ANq7a2S0Gik)):

| Prop Size | Capacitance | Notes |
| :--- | :--- | :--- |
| Up to 2.5" | 220µF | |
| Up to 4" | 330–470µF | hybrid polymer preferred on 4S/6S |
| 5" | 680–1000µF wet electrolytic, or 470–680µF hybrid polymer | solid polymer not recommended at 5"+ — tested more prone to failure under spikes |
| 7" | 1000–2000µF wet electrolytic | |
| 8"–10" | 2000µF+ wet electrolytic | |
| Over 10" | 3000µF+ wet electrolytic | |

For large builds needing 2000–4000µF, several smaller capacitors in parallel are recommended over one large one — easier to fit, and paralleling reduces net ESR (four 1000µF capacitors in parallel ≈ a quarter of the ESR of a single 1000µF capacitor).

**Brands identified in testing as suitable for FPV ESC use** (sourced Chris Rosser, [ANq7a2S0Gik](https://youtu.be/ANq7a2S0Gik)):
* Wet electrolytic: Panasonic FR (recommended default) or FM; Rubycon ZLH/ZLJ (recommended), or the more compact but lower ripple-current FS/ZLQ.
* Solid polymer: Nichicon's UPL series (used as stock equipment on some iFlight ESCs).
* Hybrid polymer: Panasonic, Rubycon, and Nichicon each make a hybrid-polymer range — recommended over solid polymer alone for 5"+ or 6S builds needing maximum durability.
* Cheap unbranded or generic-branded "low ESR" capacitors were found in testing to use lower-grade (~20% IACS) leads that ran hot enough to melt at mid-throttle, with capacitor bodies exceeding 120°C from excess internal ESR heating — avoid regardless of a printed "low ESR" marking if the brand/series can't be confirmed.

### Placement Rule
Capacitors must be soldered **directly to the ESC battery pads**. Lead length has a measurable but modest effect: extending leads from as-short-as-possible to 20mm increased spike size by roughly 0.5–1V in direct testing — worth avoiding, but not critical if a slightly longer lead is needed to fit a build. If leads must be extended, use thicker wire (e.g. spare motor wire) to keep the added resistance low. Sourced (Chris Rosser, [VgHOHcWu7-U](https://youtu.be/VgHOHcWu7-U)).

### TVS (Transient Voltage Suppressor) Diodes
A TVS diode is wired in reverse bias across the battery leads and conducts only once voltage exceeds its breakdown rating, clamping the top off a voltage spike. Directly tested on the same 6S rig: a capacitor alone held battery-lead spikes to roughly battery voltage +2V, while a TVS diode alone (no capacitor) only reduced spikes to roughly 30–32V — clamping the peak, but far less effective than a capacitor at suppressing the spike in the first place. **A capacitor is the primary defense; a TVS diode is a backup** — useful if the capacitor fails or comes desoldered mid-flight, or as extra margin on extreme builds (e.g. speed-record attempts) where individual motor-phase spikes can exceed what a battery-lead capacitor addresses. Do not solder a capacitor across motor phases directly (it disrupts commutation); a TVS diode with a breakdown voltage just under the FETs' rating is the correct component there instead. Sourced (Chris Rosser, [VgHOHcWu7-U](https://youtu.be/VgHOHcWu7-U)).

---

## 5. Dead Time / Shoot-Through Protection

Distinct from Demag Compensation (§ above) but often confused with it. **Dead time** is the brief interval where both the high-side and low-side MOSFET on the same phase half-bridge are held off during a switching transition, preventing them from ever conducting simultaneously — a direct short across the battery rail ("shoot-through") that would otherwise instantly destroy both FETs. Sourced (Ryan Harrell, [oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @05:14).

On BLHeli_S/Bluejay hardware this is a **fixed, firmware/target-level parameter**, not a user-adjustable setting on most modern firmware — it's baked into the ESC's target layout definition (visible in the layout name format `x_y_nn`, where `nn` is the deadtime value, e.g. `O_H_5`). See [bluejay.md](bluejay.md#startup-power-motor-idle--rpm-power-protection-official-wiki-data) — a higher deadtime value on a given target generally requires higher Startup Power and Motor Idle settings to start reliably.

---

*Related Documentation:*
* [Protocols & Telemetry](protocols-telemetry.md)
* [PWM Frequency & Switching Guide](pwm-frequency-heat.md)
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [Bluejay Technical Guide](bluejay.md)
* [Back to Knowledge Base Index](README.md)
