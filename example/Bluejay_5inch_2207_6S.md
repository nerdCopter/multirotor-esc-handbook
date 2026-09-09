> q&A: what bluejay timing/demag for 5 inch, 2207, 6S ?
>
> *(Corrected against the sourced findings in [desyncs.md](../desyncs.md), [bluejay.md](../bluejay.md), and [SOURCES.md](../SOURCES.md) — the original answer here had four inaccuracies: a wrong Demag default, unsourced Startup Power numbers, an unattributed RPM-gain claim, and the wrong Betaflight CLI variable name.)*

  For a 5-inch, 2207 stator, 6S build running Bluejay /bluejay.md, here are the recommended settings depending on your flight style:
  ──────
  ### 1. Recommended Settings (Freestyle / General Flight)

  • Motor Timing: Medium-High (or 22.5° / High depending on configurator display)
      • Why: Avoid Auto. A fixed Medium-High / High timing setting provides the exact phase lead needed for 2207 stators on 6S, eliminating tracking lag and desyncs during snap rolls, inverted yaw
      spins, and 0%–100% full-throttle punchouts.
  • Demag Compensation: **Start at default/Low. Only increase it if you're actually experiencing desyncs.**
      • **When to increase it:** if you get a punchout or snap-turn desync — one motor's commanded output pins near 100% while its measured RPM collapses or stutters — step Demag up one level at a
      time (Low → High), not straight to High. This is the sourced fix for that exact symptom (Ryan Harrell, real-source video @19:58 — his #1 recommendation for a build actively desyncing at
      high RPM, trading a bit of efficiency for stability). Don't default to High "just in case" — it costs efficiency for no benefit if you aren't actually desyncing.
  • PWM Frequency: 24kHz or 48kHz
      • 24kHz: sourced as the better choice for 5"+ builds (maximum active braking torque, best low-end grip) — this is a torque/efficiency tradeoff, not strictly a "desync risk" difference.
      • 48kHz: smoother throttle and slightly longer flight times, less braking authority than 24kHz.
  • Startup Power: use the official Bluejay Setup-wiki table for this class rather than a flat guess —
      • 6S is "above 2S," so: **Startup Min/Max Power `1010`/`1020`, Motor Idle `5.5%`.** (Source: Bluejay's own [Setup wiki](https://github.com/bird-sanctuary/bluejay/wiki/Setup) — see the full
      table in bluejay.md.)

  ──────
  ### 2. Dedicated Racing Setup (Maximum Top-End RPM) — not a desync fix, read the caveat below

  If you are running high-Kv 6S racing motors (e.g. 2100Kv–2150Kv) on a clean, low-vibration build with a fresh Low-ESR capacitor, **and you are not currently experiencing desyncs**:

  • Motor Timing: High
  • Demag Compensation: Low (or Off) — **only as a top-end optimization on an already-clean build, never as a response to desyncs.**
      • Attributed claim (from the original community source this KB was compiled from, credited there to a contributor "@FreedomDuck," not independently re-tested): Demag Low/Off reportedly
      recovers roughly 7,000 RPM (~20% power) on ultralight 6S 2100-2150Kv racing motors by removing artificial phase-switching delay.
      • **Important caveat:** if this same class of build is actually desyncing at high RPM, the sourced fix (see §1 above) is to raise Demag toward High, not lower it. Only try Low/Off for RPM
      gain on a build that is already stable — if you apply it to a desyncing build expecting it to help, it won't, and may make things worse.
      • Condition: only attempt this if your motor screws are properly sized, frame resonance is low, and you have a 35V–50V 1000µF Low-ESR capacitor soldered directly to the ESC pads.

  ──────
  ### 3. Corresponding Betaflight Settings

  • Protocol: DShot300 with 4.0 kHz PID loop (Optimal for 8-bit Bluejay MCUs to prevent packet jitter/errors).
  • Dynamic Idle: `set dyn_idle_min_rpm = 32` (~3,200 RPM) to keep the motor above the zero-crossing noise floor during zero-throttle maneuvers. (Corrected CLI variable name — `idle_min_rpm` is not
  the real Betaflight setting; see [betaflight-esc-tuning.md](../betaflight-esc-tuning.md).)
