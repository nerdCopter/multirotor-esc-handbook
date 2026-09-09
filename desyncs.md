# Comprehensive Guide to Motor Desyncs in FPV Multirotors

A **motor desynchronization (desync)** occurs when an Electronic Speed Controller (ESC) loses track of the brushless motor rotor's magnetic and electrical angular position. When the sensorless Back-EMF (Electromotive Force) zero-crossing detection fails, the ESC inverter bridge fires switching MOSFETs out of phase. This causes instantaneous motor stuttering, aggressive uncommanded yaw spins ("death rolls"), extreme current spikes, or total loss of flight control.

> **Sourcing note:** the settings below were cross-checked against real, auto-caption transcripts of the videos cited in [video-audio-knowledge.md](video-audio-knowledge.md). Several numbers in earlier drafts of this guide did not match what those sources actually say, or had no traceable source at all — those have been corrected or removed below rather than just softened. Anything still marked "unverified" has no authoritative source found and should be treated as a starting point to test, not a spec.

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
2. **Timing Lag During Transients:** When throttle increases rapidly, the rotor acceleration rate can lag behind the commanded commutation frequency. If timing is set to `Auto`, the predictive algorithm can miscalculate, causing the magnetic stator field to slip out of synchronization.
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
| **Demag Compensation** | `Off`, `Low`, `High` | `Off`, `Low`, `High` | `Off`, `Low`, `High` / Demag Timing | Configurable Demag | **Leave at default/Low and only step up (Off→Low→High) if you're actually experiencing desyncs** — this is the creators' own stated approach (Pawel Spychalski, [c94e9TCCP8Y](https://youtu.be/c94e9TCCP8Y) @09:33; Chris Rosser, [7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE) @29:09), not "default to High." For an actual punchout/high-RPM desync, going to **High** is the real sourced fix (Ryan Harrell, [oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @19:58 — his #1 recommendation, trading efficiency for stability). |
| **PWM Switching Frequency** | `16kHz`–`128kHz` / Variable | `24kHz`, `48kHz`, `96kHz` | `16kHz`–`128kHz` / By-RPM | `16kHz`–`96kHz` | **Not a simple "lower is safer" dial** (Ryan Harrell, [oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @25:19: he's seen both directions fix the same symptom on different builds). For 5"+ on **32-bit firmware (AM32/BLHeli_32)**, 48kHz static or 24-48kHz Variable/By-RPM is the dominant real-world default — static 24kHz is a valid choice for max braking torque but is not the sourced default for this class (see §4 sizing rules below). The "24kHz for 5"+" figure sometimes cited is specifically about **Bluejay** (8-bit, flash-time-fixed PWM), not a general 32-bit claim. For whoops on Bluejay, the real tradeoff is **duty-cycle resolution, not desync risk** — Bluejay's own maintainers documented 96kHz's reduced resolution (256 steps, worse with higher ESC dead time) as their reason for almost removing it, then restored it after community pushback — see [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown) for the full sourced history. Primarily a torque/efficiency/resolution/braking tradeoff, not a guaranteed desync fix — see [PWM Frequency & Switching Guide](pwm-frequency-heat.md). |
| **Startup / Rampup Power** | `Rampup Power: 10%–150%` | `Startup Min/Max Power` | `Startup Power` / Current Limit | `Spoolup Accel` | BLHeli_32 rampup, sourced by airframe size (Chris Rosser, [7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE)): 5" ≈ `20%–30%` (he uses 30%), 7" ≈ `15%–20%`, sub-3" possibly `>50%` default. **On Bluejay, use the official per-motor/deadtime table in [bluejay.md](bluejay.md#startup-power-motor-idle--rpm-power-protection-official-wiki-data)** (e.g. `1010/1020` for 4S+, `1080/1120` for a 0802 1S deadtime-5 whoop) rather than a flat number — Bluejay's own project wiki publishes this, and its stated philosophy is to use the *lowest* value that reliably starts the motor, not to max it out. |
| **Sine Mode / FOC Startup** | `Sine Mode: ON/OFF` | N/A | `Sinusoidal Startup` | `Sine Spoolup` | **Disable Sinusoidal Start** if arming stalls or low-RPM jitter occur on high-Kv multirotors. |
| **Betaflight Dynamic Idle** | `dyn_idle_min_rpm` | `dyn_idle_min_rpm` | `dyn_idle_min_rpm` | `dyn_idle_min_rpm` | Increase `dyn_idle_min_rpm` (CLI value is RPM ÷ 100) so motor RPM never decelerates into the zero-crossing blind zone. See [Betaflight ESC Integration](betaflight-esc-tuning.md) for the correct CLI name and officially-sourced 5" value. |

---

## 4. Specific Sizing Rules: Propeller, Motor Stator & LiPo Cell Count

### 1. TinyWhoops & Toothpicks (1S–2S | 0702 to 1202.5 | 31mm–2" Props)
* **Motor Characteristics:** Ultra-low stator inductance, sub-millivolt BEMF signal at low RPM, extreme Kv (18,000Kv–30,000Kv).
* **Firmware:** [Bluejay](bluejay.md) or [AM32](am32.md).
* **Desync Prevention Setup:**
  * **PWM Frequency:** Full sourced history in [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown) — short version: Bluejay's own maintainers (release notes v0.20.0, [issue #168](https://github.com/bird-sanctuary/bluejay/issues/168)) documented 96kHz's real limitation as **reduced duty-cycle resolution** (256 steps, worse on ESCs with higher dead time), not desync risk, and nearly removed it before restoring it (merged [PR #154](https://github.com/bird-sanctuary/bluejay/pull/154)) after users reported real flight-time gains. Check your ESC's dead time before assuming 96kHz is free efficiency; a real bench test (Chris Rosser, [EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw)) shows measurable RPM/thrust gains at 96kHz over 48kHz on an 0802-class motor, with the tradeoff being slower active braking (~10-20ms).
  * **Startup Power (Bluejay):** Use the official per-motor/deadtime table — see [bluejay.md](bluejay.md#startup-power-motor-idle--rpm-power-protection-official-wiki-data). For a typical 48kHz 1S 0802 19500Kv deadtime-5 whoop, that's `1080/1120` startup and `9%` idle, not the flat `1100/1200-1250` figure earlier drafts of this guide used.
  * **Motor Timing:** Sourced (Chris Rosser, [EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw)): **15° static** was empirically optimal for an 0802-class whoop motor — higher (22.5°/30°) gave no extra top-end and worse efficiency in his test.
  * **Flight Controller Idle:** Use the official Bluejay Motor Idle table (`8%`–`16%` depending on deadtime, see [bluejay.md](bluejay.md#startup-power-motor-idle--rpm-power-protection-official-wiki-data)) as static idle. On Betaflight 4.5+, that same value can seed Dynamic Idle; on older Betaflight with 1S/2S, stick to static idle (see [Betaflight PR #12432](https://github.com/betaflight/betaflight/pull/12432)).
  * If switching to 96kHz shifts your throttle curve, Chris Rosser's applied Betaflight compensation was `throttle_mid = 1.0`, `throttle_expo = 0.25`.

### 2. Micro & Cinewhoops (3S–6S | 1404 to 2004 | 2.5"–4" Props)
* **Motor Characteristics:** High disc loading, turbulent ducted airflow, aggressive PID responses to propwash.
* **Firmware:** [AM32](am32.md), [BLHeli_32](blheli32.md), or [Bluejay](bluejay.md).
* **Desync Prevention Setup:**
  * **Motor Timing:** Fixed **18°–22.5°** (avoid `Auto` on 4S/6S high-Kv micros).
  * **Demag Compensation:** Start at default/Low; step up only if you experience desyncs (see §3 sourcing — do not default to High).
  * **PWM Frequency:** **48kHz** static, or Variable/By-RPM on 32-bit firmware (AM32/BLHeli_32) — a torque/efficiency middle ground, not a desync-risk dial. (The "~3" = 48kHz" figure cited elsewhere in this KB, Joshua Bardwell, [yEDhnBUFQNI](https://youtu.be/yEDhnBUFQNI), is Bluejay-specific — 8-bit hardware with flash-time-fixed PWM — not a claim scoped to this whole class across all firmwares.)
  * **Dynamic Idle:** `dyn_idle_min_rpm` in the `3500–4200 RPM` range (unverified community range).

### 3. 5-Inch Freestyle & High-Performance (4S–6S | 2207 / 2306 | 1750–2550Kv)
* **Motor Characteristics:** High inertia, extreme throttle punchouts, high-G inverted yaw maneuvers, turtle mode (crash flip — see [Advanced ESC Features](advanced-esc-features.md), this is mixer-level, not a dedicated DShot command).
* **Firmware:** [AM32](am32.md), [BLHeli_32](blheli32.md), [ESCape32](escape32.md).
* **Desync Prevention Setup:**
  * **Motor Timing:** Fixed **22°–23°** (sourced, see §3).
  * **Demag Compensation:** Start at default/Low; increase to High only if experiencing punchout/snap-turn desyncs (sourced, see §3) — this class is exactly the scenario where the sourced fix is to raise Demag, not lower it.
  * **PWM Frequency:** **48kHz static, or 24-48kHz Variable/By-RPM on AM32, is the dominant real-world default for 5" freestyle on 32-bit firmware (AM32/BLHeli_32)** — gives 24kHz-level punch on demand without giving up 48kHz efficiency at cruise. The "24kHz for 5"+" figure elsewhere attributed to Joshua Bardwell ([yEDhnBUFQNI](https://youtu.be/yEDhnBUFQNI)) is specifically about **Bluejay** (8-bit), where PWM is a flash-time-fixed choice with no true variable option — that context doesn't carry over to 32-bit firmware, where Variable/By-RPM makes the 24-vs-48 tradeoff moot for most pilots. Static 24kHz remains a valid choice if you want maximum braking torque and are willing to trade efficiency for it, but it is not the sourced default for this class.
  * **Dynamic Idle:** `dyn_idle_min_rpm` in the `3000–3500 RPM` range (unverified community range, except Betaflight's own officially-sourced 5" default — see [Betaflight ESC Integration](betaflight-esc-tuning.md)).

### 4. 5-Inch Racing Optimization (6S | 2207 / 2208 | 2100–2150Kv)
* **Source of the RPM-gain claim:** the original community resource this KB was built from attributes this specifically: *"Demag compensation low saves roughly 7K RPM (20%) on high kv motors. Although off versus low has minimal effect, some prefer OFF. Thanks to @FreedomDuck for this information. (Ultralight 6S 2100kv to 2150kv.)"* — a named community member's reported result, not from any of the cited YouTube videos, and not independently re-tested here. Treat it as a specific, attributed anecdote worth trying on a clean racing build, not a guaranteed number.
* **Tension with other sourced guidance:** for the closest real matching scenario found in the video research — a 6S 2700kV build actively *desyncing* at high RPM — the sourced expert recommendation (Ryan Harrell, [oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @19:58) is the opposite: raise Demag to **High** and accept the efficiency cost. These aren't necessarily contradictory — @FreedomDuck's report is about a clean, non-desyncing racing build optimizing top-end; Harrell's is about fixing an already-desyncing one — but don't apply the Low/Off RPM-gain trick to a build that's currently desyncing and expect it to help.
* **Guidance:** only try Low/Off Demag for RPM gain on a vibration-free, clean-wired build with a fresh low-ESR capacitor, and confirm it doesn't reintroduce desyncs before racing it. If you're actively desyncing, raise Demag — don't lower it.

### 5. Long Range / Heavy / 7"–10" Macroquads & Cinelifters (6S–12S | 2806.5 to 3115+)
* **Motor Characteristics:** Massive rotor inertia, high stator inductance, huge inductive flyback energy during active braking.
* **Firmware:** [AM32](am32.md), [BLHeli_32](blheli32.md), [ESCape32](escape32.md).
* **Desync Prevention Setup:**
  * **Motor Timing:** **15°–18°** (low timing prevents core saturation and commutation failure on large stators; unverified by transcript but not contradicted, consistent with general L/R time-constant physics).
  * **Demag Compensation:** Start at default/Low, step up to High if experiencing desyncs — larger stators have more flyback energy, so this class is more likely to need it, but "strictly High" as a blanket default is not sourced.
  * **Rampup Power:** Sourced (Chris Rosser, [7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE)): **≈15%–20%** for 7" builds.
  * **PWM Frequency:** Static **24kHz** is the sensible default here for maximum active-braking torque — heavy lifters need that braking authority more than the efficiency gain of a higher frequency. (Note: the Bardwell "24kHz for 5"+" figure cited elsewhere in this KB is Bluejay-specific — see §3 above — not a claim about this class specifically; the recommendation here stands on its own reasoning about torque/inertia, not that citation.) 24-48kHz Variable/By-RPM remains a reasonable alternative if you want some efficiency back.
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
  • Step Demag Compensation up one level (Off→Low→High) — this is the sourced fix
    for punchout/snap-turn desyncs, do not start at High by default.
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
