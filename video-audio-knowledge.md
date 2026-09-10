# Video & Audio Source Transcripts

> **Sourcing note:** every claim below is drawn from real auto-generated YouTube captions, downloaded with `yt-dlp` and read in full. Timestamps are approximate (auto-caption drift, ±10-20s). Where a claim below draws a conclusion beyond what a speaker literally says, it's marked **[inferred]**.
>
> Video transcripts aren't the only primary source used in this KB — where a video claim couldn't be confirmed or contradicted one way, project GitHub history (issues, PRs, release notes, official wikis) was checked directly. The clearest example: the "avoid 96kHz on whoops" question wasn't actually resolved by any video — it was resolved by reading Bluejay's own GitHub issue tracker, which turned up the real maintainer reasoning. See [bluejay.md §PWM Switching Frequency Breakdown](bluejay.md#pwm-switching-frequency-breakdown) and video 4 below.

**Coverage:** all 10 individually-cited videos have now been transcribed. The last 2 (`3SHzyUaypFw`, `6gv0_jTEYZM`) hit YouTube rate-limiting for several days before finally succeeding. The two cited playlists (Bardwell/Harrell BLHeli_32 series, High Energy Failures AM32 series) were verified to exist and be topically relevant but not individually transcribed video-by-video.

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

* **PWM frequency by size, stated directly:** 96kHz for tiny whoops/toothpicks, 48kHz for ~3" builds, 24kHz for 5"+ (citing braking-torque loss at higher frequencies on bigger props). Basis for [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown)'s PWM-frequency-by-size guidance, presented there alongside a competing community claim from the original HackMD source — see that file for the full picture.
* **On Startup Min/Max Power, Bardwell explicitly disclaims expertise:** "just raise it until you fry a motor, I really don't [know] [how to tune it]." The specific numbers "Min 1100 / Max 1200-1250," sometimes attributed to this video, are not spoken anywhere in it.
* PWM frequency on 8-bit Bluejay is set at flash time, not adjustable live like BLHeli_32/AM32 variable PWM.
* A "Dithering" setting is mentioned but not elaborated on in the video itself — resolved from a primary source instead: Bluejay's [v0.20.0 release notes](https://github.com/bird-sanctuary/bluejay/issues/84) explain that the dithering implementation "is only applied to every DSHOT package instead of every PWM cycle," that testing showed "little to no difference in having vs. not having dithering" in that implementation, and that the team disabled it by default (later fully deprecated in [PR #143](https://github.com/bird-sanctuary/bluejay/pull/143)) to reclaim CPU cycles for other features.
* The parameter this KB previously called "rampup power" is Bluejay's "RPM Power Protection" in the actual configurator UI — confirmed again by the official wiki, see [bluejay.md](bluejay.md).

---

## 3. Joshua Bardwell — "BLHeli32 ESC's just got a BIG performance upgrade. Here's how to get it." (By-RPM PWM)
[youtu.be/xuQeJA4EGr8](https://youtu.be/xuQeJA4EGr8)

* **BLHeli_32's "By RPM" PWM feature was added in firmware v32.8.3.**
* **Real documented failure mode of the old throttle-based variable PWM:** switching frequency interacting with actual motor RPM can produce mid-throttle mechanical/electrical resonance — described as "jello" — which By-RPM PWM fixes by keying off measured RPM instead of throttle position. Added to [pwm-frequency-heat.md](pwm-frequency-heat.md) and [blheli32.md](blheli32.md).
* The video references a Chris Rosser chart for the actual benchmark numbers rather than presenting its own — see [blheli32.md](blheli32.md).

---

## 4. Chris Rosser — "ULTIMATE ESC Settings for Tiny Whoops!"
[youtu.be/EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw)

This video is sometimes cited as a source for "never max startup power" and "avoid 96kHz" — it does the opposite on camera. The "avoid 96kHz" claim traces to a real source, but not this or any other video: Bluejay's own GitHub issue tracker (see [bluejay.md](bluejay.md#pwm-switching-frequency-breakdown)) shows the maintainers seriously considered removing 96kHz because of reduced PWM duty-cycle resolution (worse on high-dead-time ESCs), then restored it after user pushback backed by real flight-time data. Both things are true at once: this bench test shows 96kHz can be a good tradeoff on the right hardware, and the maintainers' resolution concern is a real, documented reason it isn't free efficiency on every ESC.

* **Startup Min and Max Power both set to maximum** on his 0802-class whoop build, explicitly to ensure reliable starts.
* **Bench data, 24kHz → 48kHz:** ~25% RPM gain, ~55% thrust gain (thrust scales roughly with RPM²). **48kHz → 96kHz:** a further ~8% RPM gain, ~17% thrust gain. (One auto-caption artifact read "177%" for this — that's a transcription error; the actual RPM-to-thrust math for an ~8% RPM gain works out to ~17%, not 177%.)
* **The real cost of 96kHz in his test was slower active braking (~10-20ms), not increased desync rate.** He flashes his own whoop to 96kHz.
* **15° static timing was empirically optimal** for his 0802-class motor — 22.5° and 30° gave no additional top-end and measurably worse efficiency.
* **Demag left at Low** (not High) in his applied settings.
* Betaflight compensation applied for the throttle-curve shift when moving to 96kHz: `throttle_mid = 1.0`, `throttle_expo = 0.25`.

---

## 5. KababFPV — "Tune weird quads - Ramp Up power"
[youtu.be/obObn1oGWmU](https://youtu.be/obObn1oGWmU)

* **Despite sometimes being cited as a source for TinyWhoop startup-power numbers, this video is not about a whoop at all** — it covers a 5.25"/3S build. It contains **zero numeric startup-power values** and **zero discussion of back-EMF or zero-crossing physics.** A "Rampup / Max Startup Trap" narrative and specific 1100/1200-1250 numbers sometimes attributed to this video are not present in it.
* **Actual content (qualitative, not numeric):** Ramp Up Power / RPM Power Protection changes how "tight" a quad feels to the PID loop. Too high causes PID overshoot/flutter; too low causes looseness. The correct direction varies by ESC brand and by whether the build is over- or under-powered for its class.
* The creator explicitly frames this as layman's understanding, not verified engineering — he is not claiming authority on the underlying physics.
* A YouTube comment can't be fetched or audited as a source, so a "see comments by [username]" reference isn't a usable citation for this KB.

---

## 6. Joshua Bardwell & Ryan Harrell — "Stop desyncs on your 160 mph 6S 2700kv quad"
[youtu.be/oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64)

The core "how do I actually fix a desync" reference for this KB.

* **Harrell's #1 recommendation for an actively-desyncing high-RPM 6S build is Demag HIGH** (@19:58), explicitly trading efficiency for stability. This is in tension with a separate racing-context claim that Low/Off Demag recovers RPM — see [desyncs.md §4](desyncs.md) for how both are reconciled.
* **PWM frequency's relationship to desync is not one-directional** (@25:19): Harrell describes having seen both raising and lowering PWM frequency fix the same desync symptom on different builds, including an anecdote where 24kHz fixed one problem on a build post-crash but introduced a different one, reversed by going back to 48kHz.
* **Dead time / shoot-through protection** discussed as a distinct concept from Demag Compensation (@05:14) — added to [theory-operation-hardware.md §5](theory-operation-hardware.md#5-dead-time--shoot-through-protection), since it wasn't in the KB at all before.
* A specific `@21:17` timestamp and the "~7,000 RPM / 15-20% power" Demag-Low racing figure are sometimes attributed to this video — **neither is supported by the actual transcript.** That RPM figure traces instead to the original HackMD source document, attributed there to a community member `@FreedomDuck` — see [desyncs.md §4](desyncs.md) and [blheli32.md](blheli32.md) for the real attribution.

---

## 7. Pawel Spychalski (channel: FPV University) — "Demag Compensation Explained - BLHeli_32 ESC"
[youtu.be/c94e9TCCP8Y](https://youtu.be/c94e9TCCP8Y)

* Speaker self-identifies as **Pawel Spychalski** at 0:00 (auto-caption renders this as "pablo spechalski"); the channel's current display name is **FPV University**. Both are true simultaneously — cited as "Pawel Spychalski (FPV University)" throughout this repository.
* **Real guidance (@09:33): leave Demag at default/Low and only step it up (Off→Low→High) if you're actually experiencing desyncs** — not "default to High for freestyle."
* Demag mechanism is explained as a prediction-fallback-plus-power-backoff behavior — a different framing from this KB's "flyback short-circuit" description, but describing the same underlying phenomenon; not a contradiction, just a different model.

---

## 8. Chris Rosser — "Tuning your ESC (BLHeli_32) to stop desyncs and improve motor performance"
[youtu.be/7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE)

* **23° static timing stated explicitly (@16:44) as the gold standard for 5" builds** — this is the actual source for that figure elsewhere in the KB.
* **Same incremental Demag guidance as Spychalski's video (@29:09):** default/Low unless desyncing, step up as needed — independent confirmation of the same point from a different creator.
* **PWM tradeoff direction confirmed:** lower frequency = more torque, less efficiency (matches [pwm-frequency-heat.md](pwm-frequency-heat.md)).
* **Real numeric Rampup Power guidance by airframe size** — more specific than this KB's earlier flat "30%-50%": 5" ≈ 20%-30% (he uses 30%), 7" ≈ 15%-20%, sub-3" builds possibly >50% by default. Used in [desyncs.md](desyncs.md) and [blheli32.md](blheli32.md).

---

## 9. Chris Rosser — "Tuning AM32 ESCs for Ultimate Performance"
[youtu.be/3SHzyUaypFw](https://youtu.be/3SHzyUaypFw)

* **AM32 PWM frequency is size-dependent, not one universal figure.** Tested across four selectable low-high range pairs (each top exactly 2x its bottom: `16-32`, `24-48`, `36-72`, `48-96` kHz): **3" motor tested best at 36-72kHz** ("best balance of acceleration and deceleration, the best efficiency"); **5"/7" motors tested best at 24-48kHz** ("definitely use the variable pwm just to avoid any risk of that notchy throttle"). 8"/10" builds are not tested for PWM in this video. This is the sourced basis for [am32.md](am32.md)/[desyncs.md](desyncs.md) now splitting the PWM-frequency guidance by firmware *and* by size, instead of treating AM32 as one "Variable/By-RPM" bucket or generalizing the 5"/7" figure to smaller motors — a mistake this KB made once before this transcript was fully read.
* **AM32 timing:** **15°** recommended as a safe universal default across 3"/5"/7" motors (not size-tiered the way BLHeli_32 is treated elsewhere in this KB). **7.5°** usable for racing only if the build stays desync-free at that setting. **0° is demonstrated to desync** in his own test data — the motor loses control entirely under acceleration at 0° timing. This is real *demonstrated-failure* evidence, not just a recommendation to avoid low timing.
* **Genuinely new AM32 setting not previously documented in this KB: "Motor KV."** Functions like BLHeli_32's Rampup Power (lower value = more low-RPM current/torque), but is a distinct field that should be set to the motor's actual KV rating, rounding down if the exact value isn't available. Matters most on low-KV big builds (8"/10", 900/800/400KV motors) — AM32's default assumption is around 2000KV, which under-drives a low-KV motor and causes poor acceleration until corrected.
* **Rampup Power figures independently corroborated:** ~30% conservative on a 5" test motor, ~50% conservative on a 3" test motor — consistent with the different Chris Rosser video ([7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE)) already cited for these figures in desyncs.md/blheli32.md. Rampup showed zero effect on deceleration in his testing, consistent with the KB's framing that it's an acceleration-only axis.

---

## 10. Chris Rosser — "Testing to find the ULTIMATE BLHeli_32 Settings for 3\" and 5\" drones"
[youtu.be/6gv0_jTEYZM](https://youtu.be/6gv0_jTEYZM)

A rigorous thrust-stand comparison — the most direct tested source in this repository for BLHeli_32 PWM frequency, though it's one build under bench conditions, not a survey of real-world practice.

* **BLHeli_32 PWM frequency, in his tested result:** static 24kHz was the best balance of "motor responsiveness and also efficiency" for both his 5" and 3" motors. By-throttle Variable PWM "doesn't offer a lot of benefit" in his direct comparison. He runs 24kHz fixed on his own quads.
* **BLHeli_32's By-RPM range is directly stated, not left unsourced:** "frequency only rises modestly with By-RPM (~24kHz up to ~40kHz max)" — stays within the low-frequency range he found best for braking/efficiency. He only suggests switching to By-RPM if you specifically notice throttle notchiness; otherwise "I always run 24k fixed pwm."
* **Real safety-relevant finding for 3" motors specifically:** above 48kHz fixed PWM, the 3" Sky Stars motor would not accelerate cleanly at all — audibly "jerking," sometimes failing to spin up entirely, with no usable data collected at 96kHz/128kHz. The 5" Supernova didn't show this failure, only reduced acceleration at 128kHz specifically. Braking also degrades monotonically with higher PWM frequency on both motors ("can't really recommend running a fixed PWM much higher than about 48kHz").
* **This tested result is one real data point, not the whole picture for BLHeli_32 5" freestyle.** Widely-used real-world practice also runs static 48kHz for non-race 5" freestyle — smoother/quieter, at a small torque cost versus 24kHz, and not from a formal test. [desyncs.md §3](desyncs.md) and [README.md](README.md) present both — the tested 24kHz result and the widely-used 48kHz default — rather than one answer.
* **BLHeli_32 timing, more granular than this KB's existing "22°-23°" figure:** 5" test motor (Supernova) → **16°** for cruise efficiency, or **24°** for top-end power (minimal extra heat either way — his choice depends on use case, not a single "best" number). 3" test motor (Sky Stars, 12-pole/9-coil) → **24°** outright, a clear benefit across the board.
* **Demag Compensation is not mentioned at all** in this video — some panel settings were tested and showed no measurable effect, but Demag specifically isn't named among them. Neither confirms nor contradicts this KB's existing Demag guidance (sourced elsewhere).

---

*Related Documentation:*
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [BLHeli_32 Guide](blheli32.md) | [Bluejay Guide](bluejay.md) | [AM32 Guide](am32.md)
* [PWM Frequency & Switching Guide](pwm-frequency-heat.md)
* [Theory of Operation & Hardware](theory-operation-hardware.md)
* [Back to Knowledge Base Index](README.md)
