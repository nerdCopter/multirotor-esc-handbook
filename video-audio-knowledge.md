# Video & Audio Source Transcripts

> **Sourcing note:** every claim below is drawn from real auto-generated YouTube captions, downloaded with `yt-dlp` and read in full. Timestamps are approximate (auto-caption drift, ±10-20s). This replaces an earlier version of this document that attributed specific numbers to named creators without any transcript ever having been pulled — several of those numbers turned out not to be in the cited videos at all. Where this document draws a conclusion beyond what a speaker literally says, it's marked **[inferred]**.
>
> Video transcripts aren't the only primary source used in this KB — where a video claim couldn't be confirmed or contradicted one way, project GitHub history (issues, PRs, release notes, official wikis) was checked directly. The clearest example: the "avoid 96kHz on whoops" question wasn't actually resolved by any video — it was resolved by reading Bluejay's own GitHub issue tracker, which turned up the real maintainer reasoning. See [bluejay.md §PWM Switching Frequency Breakdown](bluejay.md#pwm-switching-frequency-breakdown) and video 4 below.

**Coverage:** 8 of the 10 individually-cited videos have been transcribed. 2 Chris Rosser videos (`3SHzyUaypFw` — "Tuning AM32 ESCs for Ultimate Performance", `6gv0_jTEYZM` — "Testing to find the ULTIMATE BLHeli_32 Settings for 3\" and 5\" drones") consistently hit YouTube rate-limiting (HTTP 429) across multiple retries and client types — not yet transcribed. They are **not** used as a source for any claim in this KB. The two cited playlists (Bardwell/Harrell BLHeli_32 series, High Energy Failures AM32 series) were verified to exist and be topically relevant but not individually transcribed video-by-video.

---

## 1. Joshua Bardwell interviews the AM32 creator — "Open source firmware for BLHeli32 ESC | AM32 is here"
[youtu.be/yOeVj6P9PSU](https://youtu.be/yOeVj6P9PSU)

* **"By RPM" PWM mechanism, in the creator's own words:** fixed at 24kHz until commutation frequency reaches roughly **11.5kHz**, then ramps linearly up to 48kHz. Used in [am32.md](am32.md) and [pwm-frequency-heat.md](pwm-frequency-heat.md).
* **PWM-frequency flight-time claims were never conclusively tested**, per the creator directly: "we've never really been able to come up with any conclusive answers... a lot of us haven't done the tests." This is the primary reason [pwm-frequency-heat.md](pwm-frequency-heat.md) treats the specific flight-time percentage figures as unsourced folklore rather than a spec.
* **Low RPM Throttle Protect** differs between firmwares: BLHeli uses a fixed RPM number; AM32 derives it from the motor's pole count and Kv instead.
* RDP-unlock / ST-Link flashing mechanism as described matches [flashing-unbricking-hardware.md](flashing-unbricking-hardware.md) and [blheli32.md](blheli32.md).
* **[inferred]** The AM32 creator's name is spoken but auto-caption spelling is unreliable for a proper noun — not repeated here as fact.

---

## 2. Joshua Bardwell — "I'm flashing BlueJay to all my BLHeli S ESC's and you should too"
[youtu.be/yEDhnBUFQNI](https://youtu.be/yEDhnBUFQNI)

* **PWM frequency by size, stated directly:** 96kHz for tiny whoops/toothpicks, 48kHz for ~3" builds, 24kHz for 5"+ (citing braking-torque loss at higher frequencies on bigger props). This is the opposite size-mapping from what an earlier draft of this KB claimed, and is now the basis for [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown)'s corrected guidance (presented there alongside a competing community claim from the original HackMD source — see that file for the full picture).
* **On Startup Min/Max Power, Bardwell explicitly disclaims expertise:** "just raise it until you fry a motor, I really don't [know] [how to tune it]." This directly undercuts the specific numbers ("Min 1100 / Max 1200-1250") an earlier draft of this KB attributed to this video — those numbers are not spoken anywhere in it.
* PWM frequency on 8-bit Bluejay is set at flash time, not adjustable live like BLHeli_32/AM32 variable PWM.
* A "Dithering" setting is mentioned but not elaborated on in enough detail to document here.
* The parameter this KB previously called "rampup power" is Bluejay's "RPM Power Protection" in the actual configurator UI — confirmed again by the official wiki, see [bluejay.md](bluejay.md).

---

## 3. Joshua Bardwell — "BLHeli32 ESC's just got a BIG performance upgrade. Here's how to get it." (By-RPM PWM)
[youtu.be/xuQeJA4EGr8](https://youtu.be/xuQeJA4EGr8)

* **BLHeli_32's "By RPM" PWM feature was added in firmware v32.8.3.**
* **Real documented failure mode of the old throttle-based variable PWM:** switching frequency interacting with actual motor RPM can produce mid-throttle mechanical/electrical resonance — described as "jello" — which By-RPM PWM fixes by keying off measured RPM instead of throttle position. Added to [pwm-frequency-heat.md](pwm-frequency-heat.md) and [blheli32.md](blheli32.md).
* The video references a Chris Rosser chart for the actual benchmark numbers rather than presenting its own — an earlier draft of this KB mislabeled this video's own content as "Testing & Benchmarks," which it isn't; corrected in [blheli32.md](blheli32.md).

---

## 4. Chris Rosser — "ULTIMATE ESC Settings for Tiny Whoops!"
[youtu.be/EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw)

This is one of the most load-bearing corrections in the KB — the video an earlier draft cited as the source for "never max startup power" and "avoid 96kHz" does the opposite on camera. That in turn raised the question of where "avoid 96kHz" actually came from — the answer turned out to be real, but not from any video: Bluejay's own GitHub issue tracker (see [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown)) shows the maintainers seriously considered removing 96kHz because of reduced PWM duty-cycle resolution (worse on high-dead-time ESCs), then restored it after user pushback backed by real flight-time data. Both things are true at once: Rosser's bench test shows 96kHz can be a good tradeoff on the right hardware, and the maintainers' resolution concern is a real, documented reason it isn't free efficiency on every ESC.

* **Startup Min and Max Power both set to maximum** on his 0802-class whoop build, explicitly to ensure reliable starts.
* **Bench data, 24kHz → 48kHz:** ~25% RPM gain, ~55% thrust gain (thrust scales roughly with RPM²). **48kHz → 96kHz:** a further ~8% RPM gain, ~17% thrust gain. (One auto-caption artifact read "177%" for this — that's a transcription error; the actual RPM-to-thrust math for an ~8% RPM gain works out to ~17%, not 177%.)
* **The real cost of 96kHz in his test was slower active braking (~10-20ms), not increased desync rate.** He flashes his own whoop to 96kHz.
* **15° static timing was empirically optimal** for his 0802-class motor — 22.5° and 30° gave no additional top-end and measurably worse efficiency.
* **Demag left at Low** (not High) in his applied settings.
* Betaflight compensation applied for the throttle-curve shift when moving to 96kHz: `throttle_mid = 1.0`, `throttle_expo = 0.25`.

---

## 5. KababFPV — "Tune weird quads - Ramp Up power"
[youtu.be/obObn1oGWmU](https://youtu.be/obObn1oGWmU)

* **Despite its citation in an earlier draft as a source for TinyWhoop startup-power numbers, this video is not about a whoop at all** — it covers a 5.25"/3S build. It contains **zero numeric startup-power values** and **zero discussion of back-EMF or zero-crossing physics.** The entire "Rampup / Max Startup Trap" narrative and the specific 1100/1200-1250 numbers previously attributed to this video are not present in it.
* **Actual content (qualitative, not numeric):** Ramp Up Power / RPM Power Protection changes how "tight" a quad feels to the PID loop. Too high causes PID overshoot/flutter; too low causes looseness. The correct direction varies by ESC brand and by whether the build is over- or under-powered for its class.
* The creator explicitly frames this as layman's understanding, not verified engineering — he is not claiming authority on the underlying physics.
* The "(See technical breakdown in comments by Mr.ShutterBug)" citation from an earlier draft has been removed — a YouTube comment can't be fetched or audited as a source.

---

## 6. Joshua Bardwell & Ryan Harrell — "Stop desyncs on your 160 mph 6S 2700kv quad"
[youtu.be/oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64)

The core "how do I actually fix a desync" reference for this KB.

* **Harrell's #1 recommendation for an actively-desyncing high-RPM 6S build is Demag HIGH** (@19:58), explicitly trading efficiency for stability. This directly contradicts an earlier draft's framing that Low/Off Demag was the go-to racing move — see [desyncs.md §4](desyncs.md) for how that tension is now presented.
* **PWM frequency's relationship to desync is not one-directional** (@25:19): Harrell describes having seen both raising and lowering PWM frequency fix the same desync symptom on different builds, including an anecdote where 24kHz fixed one problem on a build post-crash but introduced a different one, reversed by going back to 48kHz.
* **Dead time / shoot-through protection** discussed as a distinct concept from Demag Compensation (@05:14) — added to [theory-operation-hardware.md §5](theory-operation-hardware.md#5-dead-time--shoot-through-protection), since it wasn't in the KB at all before.
* An earlier draft cited a specific `@21:17` timestamp for this video's key claim and also stated it as the source for the "~7,000 RPM / 15-20% power" Demag-Low racing figure — **neither is supported by the actual transcript.** That RPM figure traces instead to the original HackMD source document, attributed there to a community member `@FreedomDuck` — see [desyncs.md §4](desyncs.md) and [blheli32.md](blheli32.md) for the corrected attribution.

---

## 7. Pawel Spychalski (channel: FPV University) — "Demag Compensation Explained - BLHeli_32 ESC"
[youtu.be/c94e9TCCP8Y](https://youtu.be/c94e9TCCP8Y)

* Speaker self-identifies as **Pawel Spychalski** at 0:00 (auto-caption renders this as "pablo spechalski"); his channel's current display name is **FPV University**. An earlier draft of this KB had this exactly backwards in one place (crediting "Pawel Spychalski" where the actual channel-lookup said "FPV University," without realizing both are true) — corrected to "Pawel Spychalski (FPV University)" throughout.
* **Real guidance (@09:33): leave Demag at default/Low and only step it up (Off→Low→High) if you're actually experiencing desyncs** — not "default to High for freestyle," which an earlier draft of this KB stated as the baseline recommendation.
* Demag mechanism is explained as a prediction-fallback-plus-power-backoff behavior — a different framing from this KB's "flyback short-circuit" description, but describing the same underlying phenomenon; not a contradiction, just a different model.

---

## 8. Chris Rosser — "Tuning your ESC (BLHeli_32) to stop desyncs and improve motor performance"
[youtu.be/7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE)

* **23° static timing stated explicitly (@16:44) as the gold standard for 5" builds** — this is the actual source for that figure elsewhere in the KB.
* **Same incremental Demag guidance as Spychalski's video (@29:09):** default/Low unless desyncing, step up as needed — independent confirmation of the same point from a different creator.
* **PWM tradeoff direction confirmed:** lower frequency = more torque, less efficiency (matches [pwm-frequency-heat.md](pwm-frequency-heat.md)).
* **Real numeric Rampup Power guidance by airframe size** — more specific than this KB's earlier flat "30%-50%": 5" ≈ 20%-30% (he uses 30%), 7" ≈ 15%-20%, sub-3" builds possibly >50% by default. Used in [desyncs.md](desyncs.md) and [blheli32.md](blheli32.md).

---

*Related Documentation:*
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [BLHeli_32 Guide](blheli32.md) | [Bluejay Guide](bluejay.md) | [AM32 Guide](am32.md)
* [PWM Frequency & Switching Guide](pwm-frequency-heat.md)
* [Theory of Operation & Hardware](theory-operation-hardware.md)
* [Back to Knowledge Base Index](README.md)
