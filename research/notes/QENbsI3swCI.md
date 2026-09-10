# Chris Rosser — "Testing FETTECs new PROTOTYPE ESC against BLHeli_32 AM_32 and BlueJay"
https://youtu.be/QENbsI3swCI

Test rig (new methodology, upgrade from prior thrust-stand-only setup): HQ 5x4.5x3 V1S prop, AOS Supernova 2207 motor, F7 flight controller logging bidirectional DShot at 8kHz (vs. ~180Hz on a thrust stand alone), 24V supply topping up a 6S 5200mAh pack, 100kg·mm² flywheel for torque test.

## Units tested
- SpeedyBee 55A 8-bit ESC, Bluejay firmware v0.19.2
- Diatone Mamba F60 Pro, AM32 v1.99
- T-Motor F55A Pro 3, BLHeli_32 v32.10
- Sky Stars KM55A (20x20), AM32 v1.99
- FETtec prototype 65A, SFOC firmware v1.01

## Throttle ramp (0-100%, 5s up/5s down)
- All units track near-identically to ~40% throttle, diverge slightly above that.
- Full-throttle RPM spread ~3% across all 5 units. Highest: Bluejay/SpeedyBee. Lowest: Mamba F60 Pro/AM32. Creator notes this could be firmware or hardware, not conclusively separated.
- Bluejay/SpeedyBee showed a stepped/coarser response at the very top of throttle range — plausibly the only 8-bit ESC in the batch (less throttle resolution than the 32-bit units), not confirmed as a Bluejay-firmware-specific issue.
- Ramp curves symmetric (up vs. down) for all units.

## Responsiveness (10%→50% throttle step, avg of 10 reps, time to 90% of RPM change)
- T-Motor F55A / BLHeli_32: slowest, ~52ms.
- All AM32 and Bluejay units: ~44ms (tied/near-identical).
- FETtec SFOC prototype: fastest, ~40ms.
- Creator explicitly frames the 12ms gap (BLHeli_32 vs. FETtec) as "significant" — comparable to RC-link/filter latency, and claims anecdotal evidence that pilots can perceive a ~10ms difference in video-link delay, extrapolating that a similar gap in ESC response would also be perceptible.
- Deceleration: no meaningful difference across any of the 5 units.

## Flywheel torque test (6-50% throttle step with 100kg·mm² flywheel attached)
- BLHeli_32/T-Motor F55A: slowest to accelerate flywheel = least torque generated, of the 5 units.
- Sky Stars KM55A (20x20 AM32): smaller ESC, still accelerates flywheel well.
- SpeedyBee Bluejay and Mamba F60 Pro AM32 (both 30x30): generate more torque, accelerate flywheel quickly.
- FETtec SFOC prototype: standout — significantly more torque/faster acceleration than all 4 other units.

## Caveats stated by creator
- FETtec unit is explicitly prototype hardware AND prototype firmware (v1.01) — not a released product as of this video (2024-03-08).
- Slight extra RPM-signal noise at low RPM noted on the FETtec unit specifically, attributed to early firmware state, expected to improve before release.
- Creator planned a dedicated deep-dive video on the FETtec SFOC technical approach — see M4NOIwdSBOc.
