# Chris Rosser — "Stop Buying these ESCs: 30+ ESCs Torture-Tested"
https://youtu.be/pH4K3ErugW4

Test rig: 40 popular 30x30 and 20x20 ESCs, each wired to the same AOS Supernova 3220 motor and power supply. Three tests: step response (10%-50%-10% throttle), flywheel torque test (200kg·cm flywheel, same throttle step), thermal torture test (10%-throttle-step ramp, thermal camera on MOSFETs, custom Python/OCR script built to extract per-frame temperature since the thermal-camera software's own export tool doesn't support video).

## Step response (accel/decel to/from 90%/10% of RPM change, in RPM/s)
- Top 10 dominated by breaking (deceleration) performance more than acceleration — best units decelerate as fast or faster than they accelerate.
- Bottom of the list: FETtec SFOC 65A refused to complete the step command at all — disarmed and beeped warning tones instead of responding. (Same behavior repeated in the torque test below — this is the *prototype* unit, consistent with QENbsI3swCI/M4NOIwdSBOc's framing of it as early-stage hardware/firmware.)

## Flywheel torque test (torque at 4000 RPM used as the comparison point)
- Spread: best unit delivered more than 2x the torque of the worst at 4000 RPM. Higher current rating generally correlated with more torque, but not reliably — some 55A units beat 70-90A-rated units by a wide margin.
- **Same-hardware, firmware-only comparison: T-Motor F55A Pro III tested under AM32 firmware placed 4th in this test; the identical board under BLHeli_32 firmware placed 3rd-from-bottom.** Explicitly confirmed as a hardware-identical, firmware-only difference by the creator ("we know it's not the hardware because these boards are identical. The only difference is the firmware").
- Notes some settings can be adjusted in AM32 Configurator/BLHeli_32 Suite, others are hardcoded by the manufacturer and not user-adjustable.
- Bottom of this test dominated by smaller boards/smaller MOSFETs — attributed to higher FET R_on limiting max current and therefore torque.
- FETtec SFOC prototype again disarmed/refused to complete the test.

## Thermal torture test (time to reach +100°C above ambient, seconds)
- Best unit outlasted the worst by more than 72%.
- Top 10 spans well-known and lesser-known brands; 5 of the top 10 have no aluminum heat spreader at all — relying on low-resistance FETs and a thick PCB instead. Heat spreader presence alone is not the deciding factor.
- Bottom 10 dominated by smaller/lighter PCBs with smaller FETs; **current rating again not a reliable predictor** — some 60-70A-rated boards were outperformed by lower-rated boards.

## Overall scoring methodology
- Per-category score = unit performance ÷ batch average (100% = average). Categories: thermal torture, efficiency at 50% throttle, torque at 4000 RPM, response. Final rank = average of the 4 category scores.
- Notes a substantial dozen more ESCs already queued for a future testing round, plus a planned AIO-board test video.

## Retained for this KB (firmware/hardware-design findings only, no brand rankings per AGENTS.md)
- Same PCB, different firmware (AM32 vs. BLHeli_32) → materially different torque output — real, hardware-controlled evidence firmware affects torque delivery.
- Current (amp) rating is not a reliable standalone predictor of either torque output or thermal resilience; FET R_on / PCB copper mass / heat-spreader presence matter more.
- FETtec SFOC prototype could not complete standard step-response or torque tests in this batch as of this video's testing (2026-03) — contrasts with its strong showing in the smaller, more controlled QENbsI3swCI test batch (2024-03); may reflect firmware maturity differences between the two test dates, not re-verified here.
