# Comprehensive Guide to Motor Desyncs in FPV Multirotors

A **motor desynchronization (desync)** occurs when an Electronic Speed Controller (ESC) loses track of the brushless motor rotor's magnetic and electrical angular position. When the sensorless Back-EMF (Electromotive Force) zero-crossing detection fails, the ESC inverter bridge fires switching MOSFETs out of phase. This causes instantaneous motor stuttering, aggressive uncommanded yaw spins ("death rolls"), extreme current spikes, or total loss of flight control.

> **Sourcing note:** the settings below are cross-checked against real, auto-caption transcripts of the videos cited in [video-audio-knowledge.md](video-audio-knowledge.md). Anything marked "unverified" has no authoritative source found and should be treated as a starting point to test, not a spec.

---

## 1. Physics & Root Causes of Desyncs

```
                      [ Root Mechanisms of Desyncs ]
                                    │
    ┌───────────────────────────────┼───────────────────────────────┐
    ▼                               ▼                               ▼
[ Back-EMF Signal Loss ]    [ Commutation Lag / Advance ]   [ Demagnetization Collapse ]
  • PWM freq too close to     • Rapid throttle steps (0-100%)  • Inductive energy in un-driven
    the noise floor for         overwhelms "Auto" timing.        winding not fully decayed
    the build (either           • Mismatched timing for stator     before next step fires.
    direction — see below).      inductance / pole count.       • Short-circuit current spike.
  • Ultra-low RPM produces
    sub-millivolt BEMF.
```

1. **Back-EMF Zero-Crossing Collapse:** Sensorless ESCs rely on measuring induced voltage across the un-driven floating phase. Electrical noise or a too-narrow sampling window can drown out the zero-crossing signal. **Important:** real-world testimony (Ryan Harrell, [oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @25:19) is that PWM frequency's relationship to this is build-dependent and **not one-directional** — he has seen both raising and lowering PWM frequency fix the same desync symptom on different builds. Treat "just drop the PWM frequency" as one thing to try, not a guaranteed fix. See [PWM Frequency & Switching Guide](pwm-frequency-heat.md).
2. **Timing Lag During Transients:** When throttle increases rapidly, the rotor acceleration rate can lag behind the commanded commutation frequency. If timing is set to `Auto`, the predictive algorithm can miscalculate, causing the magnetic stator field to slip out of synchronization. **This isn't just theoretical** — Chris Rosser demonstrated an actual desync at 0° AM32 timing in his own test data (the motor loses control entirely under acceleration, [3SHzyUaypFw](https://youtu.be/3SHzyUaypFw)), real observed evidence for why near-zero timing is a real failure mode, not just a best-practice caution. See [am32.md](am32.md#commutation-timing--demag-advance) for the sourced timing recommendation this points to.
3. **Demagnetization Energy Spike:** When a phase shuts off, its inductive magnetic field collapses, creating flyback current. If the ESC energizes the next phase before this current dissipates, a high-current short occurs across the bridge, blinding the BEMF comparator. Also distinct from — but related to — **dead time / shoot-through** protection; see [Theory of Operation & Hardware](theory-operation-hardware.md#5-dead-time--shoot-through-protection).

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
    (dyn_idle_min_rpm)                                    • Increase Demag Compensation
  • Try PWM Frequency one step in                            one step (Off→Low→High) —
    EITHER direction (build-dependent,                       this is the sourced fix for
    see pwm-frequency-heat.md)                                punchout/snap desyncs
  • Tune Startup Power (use official                       • Lower Rampup Power (see §3 for
    Bluejay table or BLHeli_32 §3 figures)                     sourced per-size figures)
  • Disable Sinusoidal Start (AM32)                        • Check for Motor Screw Stator Shorts
                                                            • Verify Low-ESR Capacitor & TVS Diode
```

---

## 3. Master Settings Matrix Across All ESC Firmwares

| Configuration Parameter | [BLHeli_32](blheli32.md) | [Bluejay](bluejay.md) | [AM32](am32.md) | [ESCape32](escape32.md) | Recommended Setting for Desync Prevention |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Motor Timing** | `Auto` or `1°–31°` | `Auto`, `Medium`, `Med-High`, `High` | `Auto` or `15°–26°` | `15°–26°` | **Disable `Auto`.** Set static **`22°–23°`** for standard 5" builds (sourced: Chris Rosser, [7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE) @16:44). Set **`15°–18°`** for large high-inductance stators (7"+, unverified but not contradicted). |
| **Demag Compensation** | `Off`, `Low`, `Medium`, `High` | `Off`, `Low`, `High` | **N/A — no Demag setting exists** | Configurable Demag (levels unverified) | **BLHeli_32/Bluejay: leave at default/Low and only step up one level at a time if you're actually experiencing desyncs** (Low→Medium→High on BLHeli_32's 4 levels; Low→High on Bluejay's 3), not "default to High" (Pawel Spychalski, [c94e9TCCP8Y](https://youtu.be/c94e9TCCP8Y) @09:33; Chris Rosser, [7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE) @29:09). For an actual punchout/high-RPM desync, going to **High** is the sourced fix (Ryan Harrell, [oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @19:58 — his #1 recommendation, trading efficiency for stability). **AM32 has no equivalent control** — the real AM32 configurator UI has no Demag/Demag Compensation/Demag Timing field. If an AM32 build desyncs, check **Motor KV** and **Motor poles** are set to the actual motor first — see [am32.md](am32.md#commutation-timing--am32-has-no-demag-compensation-setting). |
| **PWM Switching Frequency** | `16kHz`–`128kHz` / Variable / By-RPM | `24kHz`, `48kHz`, `96kHz` | `16kHz`–`128kHz` / Variable / By-RPM | `16kHz`–`96kHz` | **Not a simple "lower is safer" dial** (Ryan Harrell, [oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @25:19: both directions have fixed the same symptom on different builds). **Fixed 24kHz, fixed 48kHz, and Variable/By-RPM are all configurable on both AM32 and BLHeli_32** — see §3/§4 below for what two specific tests found per firmware; a firmware name next to one frequency here isn't a claim it lacks the others. For Bluejay/whoops, the real 96kHz tradeoff is **resolution, not desync risk** — see [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown). Primarily a torque/efficiency/resolution/braking tradeoff, not a guaranteed desync fix — see [PWM Frequency & Switching Guide](pwm-frequency-heat.md). |
| **Startup / Rampup Power** | `Rampup Power: 10%–150%` | `Startup Min/Max Power` | `Startup Power` / Current Limit | `Spoolup Accel` | BLHeli_32 rampup, sourced by airframe size (Chris Rosser, [7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE)): 5" ≈ `20%–30%` (he uses 30%), 7" ≈ `15%–20%`, sub-3" possibly `>50%` default. **On Bluejay, use the official per-motor/deadtime table in [bluejay.md](bluejay.md#startup-power-motor-idle--rpm-power-protection-official-wiki-data)** (e.g. `1010/1020` for 4S+, `1080/1120` for a 0802 1S deadtime-5 whoop) rather than a flat number — Bluejay's own project wiki publishes this, and its stated philosophy is to use the *lowest* value that reliably starts the motor, not to max it out. |
| **Sine Mode / FOC Startup** | `Sine Mode: ON/OFF` | N/A | `Sinusoidal Startup` | `Sine Spoolup` | **Disable Sinusoidal Start** if arming stalls or low-RPM jitter occur on high-Kv multirotors. |
| **Betaflight Dynamic Idle** | `dyn_idle_min_rpm` | `dyn_idle_min_rpm` | `dyn_idle_min_rpm` | `dyn_idle_min_rpm` | Increase `dyn_idle_min_rpm` (CLI value is RPM ÷ 100) so motor RPM never decelerates into the zero-crossing blind zone. See [Betaflight ESC Integration](betaflight-esc-tuning.md) for the correct CLI name and officially-sourced 5" value. |

---

## 4. Specific Sizing Rules: Propeller, Motor Stator & LiPo Cell Count

> **Firmware lines below show common/typical pairings for that class, not exclusivity.** Bluejay, AM32, BLHeli_32, and ESCape32 can all technically run on any airframe size — a Bluejay ESC (8-bit) doesn't stop working on a 7" build, and BLHeli_32/AM32 aren't restricted to 5"+. The pairings shown reflect what's commonly used in practice (mostly cost, weight, and current-handling reasons), not a hard rule.
>
> **Demag Compensation exists differently per firmware, not as one universal Off→High scale:** BLHeli_32 has 4 levels (`Off`/`Low`/`Medium`/`High`), Bluejay has 3 (`Off`/`Low`/`High`), and **AM32 has no Demag Compensation setting at all** — verified directly against the real AM32 configurator UI. Where a bullet below says "step up Demag," that applies to BLHeli_32/Bluejay only; for AM32, check **Motor KV** and **Motor poles** are set to your motor's actual values instead — see [am32.md](am32.md#commutation-timing--am32-has-no-demag-compensation-setting).

### 1. TinyWhoops & Toothpicks (1S–2S | 0702 to 1202.5 | 31mm–2" Props)
* **Motor Characteristics:** Ultra-low stator inductance, sub-millivolt BEMF signal at low RPM, extreme Kv (18,000Kv–30,000Kv).
* **Firmware:** [Bluejay](bluejay.md) or [AM32](am32.md).
* **Desync Prevention Setup:**
  * **PWM Frequency:** Full sourced history in [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown) — short version: Bluejay's own maintainers (release notes v0.20.0, [issue #168](https://github.com/bird-sanctuary/bluejay/issues/168)) documented 96kHz's real limitation as **reduced duty-cycle resolution** (256 steps, worse on ESCs with higher dead time), not desync risk, and nearly removed it before restoring it (merged [PR #154](https://github.com/bird-sanctuary/bluejay/pull/154)) after users reported real flight-time gains. Check your ESC's dead time before assuming 96kHz is free efficiency; a real bench test (Chris Rosser, [EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw)) shows measurable RPM/thrust gains at 96kHz over 48kHz on an 0802-class motor, with the tradeoff being slower active braking (~10-20ms).
  * **Startup Power (Bluejay):** Use the official per-motor/deadtime table — see [bluejay.md](bluejay.md#startup-power-motor-idle--rpm-power-protection-official-wiki-data). For a typical 48kHz 1S 0802 19500Kv deadtime-5 whoop, that's `1080/1120` startup and `9%` idle — not a flat `1100/1200-1250`, a figure with no traceable source.
  * **Motor Timing:** Sourced (Chris Rosser, [EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw)): **15° static** was empirically optimal for an 0802-class whoop motor — higher (22.5°/30°) gave no extra top-end and worse efficiency in his test.
  * **Flight Controller Idle:** Use the official Bluejay Motor Idle table (`8%`–`16%` depending on deadtime, see [bluejay.md](bluejay.md#startup-power-motor-idle--rpm-power-protection-official-wiki-data)) as static idle. On Betaflight 4.5+, that same value can seed Dynamic Idle; on older Betaflight with 1S/2S, stick to static idle (see [Betaflight PR #12432](https://github.com/betaflight/betaflight/pull/12432)).
  * If switching to 96kHz shifts your throttle curve, Chris Rosser's applied Betaflight compensation was `throttle_mid = 1.0`, `throttle_expo = 0.25`.

### 2. Micro & Cinewhoops (3S–6S | 1404 to 2004 | 2.5"–4" Props)
* **Motor Characteristics:** High disc loading, turbulent ducted airflow, aggressive PID responses to propwash.
* **Firmware:** [AM32](am32.md), [BLHeli_32](blheli32.md), or [Bluejay](bluejay.md).
* **Desync Prevention Setup:**
  * **Motor Timing — two separate sourced figures for two separate firmwares, not one range:**
      * **AM32, 3" motor:** **15°**, "safe for any 3" quad" (Chris Rosser, [3SHzyUaypFw](https://youtu.be/3SHzyUaypFw)).
      * **BLHeli_32, 3" motor (12-pole/9-coil):** **24°** recommended outright — clear benefit in responsiveness, efficiency, and top-end thrust vs. lower settings (Chris Rosser, [6gv0_jTEYZM](https://youtu.be/6gv0_jTEYZM), a *different* test on a *different* firmware — do not average these two numbers or treat them as interchangeable).
  * **Demag Compensation (BLHeli_32/Bluejay only — AM32 has none, see note above):** Start at default/Low; step up one level at a time only if you experience desyncs (see §3 sourcing — do not default to High, and don't skip Medium on BLHeli_32's 4-level scale).
  * **PWM Frequency — again, two separate firmware-specific sourced findings:**
      * **AM32, 3" motor:** Variable **36-72kHz** tested best ("best balance of acceleration and deceleration, the best efficiency") — [3SHzyUaypFw](https://youtu.be/3SHzyUaypFw). Noticeably higher than the 24-48kHz range sourced for AM32 5"/7" (see §3/§5) — don't carry that number down to this class.
      * **BLHeli_32, 3" motor:** Fixed **24kHz** tested as the sweet spot ([6gv0_jTEYZM](https://youtu.be/6gv0_jTEYZM), same test used for BLHeli_32's 5" figures). By-RPM's real sourced range is ~24-40kHz (same source). **Important safety note from the same test: fixed PWM above 48kHz caused this 3" motor to fail to accelerate cleanly at all** (audibly jerking, sometimes failing to spin up) — avoid 96kHz/128kHz fixed on small BLHeli_32 motors specifically, this is a documented failure, not just an efficiency tradeoff.
      * BLHeli_32's overall Variable PWM range extends up to 128kHz on paper, but the two sourced data points above (24kHz sweet spot, failure above 48kHz on this specific 3" motor) are what's actually tested for this size. (The "~3" = 48kHz" figure cited elsewhere in this KB, Joshua Bardwell, [yEDhnBUFQNI](https://youtu.be/yEDhnBUFQNI), is Bluejay-specific — 8-bit hardware with flash-time-fixed PWM — a third, separate firmware from the two above.)
  * **Dynamic Idle:** `dyn_idle_min_rpm` in the `3500–4200 RPM` range (unverified community range).

### 3. 5-Inch Freestyle & High-Performance (4S–6S | 2207 / 2306 | 1750–2550Kv)
* **Motor Characteristics:** High inertia, extreme throttle punchouts, high-G inverted yaw maneuvers, turtle mode (crash flip — see [Advanced ESC Features](advanced-esc-features.md), this is mixer-level, not a dedicated DShot command).
* **Firmware:** [AM32](am32.md), [BLHeli_32](blheli32.md), [ESCape32](escape32.md).
* **Desync Prevention Setup:**
  * **Motor Timing:** Fixed **22°–23°** (sourced, see §3).
  * **Demag Compensation (BLHeli_32/Bluejay only — AM32 has none, see note above):** Start at default/Low; increase one level at a time (not skipping Medium on BLHeli_32) toward High only if experiencing punchout/snap-turn desyncs (sourced, see §3) — this class is exactly the scenario where the sourced fix is to raise Demag, not lower it.
  * **PWM Frequency — two real data points in tension for BLHeli_32, present both rather than picking one:**
      * **24kHz:** the tested-best option in a rigorous thrust-stand comparison (Chris Rosser, [6gv0_jTEYZM](https://youtu.be/6gv0_jTEYZM)) — best responsiveness/efficiency balance on his specific 5"/3" test motors, with Variable PWM showing little benefit there. This is one build under bench-test conditions, not a claim about what most pilots fly.
      * **48kHz:** the more common static choice for non-race 5" freestyle in wider real-world practice — smoother/quieter, small torque cost versus 24kHz. Not from a formal test; a widely-reported real-world default rather than a bench-tested result.
      * **Pick by goal, not by "which is correct":** 24kHz for Rosser's tested max punch/efficiency, 48kHz for the smoother, more common freestyle default. For racing specifically, 24kHz has the much stronger case (see §4).
      * **AM32 offers the same categories of PWM control** — a wide fixed-frequency range (16-128kHz, not just "24 or 48"), Variable-by-throttle with configurable low/high bounds, and By-RPM. It is not limited to Variable/By-RPM. (This is unlike Bluejay, which genuinely has only three total selectable frequencies — 24/48/96kHz, flash-time-fixed, nothing configurable beyond that.) The same 24-vs-48kHz tradeoff above applies if you run AM32 in Fixed mode. Chris Rosser's AM32-specific test ([3SHzyUaypFw](https://youtu.be/3SHzyUaypFw)) leans toward **24-48kHz Variable/By-RPM** as his tested recommendation for 5" specifically ("definitely use the variable pwm just to avoid any risk of that notchy throttle") — consistent with the AM32 creator's own framing of that feature's intended design ([am32.md](am32.md#dynamic-pwm-modes--pwm-by-rpm)). This 24-48kHz figure is specific to 5"/7" motors in that test — a smaller 3" motor tested best at a higher 36-72kHz range instead (see §2), so don't carry this number to other sizes. BLHeli_32 and AM32 have separate sourced defaults for this reason — the AM32 finding above does not carry over to BLHeli_32, and vice versa.
  * **Dynamic Idle:** `dyn_idle_min_rpm` in the `3000–3500 RPM` range (unverified community range, except Betaflight's own officially-sourced 5" default — see [Betaflight ESC Integration](betaflight-esc-tuning.md)).

### 4. 5-Inch Racing Optimization (6S | 2207 / 2208 | 2100–2150Kv)
* **This entire Demag discussion is BLHeli_32/Bluejay-specific — AM32 has no Demag Compensation setting** (see note at the top of §4 above). If you're on AM32, this section doesn't apply; check Motor KV/Motor poles instead.
* **Source of the RPM-gain claim:** the original community resource this KB was built from attributes this specifically: *"Demag compensation low saves roughly 7K RPM (20%) on high kv motors. Although off versus low has minimal effect, some prefer OFF. Thanks to @FreedomDuck for this information. (Ultralight 6S 2100kv to 2150kv.)"* — a named community member's reported result, not from any of the cited YouTube videos, and not independently re-tested here. Treat it as a specific, attributed anecdote worth trying on a clean racing build, not a guaranteed number.
* **Tension with other sourced guidance:** for the closest real matching scenario found in the video research — a 6S 2700kV build actively *desyncing* at high RPM — the sourced expert recommendation (Ryan Harrell, [oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @19:58) is the opposite: raise Demag to **High** and accept the efficiency cost. These aren't necessarily contradictory — @FreedomDuck's report is about a clean, non-desyncing racing build optimizing top-end; Harrell's is about fixing an already-desyncing one — but don't apply the Low/Off RPM-gain trick to a build that's currently desyncing and expect it to help.
* **Guidance:** only try Low/Off Demag for RPM gain on a vibration-free, clean-wired build with a fresh low-ESR capacitor, and confirm it doesn't reintroduce desyncs before racing it. If you're actively desyncing, raise Demag — don't lower it.

### 5. Long Range / Heavy / 7"–10" Macroquads & Cinelifters (6S–12S | 2806.5 to 3115+)
* **Motor Characteristics:** Massive rotor inertia, high stator inductance, huge inductive flyback energy during active braking.
* **Firmware:** [AM32](am32.md), [BLHeli_32](blheli32.md), [ESCape32](escape32.md).
* **Desync Prevention Setup:**
  * **Motor Timing:** For 7" specifically, **15°** is sourced on AM32 (Chris Rosser, [3SHzyUaypFw](https://youtu.be/3SHzyUaypFw)) — same figure as his 3"/5" tests, "safe for all quads." No source covers 8"-10" timing directly; **15°-18°** for that range is a reasonable extrapolation (larger stators, avoid core saturation) but not independently tested.
  * **Demag Compensation (BLHeli_32/Bluejay only — AM32 has none, see note above):** Start at default/Low, step up one level at a time toward High if experiencing desyncs — larger stators have more flyback energy, so this class is more likely to need it, but "strictly High" as a blanket default is not sourced.
  * **Rampup Power:** Sourced (Chris Rosser, [7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE)): **≈15%–20%** for 7" builds.
  * **PWM Frequency:** For AM32 on 7" specifically, **24-48kHz Variable/By-RPM is sourced** (Chris Rosser, [3SHzyUaypFw](https://youtu.be/3SHzyUaypFw): "definitely use the variable pwm just to avoid any risk of that notchy throttle") — no source covers AM32 PWM at 8"-10" at all. For BLHeli_32 (any size in this class), static **24kHz** is a reasonable default for maximum active-braking torque, but this is engineering reasoning (torque/inertia), not a citation — the Bardwell "24kHz for 5"+" figure cited elsewhere in this KB is Bluejay-specific (see §3) and doesn't apply here either.
  * **Capacitors:** See [Theory of Operation & Hardware](theory-operation-hardware.md#4-power-filtering-capacitors-tvs-diodes--esc-longevity) — sizing is a community-sourced starting point, not a formal spec; size to your actual pack voltage and peak current.

---

## 5. Hardware & Electrical Root Causes of Desyncs

1. **Motor Screw Contacting Stator Windings:**
   * If a motor mount screw is even 0.5mm too long, it presses into the enamel wire of the stator coils. Under motor vibration or frame flex, it creates an intermittent ground short, causing instantaneous desync and FET destruction.
2. **Missing or Inadequate Low-ESR Capacitor:**
   * Active braking (damped light) injects voltage spikes back onto the DC rail. Without a low-ESR capacitor and TVS diode directly across the ESC battery pads, voltage ripple can corrupt the gate driver charge pumps and reset the ESC MCU in flight.
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
  • Set Dynamic Idle `dyn_idle_min_rpm` appropriately for your class (see §3/§4).

[ Step 3: ESC Configuration ]
  • Set Motor Timing to a static value (22°–23° for 5", 15°–18° for 7"+).
  • BLHeli_32/Bluejay: step Demag up one level at a time (Off→Low→Medium→High on
    BLHeli_32, Off→Low→High on Bluejay) — the sourced fix for punchout/snap-turn
    desyncs; do not start at High by default. AM32 has no Demag setting — check
    Motor KV/Motor poles instead (see §4 note).
  • Try PWM Frequency in either direction — it is build-dependent, not one-directional.
  • Set Rampup Power to the sourced per-size figure in §3/§4, not a flat guess.

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
* [Video & Audio Source Transcripts](video-audio-knowledge.md)
* [Back to Knowledge Base Index](README.md)
