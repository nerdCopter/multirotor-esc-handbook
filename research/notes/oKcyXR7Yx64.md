# oKcyXR7Yx64 — Joshua Bardwell & Ryan Harrell (miniquadtestbench)
"Stop desyncs on your '160 mph' 6S 2700kv quad (with Ryan Harrell of miniquadtestbench)"
https://youtu.be/oKcyXR7Yx64
Source: real yt-dlp auto-caption transcript, research/transcripts/oKcyXR7Yx64.clean.txt

Format: interview/podcast, Joshua Bardwell hosting Ryan Harrell (miniquadtestbench.com). Context: pilots running 6S 2700kV motors reported quads "falling out of the air" above 70% throttle.

## Desync cause explanation (Ryan Harrell)
- [00:00:39-01:05] Setup: 6S + 2700kV → very high RPM. Problem framed by a pilot report of desync above 70% throttle.
- [01:34-02:11] Higher RPM = ESC has less time to detect zero crossings. Also higher current transition spikes ("current delta") than lower-kV motors, because "kV is per... torque constant... how much current does it cost me to make torque" — rapid RPM changes demand a lot of torque = a lot of current through the coils.
- [02:34-03:04] Combined effect: higher electrical noise → harder to detect the zero crossing accurately → ESC can't tell when to fire the next phase → guesses / eventually desyncs.
- [03:08-03:26] Merry-go-round analogy for desync: grabbing the next bar exactly as it arrives (correct commutation) vs. sticking your arm out and the bar crashing into it (desync).
- [09:59-10:09]: confirms "higher rpms results in more sensitivity to mistiming / missing the zero crossing... and also increases noise... which makes it more difficult to detect the zero crossing."
- [17:38-17:57] Bardwell asks whether a 6S 2700kV motor "scaled down" via a lower throttle cap is equivalent to running a lower-kV motor at 4S — Harrell says it "kind of works" but isn't equivalent because of torque-constant differences; NOT a full substitute.

## Dead time / shoot-through (NEW CONCEPT — not currently in the KB at all)
- [05:14-06:51] "Dead time" = the delay the ESC firmware inserts between switching off the high-side FET of a half-bridge and switching on the low-side (or vice versa), to prevent both being on simultaneously ("shoot through" = a short circuit through the FETs that "fries" the ESC, as opposed to a desync's short circuit through the motor windings).
- [07:00-08:00] Dead time value is normally supplied to firmware devs by the ESC/FET manufacturer; historically at least one case existed where a manufacturer's stated dead time didn't match what the hardware/firmware actually used, causing problems after a firmware upgrade (told anecdotally, no specifics/citation given — DO NOT present as a general firmware bug pattern).
- [08:31-08:53] On BLHeli_S-era hardware, dead time was configurable via a "GH" parameter (dead time multiplier, historical/deprecated — Harrell frames this as "back in the day").
- [09:04-09:33] Component variance (cheap SMD/ceramic caps with wide tolerance) can push actual dead time out of spec on individual units, increasing either desync-adjacent noise or shoot-through risk. This is a per-unit manufacturing variance point, not a general firmware setting.

NOTE: dead time / shoot-through is a DISTINCT concept from demag compensation. It is not mentioned as a user-adjustable setting in modern (BLHeli_32/AM32) firmware in this transcript — treat any claim that a user can tune "dead time" on current firmware as unverified/likely not applicable.

## Explicit tuning recommendations for high-kV 6S builds (Ryan Harrell, ~19:00-25:00)
- [19:58-20:11] "the higher the kV, the best shot is [to] turn demag to high, number one" — DEMAG HIGH is the #1 recommendation for this scenario.
- [20:11-20:26] Explicitly frames this as a trade-off: "you may have some performance losses... we're trying to back off the instantaneous acceleration... reducing those current limits, reducing the noise factor."
- [20:28-20:52] Second recommendation: "fixing timing at between 23 and 25 degrees" — removes CPU overhead of the auto-timing calc, and running MORE timing advance = less likely to desync but costs a "tiny bit of efficiency." (Restated at 24:59-25:02 as "23-25 degrees timing" after an initial slip to "22 23" — the 23-25° figure is the one he settles on and repeats.)
- [21:00-21:10] "if you're running 6s 2,700 kV, efficiency is really not your priority" — framing for why timing/demag losses are acceptable here.
- [21:14-23:17] Third and "most interesting" recommendation: lower Rampup Power ("ramp up power," formerly called "startup power" in older BLHeli versions). Definition: "the maximum rate that it allows the ESC to change the PWM duty cycle" — i.e. how fast duty cycle can jump, which controls the current dump into the coils during a throttle step. Tuning method described: back it off until you notice a performance difference, then increase slightly until that difference disappears.
- [24:26-24:34] Default ramp-up power value stated as "50%" ("Ampa" in the raw transcript — near-certainly a mis-heard/garbled auto-caption of "ramp up power," not a firmware name — do not treat "Ampa" as a real term). He personally starts by dropping it straight to 25%, and usually doesn't notice a difference at that level.
- [24:53-25:07] Summary given by Bardwell, confirmed by Harrell: "Demag high, timing 23-25 degrees, ramp up power reduce — start by knocking it down to 25%, maybe go further."
- [34:20-34:33] Advises the OPPOSITE for very small motors (e.g. 1507-size): may actually benefit from INCREASING ramp up power beyond default rather than decreasing it, since larger ramp-up on a small/light rotor may not cause the same current-dump problem. Not a strong claim ("there may be reasons to go the other way... I have been played with it" — Harrell is speculative/hedged here).

## PWM frequency — explicitly NOT a one-directional recommendation
- [25:19-27:15] Bardwell asks directly whether changing PWM frequency helps with desyncs. Harrell's answer: "it's a really tricky one... in some cases you'll see dropping to 24 kilohertz solve some problems... you'll see the same problem solved by going 48 kilohertz." Explicitly build/motor/ESC-dependent, not a fixed rule.
- [26:07-26:43] Anecdote: a build with low-throttle vibration was fixed by dropping to 24kHz; after a crash, the same quad became unstable ("wanted to take off to the moon") at 24kHz, and going BACK to 48kHz fixed the post-crash instability. Used to illustrate that PWM frequency effects are situational, not a fixed "lower is safer" rule.
- [26:48-27:15] Mentions that around Betaflight 4.2, some devs were recommending 48kHz users drop to 24kHz because lower PWM gives more low-throttle torque — historical, versioned context (Betaflight 4.2-era), not presented as current universal guidance.
- [40:19-42:20] Later: general RPM-filter-driven PID retuning discussion (P-gain increases enabled by better RPM filtering) — tangential to ESC settings, more about flight-controller tuning; not core desync-fix content.

## Explicitly NOT present in this transcript
- No mention anywhere of "~7,000 RPM" or "15-20% power" recovered from setting Demag to Low/Off. This entire video's core recommendation for high-kV/high-RPM desync-prone builds is DEMAG HIGH, not low/off — the opposite framing from the KB's current "Racing Optimization: Demag Low/Off recovers ~7,000 RPM" claim.
- No specific timestamp "@21:17" content matches any single standalone claim — the real content spans roughly 19:58-25:07 as a connected recommendation block (demag high + timing 23-25° + lower ramp-up power together), not one isolated quote.
