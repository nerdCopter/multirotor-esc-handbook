# 7WeHTb7aBrE — "Tuning your ESC (BLHeli_32) to stop desyncs and improve motor performance in your FPV quadcopter!" (Chris Rosser, per YouTube oEmbed author_name; speaker does not self-identify by name in transcript)
https://youtu.be/7WeHTb7aBrE
Source: real yt-dlp auto-caption transcript, research/transcripts/7WeHTb7aBrE.clean.txt

Format: full BLHeliSuite32 walkthrough + settings tutorial, not an interview. Every claim below is
literally stated in the transcript with timestamps.

## Motor architecture background (new/supplementary content, not currently in theory-operation-hardware.md)
- [03:49-04:14] Shows a "12N14P" motor (12 stator windings, 14 rotor magnet poles) as the typical
  mini-quad BLDC outrunner topology.
- [04:34-05:31] Delta winding configuration (vs. star/Y) is standard for mini quads specifically
  because delta gives a HIGHER kV / more RPM than star for the same winding, at the cost of no
  neutral point.
- [08:37-09:29] For this 12N14P (7 pole-pairs) motor, ONE mechanical revolution = SEVEN electrical
  commutation cycles (7 pairs of N/S poles passing the phase per physical rotation). This is a
  specific, checkable number for this exact stator/pole combination — do not generalize to all
  motors without checking pole count.
- [10:39-12:31] Explains the un-driven "floating" phase during each commutation step is used to
  measure back-EMF and detect the zero crossing; if there's electrical noise, this measurement gets
  harder — directly ties noise to desync risk, consistent with the rest of the KB.

## Motor Timing
- [15:02-15:37] Motor timing = firing the phase a number of ELECTRICAL degrees (not mechanical)
  before the theoretical zero-crossing point, "up to 30 or 31 degrees."
- [16:44-17:24] General rule: faster/smaller motors benefit from MORE timing; slower/bigger motors
  benefit from LESS timing.
- [17:24-18:00] Explicit numeric recommendation: "for most mini quads a timing of 23 degrees... is a
  good starting value" — stated as 23° specifically for 5" builds; considering going higher for
  very small quads, and lower for 7"/8"/10" quads.
- [17:40-18:07] Trade-off stated explicitly: MORE timing → fewer desyncs + more top-end power, but
  hotter motors. LESS timing → more desync-prone but more efficient / less power.
- [18:29-19:11] BLHeli32's Auto timing mode exists and works, described as "typically a bit on the
  conservative side" — trades away a little power for safety margin; framed as a reasonable choice
  if you don't mind sacrificing some top-end.

## PWM Frequency
- [19:16-20:16] PWM frequency is the switching rate used to modulate power within each energized
  phase window (e.g. 50% throttle = 50% duty pulse train), separate from the commutation step rate.
- [21:00-21:52] Higher PWM frequency → less torque ripple (smoother) but MORE FET heating (FETs heat
  up most while switching, so more switches/sec = more heat) AND slightly LESS peak/mechanical
  torque (shorter pulses give current less time to build up in the winding inductance per pulse).
- [22:30-23:22] Explicit recommendation: use BLHeli32 32.8's Variable PWM feature — set PWM LOW to
  24kHz (used at zero throttle, maximizes low-end torque) and PWM HIGH to 48kHz (used at full
  throttle, smoothest top end), linearly interpolated by throttle position in between.

## Ramp-up Power (BLHeli_32's name for the acceleration-limit parameter)
- [23:26-24:59] Defined precisely: controls how fast PWM duty cycle is allowed to increase as the
  motor accelerates. Purpose: prevent driving huge current into the windings before the motor has
  had time to spin up and build back-EMF (back-EMF opposes and limits current; without it current
  would be high enough to melt the windings).
- [25:35-26:12] Explicit numeric guidance: default is 50%. States his personal recommended range for
  5" quads is "more around 20 to 30 percent," and personally runs 30% ("i usually set my ramp up
  power at 30 percent").
- [26:38-27:14] Explicit tuning method: minimum settable value is around 3%; increase in 5-10% steps
  from a low starting point until no further improvement in the quad's in-air response is observed,
  then stop (don't keep increasing past the point of no benefit — extra ramp-up power beyond what's
  needed for full torque access is just extra heat, no benefit).
- [30:31-30:44] Confirms personally testing up to 30% on 5" and seeing no benefit above that value.
- [30:44-31:19] For LARGER quads (7"): recommends REDUCING ramp-up power further, to roughly
  "fifteen to twenty percent for a seven inch" — NOTE: the transcript's wording around the 5"-vs-7"
  numbers is somewhat garbled/ambiguous in the auto-captions ("maybe around fifteen to twenty
  percent for a seven inch and maybe twenty five to thirty percent for a 5-inch") — treat the
  specific 15-20% (7") figure as reasonably solid, but the "25-30% for 5-inch" repetition here is
  likely just restating the earlier 20-30%/30% figure, not a new distinct number.
- [31:19-32:34] For SMALLER quads (3" or smaller, "tiny whoops"): states there MAY be a benefit to
  INCREASING ramp-up power ABOVE the 50% default, because small motors build back-EMF much faster
  and can "accept a faster increase" — framed as worth experimenting with, not a confident number.

## Demag Compensation
- [29:09-29:34] Explicit recommendation: "for most quads, default or low should be fine." If
  experiencing desyncs, try switching to High and see if it helps — but "for most mini quads that
  shouldn't be required." Only VERY LARGE motors/quads are called out as more likely to benefit from
  High.
- NOTE: this directly CONTRADICTS the KB's desyncs.md §4.3 ("5-Inch Freestyle... Demag Compensation:
  High for freestyle") as a blanket default — this creator's default guidance for typical 5" builds
  is "low is fine unless you have a problem," not "set High for freestyle as standard practice."

## Sine/Sinusoidal Modulation Mode
- [29:39-30:22] Feature varies PWM duty cycle at low RPM to approximate a sinusoidal drive voltage
  (smoother for large, slow-spinning motors); ESC reverts to normal trapezoidal drive at higher RPM
  since that's more efficient there.
- [30:22-31:10] Explicit recommendation: leave OFF unless running props LARGER than 7" diameter —
  states torque at low RPM is "critical for flight performance" for mini quads and this feature
  slightly reduces it. Strongly advises leaving off for anything 7" or smaller.

## Other settings mentioned (for completeness, minor/lower priority for the KB)
- [30:56-32:56] "Brake on Stop": makes props stop instantly on disarm via active braking rather than
  freewheeling — personal safety preference (won't hit things with a still-spinning prop), trade-off
  is the quad can't fall out of a tree as easily if caught, since props can't freewheel to escape.
- [32:59-33:53] Low Voltage Protection: recommends OFF for multirotors (cutting power mid-flight =
  crash; fine for planes that can glide, not for quads).
- [33:55-35:00]: Current Protection: recommends OFF — short current spikes during speed changes /
  prop strikes are normal and shouldn't trigger a power cut; ramp-up power is the correct tool to
  manage current instead.
- [34:14-34:23] "Non-damped mode": recommends leaving OFF — turning it on disables active braking,
  which the speaker says is important to keep.
- Temperature protection default 140°C called "fine" for typical builds; can lower to 120°C/100°C if
  concerned about ESC overheating.

## Numeric summary table (for quick cross-reference against KB claims)
| Setting | This video's stated recommendation | Context |
|---|---|---|
| Motor Timing | 23° starting point | 5" mini quad, BLHeli32 |
| Timing (large quads) | lower than 23° | 7"/8"/10" |
| PWM (Variable) | 24kHz low / 48kHz high | BLHeli32 32.8 Variable PWM |
| Ramp-up Power | 20-30% (uses 30%) | 5" |
| Ramp-up Power | ~15-20% | 7" (reduce further) |
| Ramp-up Power | possibly >50% default | 3"-or-smaller (experiment, not firm) |
| Demag | default/Low is fine for most | only escalate to High if desyncing, esp. very large motors |
| Sine mode | OFF | unless props >7" |
| Current protection | OFF | ramp-up power is the real current control |
| Non-damped mode | OFF | preserves active braking |
