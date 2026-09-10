# c94e9TCCP8Y — "Demag Compensation Explained - BLHeli_32 ESC"
https://youtu.be/c94e9TCCP8Y
Source: real yt-dlp auto-caption transcript, research/transcripts/c94e9TCCP8Y.clean.txt

## ATTRIBUTION CORRECTION (important — flag for parent)
YouTube oEmbed reports `author_name: "FPV University"` for this video's channel. BUT the speaker
self-identifies at [00:00:00-00:00:01]: "hi i'm pablo spechalski and this is another video in the
esc series about the demo compensation and the bl heli" — this is auto-caption mangling of
"Paweł Spychalski." The speaker IS Pawel Spychalski; "FPV University" is evidently the current
channel brand/name he publishes under (channels get renamed; oEmbed only reports the current
channel display name, not the host's name). The KB's ORIGINAL attribution ("Pawel Spychalski")
was accurate to the actual speaker. A prior fix pass changed the credit to "FPV University" based
solely on the oEmbed channel name — that fix was well-intentioned but incomplete: correct
attribution is "Pawel Spychalski (FPV University channel)," crediting both the person and the
current channel brand.

## Content — Demag compensation, as explained by the speaker (real, verified)
- [00:39-01:01] Motivates the video: says there's very little public documentation on BLHeli_32
  settings beyond "increase timing, increase demag compensation" folklore; the official BLHeli_32
  manual is very short ("like five sentences about almost everything").
- [01:44-02:52] Basic zero-crossing mechanism: a coil generates a magnetic field that pulls the
  rotor magnet toward it, then must release before the magnet passes, or it would "break the whole
  flow." This release point is the zero crossing, detected via back-EMF.
- [03:54-04:12] Zero-crossing detection is a PREDICTION — the coil must be energized slightly
  BEFORE the actual zero crossing, so the ESC predicts the next commutation moment in advance;
  sometimes the prediction is wrong (fires too early or too late).
- [04:21-05:22] Causes of misprediction: general electrical noise, plus the fundamental physics of
  inductance — current lags voltage by ~90° in an inductor, so energizing/de-energizing a coil is
  never instantaneous (explicitly says "current does not follow voltage... lagging by 90 degrees").
- [05:34-06:28] When misprediction happens, symptom is the motor stuttering or stopping ("dashing" =
  auto-caption mangling of "desyncing" throughout this transcript). Most commonly triggered by rapid
  throttle increase (fast acceleration demand).
- [06:28-07:23] What demag compensation actually does, in two parts:
  1. Detects a likely-missed zero crossing and, rather than trusting the current (probably wrong)
     RPM estimate, falls back to the PREVIOUSLY computed RPM/phase-timing value.
  2. Can also SLIGHTLY REDUCE POWER to the coil during the acceleration where the desync risk was
     detected.
- [07:23-08:00] The power reduction amount is set by the Demag parameter itself: Off = no power
  reduction; Low = power cut slightly; High = power cut more. Explicitly: this reduction ONLY
  happens when the ESC thinks a desync is occurring or about to occur — not a constant power cap.
- [08:52-09:01] Mechanism for why reducing power helps: the reduction happens during acceleration,
  so it slightly slows acceleration, which increases the chance that the previous RPM/zero-crossing
  estimate is still valid (less to "catch up" on).
- [09:33-09:52] Explicit recommendation: "if you are not facing any kind of the [desyncs] in flight
  ... leave the demo compensation how it is by default, don't touch it."
- [09:52-10:15] If experiencing desyncs "especially on the low kv and the bigger motors" during
  rapid throttle application — increase demag compensation.
- [11:03-11:06] the general remedy framed as: "increase the demand compensation slightly from off
  to low and from low to high" as needed — i.e. step it up incrementally, not jump straight to High.
- [12:12-12:24] Explicitly says total motor power is NOT reduced overall by raising demag — only
  the power during a detected/predicted desync event is adjusted.

## Cross-check against current KB (video-audio-knowledge.md, blheli32.md)
- KB's "What is Demag?" summary (flyback current, short-circuit spike, zero-crossing blinding) is a
  reasonable paraphrase of the underlying physics but is NOT what this specific video says — this
  video frames demag purely as a PREDICTION-CORRECTION + POWER-BACKOFF mechanism (stick with last
  known-good RPM estimate + slightly reduce acceleration), not a "short circuit current spike"
  explanation. The KB's flyback-current framing is closer to general ESC/demag literature but should
  not be attributed to this video specifically.
- No numeric RPM/percentage figures are stated anywhere in this video (no "~7,000 RPM" claim, no
  specific timing degrees, no specific KV numbers). This video is qualitative/conceptual only.
