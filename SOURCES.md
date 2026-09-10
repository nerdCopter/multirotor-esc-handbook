# Sources

Every URL that backs a claim anywhere in this knowledge base, plus additional sources found during research that aren't directly cited in the prose yet. This file exists so nothing in the KB traces only to "compiled from a HackMD page" — every real underlying source is listed here directly.

---

## 1. Video Sources (YouTube)

Real captions were downloaded and read in full for all 16 of these. Raw transcripts are tracked under [research/transcripts/](research/transcripts/) — see [TAKEDOWN.md](TAKEDOWN.md) for the removal policy.

**On recency:** publish dates below are verified directly from YouTube (`yt-dlp --print upload_date`), not assumed. The original 10-video set spans 2020-2024; a further 6 Chris Rosser videos (capacitor/TVS testing, ESC bench comparisons, FETtec SFOC) span 2024-03 to 2026-03 and are the newest sources in the KB. Where an older and newer source appear to disagree, the newer one is not automatically correct, but it reflects testing done against more recent firmware/hardware and should be weighted accordingly rather than treated as equally-aged with the rest.

| Video | Creator | Published | Status | Cited in |
| :--- | :--- | :--- | :--- | :--- |
| [youtu.be/oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) — "Stop desyncs on your 160 mph 6S 2700kv quad" | Joshua Bardwell w/ Ryan Harrell | 2020-06-15 | Transcribed | desyncs.md, blheli32.md, theory-operation-hardware.md, video-audio-knowledge.md |
| [youtu.be/c94e9TCCP8Y](https://youtu.be/c94e9TCCP8Y) — "Demag Compensation Explained - BLHeli_32 ESC" | Pawel Spychalski (channel: FPV University) | 2020-10-05 | Transcribed | desyncs.md, blheli32.md, video-audio-knowledge.md |
| [youtu.be/yOeVj6P9PSU](https://youtu.be/yOeVj6P9PSU) — "Open source firmware for BLHeli32 ESC \| AM32 is here" | Joshua Bardwell | 2021-03-13 | Transcribed | am32.md, pwm-frequency-heat.md, video-audio-knowledge.md |
| [youtu.be/obObn1oGWmU](https://youtu.be/obObn1oGWmU) — "Tune weird quads - Ramp Up power" | KababFPV | 2021-03-27 | Transcribed | bluejay.md, video-audio-knowledge.md |
| [youtu.be/7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE) — "Tuning your ESC (BLHeli_32) to stop desyncs..." | Chris Rosser | 2021-08-06 | Transcribed | desyncs.md, blheli32.md, video-audio-knowledge.md |
| [youtu.be/yEDhnBUFQNI](https://youtu.be/yEDhnBUFQNI) — "I'm flashing BlueJay to all my BLHeli S ESC's..." | Joshua Bardwell | 2022-01-10 | Transcribed | bluejay.md, desyncs.md, video-audio-knowledge.md |
| [youtu.be/xuQeJA4EGr8](https://youtu.be/xuQeJA4EGr8) — "BLHeli32 ESC's just got a BIG performance upgrade..." | Joshua Bardwell | 2022-02-16 | Transcribed | blheli32.md, pwm-frequency-heat.md, video-audio-knowledge.md |
| [youtu.be/6gv0_jTEYZM](https://youtu.be/6gv0_jTEYZM) — "Testing to find the ULTIMATE BLHeli_32 Settings for 3\" and 5\" drones" | Chris Rosser | **2024-05-31** | Transcribed (took several days of retries past YouTube rate-limiting) | blheli32.md, desyncs.md, README.md, video-audio-knowledge.md |
| [youtu.be/3SHzyUaypFw](https://youtu.be/3SHzyUaypFw) — "Tuning AM32 ESCs for Ultimate Performance" | Chris Rosser | **2024-07-12** | Transcribed (took several days of retries past YouTube rate-limiting) | am32.md, desyncs.md, video-audio-knowledge.md |
| [youtu.be/EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw) — "ULTIMATE ESC Settings for Tiny Whoops!" | Chris Rosser | **2024-08-02** | Transcribed | bluejay.md, desyncs.md, video-audio-knowledge.md |
| [youtu.be/QENbsI3swCI](https://youtu.be/QENbsI3swCI) — "Testing FETTECs new PROTOTYPE ESC against BLHeli_32 AM_32 and BlueJay" | Chris Rosser | 2024-03-08 | Transcribed | foc-vs-trapezoidal.md, firmware-comparison.md, video-audio-knowledge.md |
| [youtu.be/M4NOIwdSBOc](https://youtu.be/M4NOIwdSBOc) — "Motor Commutation Explained: Featuring FETTECs new SFOC approach" | Chris Rosser | 2024-03-13 | Transcribed | foc-vs-trapezoidal.md, video-audio-knowledge.md |
| [youtu.be/VgHOHcWu7-U](https://youtu.be/VgHOHcWu7-U) — "Protect Yourself from ESC Voltage Spikes: Testing Capacitors and TVS Diodes" | Chris Rosser | 2025-06-06 | Transcribed | theory-operation-hardware.md, video-audio-knowledge.md |
| [youtu.be/mdM4jEWOqx4](https://youtu.be/mdM4jEWOqx4) — "Don't buy another ESC Until You See These 150°C Torture Test Results" | Chris Rosser | **2026-01-24** | Transcribed | firmware-comparison.md, video-audio-knowledge.md |
| [youtu.be/ANq7a2S0Gik](https://youtu.be/ANq7a2S0Gik) — "Stop Killing ESCs: The Ultimate Capacitor Buying Guide" | Chris Rosser | **2026-03-10** | Transcribed | theory-operation-hardware.md, video-audio-knowledge.md |
| [youtu.be/pH4K3ErugW4](https://youtu.be/pH4K3ErugW4) — "Stop Buying these ESCs: 30+ ESCs Torture-Tested" | Chris Rosser | **2026-03-20** | Transcribed | firmware-comparison.md, video-audio-knowledge.md |

**Playlists** (verified to exist and be topically relevant; not transcribed video-by-video):
* [Bardwell/Harrell BLHeli_32 series](https://www.youtube.com/playlist?list=PLwoDb7WF6c8kXOyPdBog1wtRcxnXMasUb)
* [High Energy Failures — AM32 Settings Explained](https://www.youtube.com/playlist?list=PLrxUiYCo7HwrIz1uE0aIfNT4GUgVMwc1t)

---

## 2. Official Firmware Repositories, Wikis & Configurators

* **AM32:** [am32-firmware/AM32](https://github.com/am32-firmware/AM32) (current org) · [AlkaMotors/AM32-MultiRotor-ESC-firmware](https://github.com/AlkaMotors/AM32-MultiRotor-ESC-firmware) (legacy/archived, same project) · [wiki.am32.ca](https://wiki.am32.ca/) · [ESC Settings Explained wiki page](https://github.com/AlkaMotors/AM32-MultiRotor-ESC-firmware/wiki/ESC-Settings-Explained) · [am32.ca configurator](https://am32.ca/)
* **Bluejay:** [bird-sanctuary/bluejay releases](https://github.com/bird-sanctuary/bluejay/releases) · [Setup wiki](https://github.com/bird-sanctuary/bluejay/wiki/Setup) (source of the official Startup Power/Motor Idle table) · [Migrating from BLHeli_S wiki](https://github.com/bird-sanctuary/bluejay/wiki/Migrating-from-BLHeli_S) · [Extended DShot Telemetry](https://github.com/bird-sanctuary/extended-dshot-telemetry)
* **BLHeli_32 / BLHeli_S:** [bitdump/BLHeli releases](https://github.com/bitdump/BLHeli/releases) (community-maintained release archive; BLHeli_32 itself is closed-source)
* **ESCape32:** [neoxic/ESCape32](https://github.com/neoxic/ESCape32) · [Targets wiki page](https://github.com/neoxic/ESCape32/wiki/Targets)
* **Betaflight:** [dshot_command.h](https://github.com/betaflight/betaflight/blob/master/src/main/drivers/dshot_command.h) (DShot special-command enum, master branch — may drift; re-check against your Betaflight version) · [PR #12432](https://github.com/betaflight/betaflight/pull/12432) (1S/2S Dynamic Idle fix) · [official Dynamic Idle docs](https://betaflight.com/docs/wiki/guides/current/Dynamic-Idle)
* **Configurator:** [esc-configurator.com](https://esc-configurator.com/) (Bluejay/AM32/BLHeli_S web flasher)

## 3. Specific GitHub Issues/PRs Cited as Primary Sources

These are the exact discussions that resolved specific disputed claims in this KB — see the linked files for the full story.

* [bird-sanctuary/bluejay#84](https://github.com/bird-sanctuary/bluejay/issues/84) — "Release Notes v0.20.0": the actual **primary source** of the maintainers' 96kHz reasoning (PWM resolution, not desync risk) and of the real explanation of the "Dithering" setting. Cited in bluejay.md, video-audio-knowledge.md.
* [bird-sanctuary/bluejay#145](https://github.com/bird-sanctuary/bluejay/pull/145) — merged PR that actually removed 96kHz the first time, restating the same deadtime/resolution reasoning independently. Cited in bluejay.md.
* [bird-sanctuary/bluejay#143](https://github.com/bird-sanctuary/bluejay/pull/143) — merged PR that fully deprecated dithering, consistent with #84. Cited in video-audio-knowledge.md.
* [bird-sanctuary/bluejay#168](https://github.com/bird-sanctuary/bluejay/issues/168) — "please preserve 96khz pwm mode": real user-reported flight-time data and community pushback that got 96kHz restored. Cited in bluejay.md, desyncs.md, pwm-frequency-heat.md.
* [bird-sanctuary/bluejay#154](https://github.com/bird-sanctuary/bluejay/pull/154) — merged PR that restored 96kHz mode with braking-value fixes. Cited alongside #168.

## 4. Editorial/News Sources

* [QuadMeUp — "BLHeli_32 is dead, killed by the war in Ukraine"](https://blog.quadmeup.com/2024/05/31/blheli_32-is-dead-killed-by-the-war-in-ukraine/)
* [Oscar Liang — "The End of BLHeli_32 and The Future of ESC Firmware in FPV Drones"](https://oscarliang.com/end-of-blheli_32/)
* [Hackaday — "The End Of BLHeli_32: Long Live AM32?"](https://hackaday.com/2024/06/07/the-end-of-blheli_32-long-live-am32/)

All three independently corroborate the same underlying fact (BLHeli AS wound down BLHeli_32 in mid-2024 over export-control/sanctions concerns tied to the war in Ukraine), cited in blheli32.md.

## 5. Community Discord Servers (linked as resources, not used as a citation source — Discord history isn't fetchable/auditable)

* [AM32 Discord](https://discord.gg/h7ddYMmEVV)
* [Bluejay Discord](https://discord.gg/ddyzguPB5t)
* [ESCape32 Discord](https://discord.gg/aed6xdSM5Y)

## 6. Original Compilation Source

* [HackMD: "FAQ / ESC tuning resources"](https://hackmd.io/6meEOax2T-KuzpujxHHSMw) — the page this whole KB started from. Kept as a real source (it's where the `@FreedomDuck`-attributed Demag/RPM claim and several other community notes originate), not as a substitute for the primary sources above.

---

## 7. Surfaced During Research, Not Yet Cited In-Text

Found while verifying the above, directly relevant, but not currently referenced by any file's prose — flagging per request rather than silently working them in:

* [bird-sanctuary/bluejay#184](https://github.com/bird-sanctuary/bluejay/pull/184) — open, unmerged PR proposing a throttle-tiered 24/48/96kHz mode; background context, not a shipped feature.
* [bird-sanctuary/bluejay#166](https://github.com/bird-sanctuary/bluejay/pull/166) — "Remove power limits after initial run phase" — potentially relevant to the Startup Power discussion in bluejay.md; not yet reviewed for content.
* ST Community — [OpenOCD and RDP protection [STM32g0x]](https://community.st.com/t5/stm32-mcus-products/openocd-and-rdp-protection-stm32g0x/td-p/776868) — the thread the corrected OpenOCD RDP command in flashing-unbricking-hardware.md is drawn from.

---

*Related: [Video & Audio Source Transcripts](video-audio-knowledge.md) · [Back to Knowledge Base Index](README.md)*
