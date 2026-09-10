# Tune weird quads - Ramp Up power — KababFPV
https://youtu.be/obObn1oGWmU

NOT a tiny-whoop video. The quad shown flying is a 300g 5.25" 2204/2800Kv on 3S — explicitly framed by the creator as an oddball/nonstandard build outside Betaflight's expected performance envelope (which he says "wants" ~500-600g with 2207/2306 motors and 5" props). This is a talking-head/opinion video, not a bench test — no instrumented data, no numeric before/after measurements.

## Terminology (creator's own understanding, explicitly caveated)
- [04:14-04:35] "Ramp up power" = "startup power" in BLHeli_S naming, "ramp up power" in BLHeli32 naming — creator says "I believe the two do exactly the same thing" (stated as belief, not verified).
- [04:35-05:10] Recent Bluejay firmware splits it into 3 settings: **startup power** (used to un-stick/start the motor from rest, e.g. grass/dust), **ramp up min** (min power for low→high RPM transition), **ramp up max** (max power for low→max RPM transition).
- [05:44-05:52] Explicit disclaimer: "this is not correct in any technical sense, this is a very layman's understanding of how it may or may not work."

## Claimed effects (all anecdotal, all hedged by the creator as build-specific)
- [06:08-06:34] Reducing ramp-up power "loosens" PID-loop response; increasing it "tightens" response.
- [06:34-06:52] Too high → PID overshoot/oscillation, described as a "flutter" and an audible "robotic" motor sound during rolls/flips — he says this is often mistaken for a noise/vibration problem to be fixed via filtering.
- [07:19-07:25] Dropping PIDs near zero can mask this symptom (quad flies badly but the flutter goes away) — offered as a diagnostic clue, not a fix.
- [07:33-07:48] Too low → quad feels "loose," poor propwash handling, PIDs can't be tuned to compensate.
- [08:00-08:23] Explicit recommendation: **set ramp-up power (and demag, and motor timing) BEFORE touching filters or adding/upsizing a capacitor** — claims reducing ramp-up power can reduce back-EMF noise feedback enough to lessen the need for a bigger cap (anecdotal, not measured).
- [10:00-10:31] For 3" quads on 6S showing flutter/noise/no-power symptoms: reducing "the power up timing" (his phrasing — ambiguous whether he means ramp-up power or motor timing) fixed it for the setups he tried.
- [11:23-11:38] For 7"+ quads with weak performance: **increasing** ramp-up power drastically improved response and throttle-pump behavior, on Betaflight and EmuFlight.
- [12:31-13:12] Reports ESC-brand variance in real-world ramp-up power output at the *same* nominal setting (0.5 default): "primarily HGLRC" ESCs (six or seven models) reportedly deliver less power than most other brands at that setting; some T-Motor ESCs reportedly deliver more. Explicitly sourced as "talking to like 100 people about this issue" — crowdsourced anecdote, not the creator's own measurement.
- [13:19-13:57] Kv/voltage mismatch: for a low-Kv motor run on lower-than-rated voltage (e.g., a 6S-Kv motor flown on 4S), increasing ramp-up power can help it "fit" the PID loop. For the reverse (e.g., 2500Kv motor flown on 6S), reducing ramp-up power can help.
- [14:14-14:38] Notes KISS ESCs (at time of recording) don't have a ramp-up-power setting at all.

## No numeric settings given
This video contains **no specific numeric startup/ramp-up power values** (no "1100," "1200-1250," or similar) and **no whoop-specific (0702-1202) guidance at all** — it's general-audience, qualitative, and centered on oddball/off-spec builds from 3" to 7"+.
