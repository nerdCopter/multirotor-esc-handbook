# ESC Diagnostics & Blackbox Log Analysis Guide

Blackbox flight logging provides flight controller and motor telemetry data recorded at rates up to 4kHz or 8kHz. Analyzing Blackbox traces allows precise identification of ESC desyncs, motor saturation, electrical noise, and commutation loss.

---

## 1. Setting Up Blackbox for ESC Diagnostics

To accurately diagnose ESC behavior and desyncs, configure Betaflight Blackbox with the following parameters:

```bash
# In Betaflight CLI
set blackbox_sample_rate = 1/1    # full-rate logging (halve to 1/2, 1/4... on limited storage)
set blackbox_mode = ARMED         # start logging automatically on arm (or ALWAYS / SWITCH)
set blackbox_device = SDCARD      # or FLASH
set debug_mode = DSHOT_RPM_ERRORS # per-motor invalid-telemetry-packet percentage (DEBUG_DSHOT_RPM_ERRORS)
save
```

### Critical Trace Channels to Enable:
1. **`axisRate[0,1,2]` (Gyro Rates):** Roll, Pitch, Yaw angular velocities (°/s).
2. **`motor[0,1,2,3]` (Commanded Motor Output):** FC digital throttle command sent to ESCs (0–100% or 0–2000).
3. **`dshot_rpm[0,1,2,3]` (Real-time e-RPM Telemetry):** Measured physical motor speed reported via Bidirectional DShot.
4. **`dshot_err[0,1,2,3]` (DShot Error Rate):** Frame checksum failure count.

---

## 2. Signature of a Motor Desync in Blackbox

When an ESC desyncs in flight, the trace exhibits a distinctive, unmistakable signature:

```
Gyro Yaw/Roll (deg/s)
         ▲
         │           /═════► Severe Uncommanded Spin ("Death Roll")
    0 ───┼──────────/
         │
Motor Command (PID Out)
  100% ──┼───────┌─────────► Motor #2 Commanded to 100% (FC attempting to counteract roll)
    0% ──┼───────┘
Measured DShot RPM
  Max ───┼───────┐
         │       └─────────► Motor #2 RPM drops to ZERO or stutters wildly
    0 ───┼─────────────────►
         └────────────────────────► Time (ms)
```

### Step-by-Step Anatomy:
1. **Step 1 (The Trigger):** High throttle punch, rapid stick reversal, or hard turn.
2. **Step 2 (The Slip):** One motor's **Measured RPM (`dshot_rpm`)** suddenly collapses toward zero or flatlines with erratic jitter.
3. **Step 3 (PID Windup):** The Flight Controller PID loop detects that the quad is losing attitude and commands that motor (`motor[n]`) to **100% maximum throttle**.
4. **Step 4 (Death Roll):** Because the desynced motor cannot deliver torque, the remaining three motors spin up asymmetrically, sending the quadcopter into an uncontrolled spiral.

### Real Example
A real Betaflight blackbox render matching this exact signature (community-shared, not from any of the videos or wikis): at the trigger point, Gyro traces spike sharply on all three axes; the **Motors** pane shows one motor's commanded trace climb and hold near-flat at a high plateau while the others settle back down after the transient; the **RPM** pane shows that same motor's measured RPM collapse and flatline low while the others recover to a steady cruise RPM — the "commanded high, measured RPM low" divergence described in Step 2/3 above. The pilot in this example disarmed via switch shortly after the event rather than riding out a full death roll.

---

## 3. Blackbox Diagnosis: Mechanical vs Electrical vs Tuning Issues

| Blackbox Symptom | Root Cause | Solution |
| :--- | :--- | :--- |
| Single motor trace at 100% while its DShot RPM collapses to 0 during punchout. | **Motor Desync** (Commutation slip). | **BLHeli_32/Bluejay:** step Demag up one level at a time (BLHeli_32: Off→Low→Medium→High; Bluejay: Off→Low→High — don't skip levels or jump straight to High). **AM32 has no Demag setting** — check Motor KV/Motor poles instead. All firmwares: set static Motor Timing to `22°–23°`, check Rampup Power against the sourced per-size figures in [desyncs.md §3](desyncs.md#3-master-settings-matrix-across-all-esc-firmwares). |
| All 4 motors show high-frequency sinusoidal ripple (150Hz–300Hz) correlated with gyro noise. | **D-Term Resonance / Noisy Gyro**. | Lower D-gain, verify dynamic RPM notch filters are tracking motor harmonics. |
| `dshot_err` climbs above 0.1%–1.0% in flight. | **Electrical Signal Noise / Inductive Ground Bounce**. | Add Low-ESR capacitor across battery pads, ensure signal ground wire is twisted with signal wire. |
| Motor RPM drops to zero during inverted zero-throttle freefall. | **Idle RPM Dropped Below Commutation Threshold**. | Raise `dyn_idle_min_rpm` in Betaflight Dynamic Idle. |

---

## 4. Not Actually a Desync: Differential Diagnosis

> **Unverified community report** (Discord, not independently confirmed — flagged here as a real diagnostic possibility to rule out, not a validated mechanism): a GPS module that disconnects or initializes improperly can reportedly cause Betaflight to freeze the entire flight controller mid-flight. The crash this produces can look identical to a desync death-spiral from the outside, but the **key differentiator is total FC lockup** — the beeper does not sound and Turtle Mode does not respond, because the FC itself is frozen, not just one ESC. A genuine desync leaves the FC alive: beeper and turtle mode still work after the crash. If a "desync" doesn't respond to a beeper or turtle command, check GPS wiring/connection before re-tuning ESC settings.

---

*Related Documentation:*
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [Betaflight ESC Integration](betaflight-esc-tuning.md)
* [Protocols & Telemetry](protocols-telemetry.md)
* [Back to Knowledge Base Index](README.md)
