# ULTIMATE ESC Settings for Tiny Whoops! — Chris Rosser
https://youtu.be/EhYKeZfSQIw

Test setup: BLHeli_S ESC, 7A rating, flashed to Bluejay, tested on a single 0802 motor + prop on a static test stand (thrust/RPM/current bench, not an in-flight test), using bidirectional DShot telemetry logged via an FC. Four tests: KV test (0-100% at 3V, no prop), throttle ramp (5s to full throttle and back), response test (10-50% throttle steps), efficiency test (1A/4V constant, measure RPM).

## Motor Timing (0802-class whoop motor, Bluejay)
- [05:47] Tested 0°, 7.5°, 15°, 22.5°, 30°.
- [05:59-06:23] Increasing timing raises KV/top-end power, but **no improvement beyond 15°** — 15°, 22.5°, 30° all give the same max RPM/thrust. 0° and 7.5° give significantly less top-end power.
- [07:12-07:52] Efficiency (RPM at constant 1A/4W) peaks at 15°; **22.5° and 30° are LESS efficient than 15°**.
- [07:52-08:03] Timing has **no measurable effect on responsiveness** (accel/decel same regardless of setting).
- [08:03-08:18] **Conclusion, stated as a direct recommendation: 15° is ideal for Bluejay ESCs on very small (0802-class) motors. Timing lower OR higher than 15° reduces both performance and efficiency.**

## PWM Frequency (24k/48k/96kHz)
- [09:33-09:44] Full-throttle KV is unaffected by PWM frequency (ESC isn't switching at 100% duty).
- [09:44-10:35] Below full throttle, **higher PWM frequency reduces thrust at low/mid throttle** — changes throttle-curve shape. Compensation given: to match the 24/48kHz linear throttle curve when moving to 96kHz, set `throttle_mid = 1.0` and `throttle_expo = 0.25`.
- [11:52-12:22] Efficiency (RPM at constant 1A/4W): 24kHz→48kHz gives **~25% RPM increase**, and since thrust ∝ RPM², **~55% thrust increase**.
- [12:22-12:39] 48kHz→96kHz gives a smaller **~8% RPM increase**. Auto-caption transcribed the thrust-increase figure as "177%" — this is almost certainly a caption error (8% RPM → ~17% thrust by RPM² scaling, not 177%; math doesn't support 177%). Treat the 96kHz thrust-gain figure as **~17%, not 177%** — flagging the transcript artifact rather than asserting either number as certain.
- [12:53-13:12] Efficiency gain is only present at partial throttle — no difference at or near full throttle.
- [13:14-13:47] Downside: **higher PWM frequency measurably slows motor deceleration/braking** (roughly 10-20ms slower for a 50%→10% throttle step at 96kHz vs 24kHz). Acceleration is not meaningfully affected.
- [13:57-14:07] Creator's overall take: the efficiency gain is "probably worthwhile" even with the braking tradeoff.

## Applied settings (what he actually set on his own whoop, at the end of the video)
- [14:52-14:57] Flashed to **latest Bluejay, 96kHz PWM** (not 48kHz).
- [15:26-15:41] **Minimum AND maximum startup power set to the maximum value** for these tiny motors, explicitly "to make sure they start reliably." (Directly contradicts a "never max out startup power" framing.)
- [15:43-15:48] Motor timing: 15° (matches the bench-test conclusion above).
- [15:48-15:50] **Demag compensation left at LOW** (not raised to High).
- [15:50-16:06] "RPM Power Protection" set to the lowest value that doesn't limit acceleration (he says "three times") — to minimize power spikes.
- [16:16-16:36] Everything else left default; ESC power-rating cell setting (1S vs 1-2S) set to match his board.

## Explicit disclaimers/limits from the creator
- All findings are from a single 0802 motor on a bench rig — no claim of universality across motor sizes/Kv is made in this video.
- No claims at all about Demag compensation physics, desync causes, or numeric startup-power ranges (1100/1200 etc.) appear anywhere in this transcript.
