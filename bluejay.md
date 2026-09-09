# Bluejay (BLHeli_S 8-bit) Technical Reference Guide

Bluejay is an open-source firmware designed for 8-bit Silabs EFM8 microcontrollers (historically running BLHeli_S). Currently supported targets are **EFM8BB21** and **EFM8BB51**; the older **EFM8BB10** ("L" layout) is deprecated, with its last official release at v0.18.1. It modernizes legacy 8-bit hardware by adding **Bidirectional DShot** (e-RPM telemetry), variable PWM frequencies, customizable startup melodies, and optimized commutation routines.

---

## 1. Web Configurator & Official Resources

* **Official Repository:** [GitHub bird-sanctuary/bluejay](https://github.com/bird-sanctuary/bluejay/releases)
* **Web Flasher & Configurator:** [ESC Configurator (esc-configurator.com)](https://esc-configurator.com/)
* **Official Wiki / Setup Guide:** [Bluejay Setup Wiki](https://github.com/bird-sanctuary/bluejay/wiki/Setup)
* **BLHeli_S Migration Guide:** [Migrating from BLHeli_S](https://github.com/bird-sanctuary/bluejay/wiki/Migrating-from-BLHeli_S)
* **Community Discord:** [Bluejay Discord Server](https://discord.gg/ddyzguPB5t)

---

## 2. Core Technical Parameters & Tuning

### PWM Switching Frequency Breakdown
On Bluejay, PWM frequency is chosen in the configurator and flashed — it is **not** a live/variable setting like BLHeli_32's or AM32's variable PWM, so pick it per-build rather than expecting it to adapt automatically.

**The real history here, found in Bluejay's own GitHub issue tracker — this is the primary source, and it's more precise than either "avoid 96kHz" or "96kHz is fine" as a blanket rule:**

* **Bluejay's own release notes for v0.20.0** stated the maintainers' actual technical objection to 96kHz: *"In process of experimenting with dynamic PWM we came to the conclusion that 96kHz mode is pretty much useless and we might get rid of it in the future. The resolution on 96kHz is 256 steps in the best case. Considering deadtime (which needs to be subtracted) — having high dead times reduces this even further."* (quoted in [issue #168](https://github.com/bird-sanctuary/bluejay/issues/168)) — the concern was **duty-cycle resolution** (coarser power-delivery steps at 96kHz, worse on ESCs with higher [dead time](theory-operation-hardware.md#5-dead-time--shoot-through-protection)), not desync risk specifically.
* The reporter on that issue, a whoop pilot, confirms the wider community sentiment at the time: *"I know many/most people recommend against 96khz because of worse flight characteristics"* — while also reporting real anecdotal flight-time gains on his own 65mm/75mm whoops (300mAh: ~3:09→3:20 on 65mm, ~4:30→5:15 on 75mm) that he valued enough to ask for 96kHz to be kept.
* **96kHz was restored, not deprecated**: [PR #154 "bring back 96khz mode"](https://github.com/bird-sanctuary/bluejay/pull/154) was merged, explicitly citing "there seem to be lots of people using it" — the merge also fixed broken active-braking values from the earlier dynamic-PWM work and added a configurator-side 96kHz threshold setting. Current Bluejay firmware's 96kHz mode is this reworked, fixed version, not the original one the maintainers called "pretty much useless."

**Practical takeaway:** on a low-deadtime target, 96kHz's resolution penalty is small and real-world results (Chris Rosser's bench test below) can be positive. On a high-deadtime target, the maintainers' own stated concern applies more — the resolution loss is worse. Check your ESC's deadtime (see the [Startup Power table](#startup-power-motor-idle--rpm-power-protection-official-wiki-data) below) before assuming 96kHz is free efficiency.

* **24kHz:** Maximum active braking authority (damped light), highest low-end torque, and crispest control response. Trade-off: shorter flight times, audible switching whine.
* **48kHz:** Balance of efficiency, quiet operation, and back-EMF detection margin. The safe, widely-used default.
* **96kHz:** Longest flight time; coarser duty-cycle resolution (worse on high-deadtime targets, per the maintainers' own stated reasoning above). A real bench test (Chris Rosser, [EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw)) on an 0802-class whoop motor shows measurable RPM/thrust gains moving 48kHz→96kHz, with the tradeoff in that test being **slower active braking** (~10-20ms). His applied Betaflight compensation for the throttle-curve shift was `throttle_mid = 1.0`, `throttle_expo = 0.25`.

### Startup Power, Motor Idle & RPM Power Protection (Official Wiki Data)
Bluejay's "startup power" (what earlier drafts of this KB called "rampup power") controls how much current is injected during open-loop startup before stable Back-EMF is acquired. **This is not a flat number — it is motor/deadtime-specific, and the project's own [Setup wiki](https://github.com/bird-sanctuary/bluejay/wiki/Setup) publishes an actual recommended-settings table**, reproduced here verbatim:

**Motor voltage above 2S:**

| Motor Voltage | Min/Max Startup Power | Motor Idle |
| :--- | :--- | :--- |
| 4S and above | `1010`/`1020` | `5.5%` |
| 3S | `1020`/`1040` | `5.5%` |

**Motor voltage at or below 2S** — find your ESC's deadtime first (visible in ESC-Configurator's target layout name, format `x_y_nn` where `nn` is the deadtime, e.g. `O_H_5` = deadtime 5):

| Config (deadtime < 30) | Min/Max Startup Power | Motor Idle |
| :--- | :--- | :--- |
| 48kHz 1S 0802 19500Kv, deadtime 5 | `1080`/`1120` | `9%` |
| 48/96kHz 1S 1102 22000Kv, deadtime 5 | `1020`/`1040` | `8%` |
| 48kHz 1S 0702 28500Kv, deadtime 5 | `1040`/`1060` | `8%` |
| 48kHz 1S 0702 32500Kv, deadtime 10 | `1050`/`1070` | `8%` |
| 48kHz 1S 0603 17000Kv, deadtime 20 | `1090`/`1150` | `10%` |
| 48kHz 1S 0802 19500Kv, deadtime **70** | `1125`/`1200` | `16%` |

The project's stated philosophy is the opposite of "find a safe high number": **use the lowest startup power that reliably starts the motor**, because BLHeli_S-based ESCs have no phase-current feedback (open-loop) and can't tell when a stalled motor is drawing damaging current — the startup limit is the only protection. Their tuning method: start from the closest table row, and if motors don't spin, increase both Min/Max by `10`; if they spin reliably, try decreasing by `10` to find the lowest working value; a difference of more than ±10 from the closest matching table row is treated as a sign of a hardware problem worth inspecting, not just "your build is different."

This is a materially different, more precise picture than the flat "Min 1100 / Max 1200-1250" figure earlier drafts of this KB attributed to a YouTube video — that video doesn't contain those numbers (see below), though they happen to land in a plausible range next to the deadtime-5/10 table rows above. Separately, Chris Rosser's own working Tiny Whoop config ([EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw)) runs both values at maximum for reliable starts, which the wiki's own risk model would call unnecessary risk exposure, not a recommended target — use the official table first, and only push higher if your specific motor won't start.

The video previously cited as the source for those flat numbers, Joshua Bardwell's Bluejay setup video ([yEDhnBUFQNI](https://youtu.be/yEDhnBUFQNI)), doesn't contain them — Bardwell says on camera he doesn't know how to tune this ("just raise it until you fry a motor, I really don't [know]"), which is exactly why the official wiki table above is the better reference.

**RPM Power Protection (Rampup):** the Migration wiki gives the real BLHeli_S→Bluejay correspondence: BLHeli_S `0.031`→Bluejay `2x`, up to BLHeli_S `1.50`→Bluejay `13x`. Lower values avoid power spikes but reduce acceleration.

### Whoop Idle & Commutation Stability
* Use the **Motor Idle** values from the official table above (`8%`–`16%` depending on deadtime) rather than a flat whoop-wide number.
* **Betaflight 4.5+ note:** prior to Betaflight 4.5, Dynamic Idle on 1S/2S Bluejay builds was risky — pilots would find motors hard to start and crank up Min/Max Startup Power to compensate, which silently disabled the startup protection described above. [Betaflight PR #12432](https://github.com/betaflight/betaflight/pull/12432) addressed this; the wiki now suggests using the Motor Idle table value as the Dynamic Idle start value. If you're on an older Betaflight version with a 1S/2S Bluejay build, use static idle instead.
* Sourced motor timing data point (Chris Rosser, [EhYKeZfSQIw](https://youtu.be/EhYKeZfSQIw)): **15° static** was empirically optimal on an 0802-class motor — higher settings (22.5°/30°) gave no extra top-end and worse efficiency in his test.

### Extended DShot Telemetry (EDT) & Temperature Protection
* Bluejay implements [EDT](https://github.com/bird-sanctuary/extended-dshot-telemetry), an open extension to bidirectional DShot telemetry, supported by Betaflight 4.4+. Enable with `set dshot_edt = on` in the Betaflight CLI, then `save`.
* Temperature protection was broken prior to Bluejay v0.19; it's fixed as of v0.19 but **disabled by default** (opt-in), because some 1S builds and AIO boards with miscalibrated temperature sensors caused false triggers. To use it: enable the ESC Temperature OSD element, enable EDT via CLI, set the correct power rating (2S+ unless your AIO is explicitly 1S-rated) in ESC-Configurator, and sanity-check the reported temperature is roughly ambient +10–20°C before trusting it.

---

## 3. Bidirectional DShot Implementation on 8-bit Hardware

* Bluejay implements DShot telemetry over the motor signal line using an inverted e-RPM pulse return immediately following the 16-bit DShot command.
* **Best Practice:** Run **DShot300 at a 4.0kHz PID loop** in Betaflight. Running DShot600 on 8-bit EFM8BB21 microcontrollers can saturate MCU interrupt servicing and increase DShot packet error rates.

---

## 4. Curated Video & Community Resources

* **Joshua Bardwell Bluejay Flashing & Setup Guide:** [YouTube Video](https://youtu.be/yEDhnBUFQNI)
* **Chris Rosser, "ULTIMATE ESC Settings for Tiny Whoops!":** [YouTube Video](https://youtu.be/EhYKeZfSQIw) — real bench-tested PWM/timing/startup-power data for an 0802-class whoop, see above.
* **KababFPV, "Tune weird quads - Ramp Up power":** [YouTube Video](https://youtu.be/obObn1oGWmU) — despite the title, this is about a 5.25"/3S build, not a whoop, and contains no numeric startup-power values or back-EMF discussion. Its actual content is qualitative: Ramp Up / RPM Power Protection changes how "tight" the quad feels to the PID loop, with the correct direction varying by ESC brand and by whether the build is over/under-powered for its class — the creator explicitly frames this as layman's understanding, not verified engineering.

---

*Related Knowledge Base Modules:*
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [Protocols & Telemetry](protocols-telemetry.md)
* [PWM Switching & Thermal Dynamics](pwm-frequency-heat.md)
* [Betaflight ESC Integration](betaflight-esc-tuning.md)
* [Back to Knowledge Base Index](README.md)
