# Chris Rosser — "Protect Yourself from ESC Voltage Spikes: Testing Capacitors and TVS Diodes"
https://youtu.be/VgHOHcWu7-U

Test rig: 2207 1850KV 6S motor, 5x4.3x3 prop, Sky Stars KM55A AM32 ESC, oscilloscope probes on battery leads + one motor phase, 4S 5000mAh + 6S 5000mAh packs. Prompted by Luke Maximo Bell reporting ESCs spontaneously catching fire on disarm on his world-record drone.

## Baseline spikes, no capacitor/TVS fitted (still has whatever cap ships stock on the ESC board itself — this is testing *additional* filtering)
- 4S, running: spikes to ~19-21V battery-lead depending on RPM (16.1-16.2V average pack voltage), ~22-26V on motor phase.
- 4S, active braking (disarm): bigger spikes than running. 10k RPM → 25V battery/25V phase. 15k RPM → 27-28V battery/26V phase. 20k RPM → similar. 25k RPM (highest tested) → 30V battery-lead / 35V motor-phase — "more than twice average battery voltage" on the phase.
- 6S, 30k RPM running: 32V battery / 34V phase (23.1V avg). 6S braking: 44.5V battery-lead spike / 28V phase (23.8V avg) — voltage *dips* first at start of braking on 6S (opposite of 4S, where it rises first) — creator notes he doesn't know why, speculates commutation-related.

## With 470µF capacitor, short legs, soldered directly to battery pads
- 20k RPM running: battery spike ~16.5V (15.8V avg) — "barely any spikes." Motor phase still spikes to ~19.5V — capacitor doesn't fully tame phase-level spikes.
- 20k RPM braking: battery ~18V, phase ~22.5V.
- 25k RPM braking (worst case, catches first braking phase): phase spike 24V, battery only ~17.5V.
- Conclusion: capacitor handles battery-lead spikes very well; motor-phase spikes remain elevated above battery voltage even with cap fitted.

## Lead length test (same 470µF cap, legs left at 20mm instead of shortest-possible)
- Degradation is real but small: ~0.5-1V bigger spike at 20mm vs. shortest-possible, across both running and braking tests at 20k/25k RPM.
- Recommendation: keep legs short, but don't over-worry about a build requiring slightly longer legs. If extending, use thicker wire (e.g. spare motor wire) for lower added resistance.

## 470µF vs 1000µF
- No meaningful improvement found moving from 470µF to 1000µF on this single-motor rig (possibly diminishing returns; might matter more on a 4-in-1 ESC with 4 motors braking simultaneously — not tested here).

## TVS diode only (no capacitor), 6S tests
- "FET spike absorber" = 3x TVS/zener diodes. Explains reverse-bias operation and breakdown-voltage clamping mechanism in the video.
- 10k RPM: 27V battery / 29V phase running; 30.5V battery braking (visibly different waveform shape — diode clamping visible as a "step").
- 15k-25k RPM: battery-lead spikes clamp around 30-32V regardless of RPM increase (diode doing its job), but motor-phase spikes still reach up to 36-38.5V.
- Conclusion: TVS diode alone is much less effective than a capacitor at suppressing battery-lead spikes (~30-32V vs. cap's ~battery+1-2V), though it does cap the peak rather than let it scale with RPM.

## Recommendations (direct)
- Never run an ESC without its capacitor soldered on — spikes reach at least double battery voltage without one.
- Capacitor > TVS diode for primary protection. TVS diode is a good backup/fallback (protects if capacitor fails/desolders mid-flight) and is standard-equipment on some modern ESCs (T-Motor F55 Pro 3 named as an example with onboard TVS).
- For extreme builds (speed records): want both max capacitance AND a TVS diode, because motor-phase spikes can exceed what a battery-lead capacitor alone addresses.
- Never solder a capacitor across motor phases (disrupts commutation) — a TVS diode with breakdown voltage just under the FET rating is the correct component there if needed.
