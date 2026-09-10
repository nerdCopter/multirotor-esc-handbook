# Chris Rosser — "Don't buy another ESC Until You See These 150°C Torture Test Results"
https://youtu.be/mdM4jEWOqx4

Test rig: Tytto Robotics Flight Stand 15 Pro (1000Hz sampling, ASM calibrated load cell, up to 150A), Thermal Master P3 thermal camera, AOS Supernova 3220 motor (draws 80A+ at full throttle — above every tested ESC's continuous rating, intentionally). Factory firmware on all units; only KV setting adjusted in AM32 to match motor.

## Units tested
- T-Hobby (formerly T-Motor): Velox 50A, Velox 70A, F55A Pro 3, Pacer P60A V2 (the only BLHeli_32 unit in this batch — all others AM32).
- ZDX Future: Boret 80A, Boret 80A Mini.
- Newbie Drone: Hummingbird 305, Hummingbird 200 Race Spec.

## Acceleration/deceleration (10%→50% throttle, avg of 20 reps)
- All units except Pacer 60A V2 start ~2000 RPM, accelerate to ~9000 RPM in 140-150ms. Pacer (BLHeli_32) starts at a different RPM — attributed to its different firmware.
- Fastest accelerating: F55A Pro 3 and Pacer 60A V2. Slowest: Hummingbird 200 Race Spec.
- Deceleration: fastest = Hummingbird 305 and Pacer 60A V2; slowest = Hummingbird 200 Race Spec. Otherwise not a large spread.

## Flywheel torque test (10%→50% throttle, flywheel to 10,000 RPM)
- **Pacer 60A V2 (BLHeli_32) had clear peak-torque advantage: ~1.5 N·m at 3500 RPM.** Its torque curve shape is qualitatively different from the AM32 units — AM32 units peak earlier (~2500-2700 RPM) but at lower peak magnitude.
- Among AM32 units: F55A Pro 3 best, Hummingbird 200 Race Spec worst.
- Within same firmware, torque differences attributed mainly to current-delivery capability (lower resistance → more current → more torque), not firmware logic.

## Efficiency (g thrust / W electrical, across throttle ramp)
- Big spread at low throttle: ~4.5-7.5 g/W depending on unit (same motor/prop across all).
- Efficiency converges across units as throttle rises toward 100% (theoretical floor = ESC resistance when FETs are just "on").
- **Pacer 60A V2 (BLHeli_32): lowest efficiency at low throttle, but improves relatively at high throttle** — interpreted as "BLHeli_32 isn't a super efficient firmware at low throttle, but this ESC's low resistance construction helps it at high throttle."
- AM32 units cluster tightly at low throttle (same firmware/settings); F55A Pro 3 slightly ahead at high throttle (lower resistance).
- Hummingbird 200 Race Spec: most efficient at low throttle, but became relatively less efficient at higher throttle and was among the first to overheat (only reached ~70% throttle before hitting 150°C).

## Thermal torture test (10%-throttle steps every 10s, stop at 150°C FET temp)
- None of the 8 units reached 100% throttle for a full 10s step before hitting 150°C.
- Overheat order (first to last): Velox 50A and Hummingbird 200 Race Spec (first) → Hummingbird 305 → Boret 80A Mini → F55A Pro 3 → Boret 80A, Velox 70A, Pacer 60A V2 (best, reaching 100% throttle for a few seconds before cutoff).
- **Key finding: current (amp) rating does NOT reliably predict thermal resilience** — some lower-A-rated units outperformed 80A-rated units.
- **Better predictor: PCB thickness/copper mass** (more copper = lower resistance + more heat-spreading mass) **and presence of an aluminum heat spreader** across the MOSFETs (even measured from the opposite side of the board from the spreader, it still measurably helped).

## Overall scoring methodology
- Per-metric score = unit's performance ÷ batch average (100% = average, >100% = better than average). Total score = average of the 4 category scores (accel/decel treated together as "response," torque, efficiency, thermal).
- Best overall: Pacer 60A V2 (weak on efficiency, strong everywhere else). Second: F55A Pro 3 (well-rounded). Notably, this is the SAME PCB hardware as the F55A Pro 3 used with AM32 firmware in pH4K3ErugW4/QENbsI3swCI — direct point of comparison for firmware-only effect, though not literally the same physical unit across videos.

## Not covered
- No mention of Demag Compensation, PWM frequency, or motor timing settings — this video is purely comparative bench testing at factory-default firmware settings.
