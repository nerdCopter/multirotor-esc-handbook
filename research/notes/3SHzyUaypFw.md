# Chris Rosser — "Tuning AM32 ESCs for Ultimate Performance"
https://youtu.be/3SHzyUaypFw

Test rig: Sky Stars km55 ESC, AM32 v2.12, F7 FC logging bidirectional DShot, Tita Robotics 1585 thrust stand.
Motors: 3" = Zing 2404 3800KV / HQProp 3x3x3, 4S 5200mAh @16V. 5" = AOS Supernova 2207 1980KV / HQProp 5x4.5x3 V1S, 7" = AOS Supernova 2807 1440KV / HQProp 7x3.5x3 V1S, both 6S 5200mAh @24V.
Two tests: 0-100% throttle ramp over 10s (thrust curve, efficiency, max thrust/power), and 10%-50% throttle step-and-back (acceleration/deceleration rate).

## Timing (AM32, "1° to 31°" range as shown, plus Auto)
- 3" motor: accel improves 22.5°→15°→7.5°, then negligible 7.5°→0°. Decel: small improvement 22.5°→15°, then almost none below 15°. Thrust/throttle curve: higher timings slightly higher curve, converges at full throttle (not stick-noticeable). Efficiency: no firm winner, all timings very similar, maybe 7.5° very slightly better at high throttle but marginal.
- **3" recommendation: 15° as a safe setting for any 3" quad** (good performance, resilient to desyncs). Racing pilots could try 7.5° for more performance "but you will need to test to make sure you're not getting desyncs."
- 5"/7": demonstrated a **real desync in test data at 0° timing** on the 5" acceleration test — "the ESC is no longer able to accelerate the motor and it just loses control... this is the risk with running zero degrees of timing and that's why I would never recommend that anyone run timings that low." Improvement 22.5°→15°, then no benefit decreasing further.
- **5"/7" recommendation: 15° safe for all quads** (same as 3"), "slightly better performance than 22.5°." 7.5° possible for a bit more performance but "increase the risk of desyncs a bit." **Never use 0°** — "the risk of desyncs is just way too high."

## PWM Frequency (AM32 variable-by-default; ranges given are low-high pairs, top always 2x bottom: 16-32, 24-48, 36-72, 48-96)
- AM32 has variable PWM by default specifically to keep switching frequency away from commutation frequency (avoids "notchy throttle" where motor sticks at certain RPM/throttle).
- 3" motor: acceleration improves 16-32→24-48→36-72kHz (diminishing returns each step, smallest gain 36-72→48-96). Deceleration: opposite direction — improves as frequency drops, diminishing returns below ~36-72kHz range. Thrust and efficiency curves nearly identical across all ranges except very low (16-32kHz) which is less efficient at low throttle.
- **3" recommendation: 36-72kHz range** — "best balance of acceleration and deceleration, the best efficiency."
- 5"/7": same general pattern, but "slightly different tradeoff" favors a lower range.
- **5"/7" recommendation: 24-48kHz range**, and "definitely use the variable pwm just to avoid any risk of that notchy throttle."

## Motor KV setting (AM32-specific, no BLHeli_32 equivalent named directly but compared to rampup power)
- Controls how much throttle AM32 applies to the motor at low RPM — "somewhat similar to ramp up power in BLHeli_32 apart from a lower motor KV setting corresponds to a higher ramp up power."
- Setting KV way too high (e.g. 10,000 for a 3800KV motor) starves the motor of current — it accelerates very slowly and doesn't even reach full RPM (15-16k instead of 19k in his 3" test; 6k instead of ~14k in his 7" 1400KV test at 10,000/8,000 setting).
- Decreasing the KV setting toward the true value improves acceleration "massively." **No benefit decreasing KV setting below the true value** — extra current just becomes heat, no more RPM.
- Has **no effect on deceleration** at all (unless set so high the motor never reaches proper RPM in the first place).
- **Recommendation: set Motor KV to match the actual motor's stated KV; if you can't match exactly, always round to the lower number** (e.g. pick 3780 over 3820 for a 3800KV motor) — guarantees you're never under-driving it.
- **Explicitly flagged as critical for 8"/10" builds with low-KV motors** (900, 800, even 400KV): AM32's default motor KV setting is "around 2000 KV," which is fine for 5"/6"/7" quads but wrong for low-KV 8"/10" builds — those need the setting manually lowered or the motor "may not accelerate properly or as fast as it's able to."

## Not covered / no data
- No mention of Demag Compensation at all in this video.
- No mention of PWM By-RPM specifically (only "variable" is discussed, which for AM32 is inherently RPM-tracking by default per the transcript's own explanation — this may be describing what the KB elsewhere calls "By RPM" under different terminology, or a separate throttle-based variable mode; the transcript's explanation of *why* variable PWM ranges are used — "the reason they do this is to keep the pwm frequency away from the commutation frequency" — matches the KB's existing "By RPM" mechanism description, not the by-throttle one).
