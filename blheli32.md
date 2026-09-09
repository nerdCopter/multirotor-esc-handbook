# BLHeli_32 Technical Reference Guide

BLHeli_32 is a third-generation, 32-bit closed-source ESC firmware designed for ARM Cortex-M0/M4 (STM32F051, STM32G071, AT32F421) microcontrollers. It pioneered bidirectional DShot, variable PWM switching rates, integrated current/temperature telemetry, and programmable commutation timing.

---

## 1. Official Resources

* **Community-maintained release archive:** [GitHub bitdump/BLHeli](https://github.com/bitdump/BLHeli/releases) — BLHeli_32/BLHeli_S hex releases (BLHeli_32 itself is closed-source; this is the standard distribution point for compiled releases and configurator files, not a source repo).

---

## 2. Core Configuration Parameters & Engineering Context

### Motor Timing Advance
* **Parameter Range:** `Auto` or `1°` to `31°` (Default: `Auto` or `16°`).
* **Physics:** Advances the phase commutation angle ahead of the magnetic neutral point to compensate for stator coil current rise time ($L/R$ time constant).
* **Tuning Recommendations:**
  * **Auto:** Dynamic algorithm adjusts timing based on estimated load. While efficient in steady state, aggressive throttle steps (0% to 100% punches) can outpace the algorithm, causing desyncs.
  * **Static 22°–23°:** The gold standard for 5" freestyle and racing builds. Completely eliminates tracking lag during violent punchouts.
  * **Static 15°–18°:** Essential for large, high-inductance stators (2806.5 to 3115 on 7"–10" builds) to avoid magnetic core saturation.

### Demagnetization Compensation (Demag)
* **Parameter Range:** `Off`, `Low`, `High`.
* **Physics:** When a phase is turned off, its magnetic field collapses, driving inductive flyback current. Demag compensation delays energizing the subsequent phase until this current decays below a safe threshold.
* **Tuning Recommendations (sourced, see [desyncs.md](desyncs.md#3-master-settings-matrix-across-all-esc-firmwares)):**
  * **Start at Low (default) and only step up toward High if you're actually experiencing desyncs.** This is the creators' own stated approach (Pawel Spychalski, [c94e9TCCP8Y](https://youtu.be/c94e9TCCP8Y) @09:33; Chris Rosser, [7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE) @29:09) — not "default to High."
  * **High:** The real sourced fix for an actual punchout/high-RPM desync (Ryan Harrell, [oKcyXR7Yx64](https://youtu.be/oKcyXR7Yx64) @19:58 — his #1 recommendation, trading efficiency for stability), not just for "motors prone to desyncing" in the abstract.
  * **Off / Low (Racing RPM gain):** attributed in the original community resource this KB was built from to a named contributor: *"Demag compensation low saves roughly 7K RPM (20%) on high kv motors... off versus low has minimal effect, some prefer OFF. Thanks to @FreedomDuck for this information. (Ultralight 6S 2100kv to 2150kv.)"* This is a specific, attributed community report — not from any of the cited videos, and not independently re-tested here. It's also in tension with Harrell's High-for-desync guidance above: only try this on a clean, non-desyncing racing build, and don't use it as a desync fix.

### PWM Switching Frequency & Variable PWM
* **Range:** `16kHz` to `128kHz` (or Low/High variable bounds, e.g. `24kHz–48kHz`).
* **Legacy Variable PWM (by throttle):** continuously increases switching frequency with throttle stick position (24kHz at low throttle for braking/low-end torque, up to 48/96kHz at high throttle). Sourced failure mode (Joshua Bardwell, [xuQeJA4EGr8](https://youtu.be/xuQeJA4EGr8)): fixed-by-throttle PWM can interact with actual motor RPM to create mid-throttle mechanical/electrical resonance ("jello"/harmonics) at specific throttle bands.
* **"By RPM" PWM:** added in **BLHeli_32 firmware v32.8.3** (sourced, [xuQeJA4EGr8](https://youtu.be/xuQeJA4EGr8)) — switches frequency based on actual measured motor RPM instead of throttle position, which is what fixes the resonance issue above. See [PWM Frequency & Switching Guide](pwm-frequency-heat.md).

### Rampup Power & Acceleration Limits
* **Parameter Range:** `10%` to `150%` (Default: `50%` or `100%`).
* **Physics:** Restricts maximum instantaneous current slew rate during throttle transients.
* **Tuning Recommendation, sourced by airframe size** (Chris Rosser, [7WeHTb7aBrE](https://youtu.be/7WeHTb7aBrE)): **5" ≈ `20%–30%`** (he uses 30%), **7" ≈ `15%–20%`**, **sub-3" possibly `>50%`** default — not a flat `30%–50%` for every size.

---

## 3. Converting & Unlocking BLHeli_32 Boards to Open-Source (AM32 / ESCape32)

BLHeli AS wound down BLHeli_32 licensing and development in mid-2024, citing illegal use of the firmware in sanctioned countries during the war in Ukraine ([QuadMeUp: "BLHeli_32 is dead, killed by the war in Ukraine"](https://blog.quadmeup.com/2024/05/31/blheli_32-is-dead-killed-by-the-war-in-ukraine/); also reported by [Oscar Liang](https://oscarliang.com/end-of-blheli_32/) and [Hackaday](https://hackaday.com/2024/06/07/the-end-of-blheli_32-long-live-am32/)). Existing ESCs keep working, but no new licenses or updates are issued. Locked boards can be converted to open-source [AM32](am32.md) or [ESCape32](escape32.md) firmware via SWD flashing and a Readout Protection unlock — **the actual commands, tool syntax, and hardware pin details live in [ESC Flashing & Unbricking Guide](flashing-unbricking-hardware.md) only**, to avoid the two copies drifting out of sync.

---

## 4. Curated Video & Audio Resources

* **Joshua Bardwell & Ryan Harrell BLHeli_32 Deep Series:** [YouTube Playlist](https://www.youtube.com/playlist?list=PLwoDb7WF6c8kXOyPdBog1wtRcxnXMasUb)
* **Bardwell / Harrell Desync & Demag Deep Dive:** [YouTube Video](https://youtu.be/oKcyXR7Yx64) (the earlier `@21:17` timestamp claim was unverified and has been removed; his real Demag-High recommendation for this scenario is at @19:58).
* **Pawel Spychalski (FPV University), "Demag Compensation Explained":** [YouTube Video](https://youtu.be/c94e9TCCP8Y) — speaker self-identifies at 0:00 as Pawel Spychalski; his channel is currently branded "FPV University."
* **Joshua Bardwell Variable PWM Testing & Benchmarks:** [YouTube Video](https://youtu.be/xuQeJA4EGr8)
* **Chris Rosser BLHeli_32 Performance & Tuning (2021):** [YouTube Video](https://youtu.be/7WeHTb7aBrE)
* **Chris Rosser BLHeli_32 & State of ESCs (2024):** [YouTube Video](https://youtu.be/6gv0_jTEYZM)

---

*Related Knowledge Base Modules:*
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [Theory of Operation & Hardware](theory-operation-hardware.md)
* [PWM Switching & Thermal Dynamics](pwm-frequency-heat.md)
* [ESC Flashing & Unbricking Guide](flashing-unbricking-hardware.md)
* [Back to Knowledge Base Index](README.md)
