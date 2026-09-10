# Chris Rosser — "Motor Commutation Explained: Featuring FETTECs new SFOC approach"
https://youtu.be/M4NOIwdSBOc

Technical explainer, not a bench test. Uses back-EMF/phase-current measurement graphs credited to "Felix at FETtec," recorded on a T-Motor F40 Pro.

## Motor background
- BLDC motor phases (A/B/C) wired to 3 or 4 coils each depending on 9-pole (3 coils/phase) vs. 12-pole (4 coils/phase) motor.
- Back-EMF current shape on a T-motor F40 Pro measured "very close to sinusoidal" — implies the ideal drive current is also near-sinusoidal for max torque/efficiency.

## Block commutation
- Drive phase fully on when ideal driving current > 50%, off when < 50% (simple thresholding/rounding).
- Zero-crossing of back-EMF (when phase isn't driven) used to locate rotor position.
- Problems: (1) fast on/off switching → demagnetization current spikes (visible on ESC input current); (2) ~33% of cycle time not driving the phase (listening for zero-crossing instead) — lost torque/power opportunity; (3) drive current shape ≠ back-EMF shape → under/over-driving at different points in the cycle → less efficient than ideal.

## Pure sinusoidal commutation
- PWM synthesizes a sinusoidal voltage on each phase, matching back-EMF shape — most efficient/smooth.
- Problem 1: constant switching (vs. block's mostly-static on/off) → higher switching losses/heat, same reason CPU overclocking generates more heat.
- Problem 2: max phase-to-phase voltage only reaches ~87% of battery voltage (two sinusoids never maximally separated at the same instant) — less torque/power ceiling than block commutation's full battery voltage at 100% throttle.

## Space Vector Modulation (SVM)
- Take the instantaneous minimum of all 3 phase voltages, subtract it from all 3 phases → produces the characteristic "double hump" SVM waveform.
- Because the same voltage is subtracted from all phases equally, there's zero voltage difference between phases from the subtracted term → zero current contribution from it → the actual phase *current* stays a clean sinusoid despite the drive *voltage* no longer looking sinusoidal.
- Each phase spends real time at 0V (reduces switching losses vs. pure sinusoidal) AND reaches 100% battery voltage at full throttle (recovers block's max torque/power) — solves both sinusoidal-commutation problems simultaneously.
- Real graph shown (Felix/FETtec, T-motor F40 Pro, full throttle): SVM drive voltage visibly PWM-synthesized double-hump, measured phase current very close to a perfect sinusoid.
- Remaining limitation: SVM still needs a brief zero-drive window per phase to sense the back-EMF zero-crossing (can't listen for BEMF while actively driving all 3 phases at once) — this window slightly distorts the otherwise-clean current waveform ("slightly jagged" vs. block commutation's much rougher waveform).

## FETtec SFOC (Simple/Sensorless FOC) — new approach
- Removes the zero-drive sensing window entirely.
- Requires measuring both current AND voltage on each phase continuously — needs a shunt resistor per phase (not just the single ESC-wide battery-lead shunt most FPV ESCs use for whole-ESC current sensing/telemetry).
- FETtec's SFOC prototype board shown with what appear to be 2 shunt resistors per motor (4 motors × 2 shunts visible near MCU/gate drivers).
- With continuous current+voltage per phase, back-EMF (and therefore rotor position) can be calculated continuously rather than only at the 3 zero-crossing points per electrical cycle that block/sinusoidal/SVM rely on.
- Claimed benefits: smoother motor operation (finer timing resolution), perfect sinusoidal current with no jagged zero-drive-window artifact, phase driven 100% of the time (more torque/power available).
- Explicitly hardware-dependent: most existing FPV ESC hardware only has one shunt (battery-lead, whole-ESC current) — not appropriate for this per-phase BEMF calculation. Not a drop-in firmware update for existing boards.
- Video ends noting a follow-up video/bench test was planned — see QENbsI3swCI for the actual bench comparison.
