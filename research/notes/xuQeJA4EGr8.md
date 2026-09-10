# xuQeJA4EGr8 — Joshua Bardwell
"BLHeli32 ESC's just got a BIG performance upgrade. Here's how to get it."
https://youtu.be/xuQeJA4EGr8
Channel: Joshua Bardwell

## Timestamped claims

- [00:00-01:01] Topic: BLHeli_32's new "By RPM" variable PWM frequency feature. **Specific version number given: "I think it's 32.8.3 that added that"** — real, checkable, specific fact not currently in the KB (blheli32.md doesn't cite a version number for this feature).
- [01:07-01:26] Confirms BLHeli_S (Bluejay) cannot use this exact feature — separate firmware.
- [05:12-08:03] Mechanism explanation, general/qualitative (references a chart from a Chris Rosser video for the actual numbers, which Bardwell does not repeat verbatim): as PWM frequency rises, motor efficiency rises but braking performance drops. Old (pre-"By RPM") BLHeli_32 variable PWM was simply tied to throttle position — "PWM frequency Low" and "PWM frequency High" bounds, frequency scaling with throttle stick position.
- [08:03-08:29] **Real documented failure mode of the OLD throttle-based variable PWM, not currently in KB**: "the problem is they tied it simply to the throttle position, and it turns out that when you do that the PWM frequency can interact with the RPM of the motor to create harmonics, and those harmonics can cause mid-throttle oscillation, jello, and other flight performance problems." This is a specific, real root-cause claim relevant to desyncs.md / theory-operation-hardware.md's resonance discussion.
- [08:31-08:53] **Configuration mechanism, specific and directly actionable**: for "By RPM" mode, set "PWM frequency Low" as low as possible (max torque/braking at low RPM) and set "PWM frequency High" to "By RPM" — i.e., the BLHeli_32 UI has a dedicated "By RPM" option value for the *High* bound field, distinct from a numeric kHz value. Confirms the KB's general "By RPM" description but adds the concrete UI mechanic.
- [09:23-10:11] To get full benefit, need an ESC whose hardware PWM range spans low (16 or 24kHz) to high (96 or 128kHz) — an ESC that "only goes up to 48k" is explicitly called out by Bardwell as inadequate for this feature.

## KB cross-check (blheli32.md, firmware-comparison.md current tracked content)
- blheli32.md's "Variable PWM Behavior" section describes only the OLD throttle-based behavior, generically, without noting: (a) the real version number (32.8.3) when "By RPM" was added, (b) the specific harmonics/mid-throttle-oscillation ("jello") failure mode of the old system that "By RPM" fixes, (c) the "PWM Low fixed value / PWM High = By RPM" UI mechanic.
- Video title in KB's "Curated Video" list says "Joshua Bardwell Variable PWM Testing & Benchmarks" — this is a mislabel; the video does not contain Bardwell's own testing/benchmarks, it explains the feature and references a chart from a separate Chris Rosser video for the actual efficiency/braking numbers (Chris Rosser videos are the two that failed to download — 3SHzyUaypFw, 6gv0_jTEYZM — so those specific numbers remain unverified pending retry).
- This transcript reinforces (independently of the AM32 Pete Smits interview) that "higher PWM = more efficiency, less braking" is a real, stated trend by multiple creators, though the exact magnitude figures trace back to Chris Rosser's charts, not yet independently verified from his own video.
