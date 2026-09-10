# Chris Rosser — "Testing to find the ULTIMATE BLHeli_32 Settings for 3" and 5" drones"
https://youtu.be/6gv0_jTEYZM

Test rig: BLHeliSuite32, bidirectional DShot on thrust test stand, plus flywheel tests (100 kg·mm² flywheel for 5", 10 kg·mm² for 3").
Motors: 5" = AOS Supernova 2207 (14 magnets/12 coils) / HQProp 5x4.5x3 V1S. 3" = Sky Stars 1404 (12 magnets/9 coils) / HQProp 3x3x3.
States plainly: "P modulation mode" and other panel settings "didn't have any measurable impact on Motor Performance on either the 5 or the 3-in motor" — not covered further.

## Rampup Power (BLHeli_32, "3% to 150%")
- 5" Supernova: no acceleration difference above ~10% rampup; motor gets progressively warmer above 10% with no further accel benefit. **Recommendation: 30% as a conservative setting for a high-powered 5" motor.**
- 3" Sky Stars: bigger influence than on 5" — accel improves up to ~20-30%, no further benefit above that. At 3% rampup, motor failed to start (both sizes). **Recommendation: 50% as a conservative setting for a 3" motor.**
- **Rampup power has no measurable effect on deceleration, on either motor** (confirmed by both prop and flywheel tests).
- Flywheel tests (slower, larger-inertia acceleration) show the same pattern as prop tests for both sizes.

## Motor Timing (BLHeli_32, "1° to 31°" + Auto)
- Increasing timing significantly increases *measured* KV: 5" motor +100KV+ going 8°→31°; 3" motor +~200KV over the same range. Auto timing measures about the same KV as 16° fixed.
- 5" Supernova: timing has only a small effect on responsiveness/acceleration; 31° timing measurably increased motor heating during testing. Deceleration: negligible effect from timing (31° very slightly slower).
- 3" Sky Stars (fewer poles/coils): much bigger effect — 8° struggles badly, big improvement 8°→16°→24°, **no further benefit above 24°** (31° gives no more, and top-end thrust actually falls slightly at 31°). Speaker speculates the difference in pole/coil count vs. the 5" motor explains why optimum timing differs by motor.
- Efficiency: 5" motor — timing has little effect; 16° slight edge at low power/cruise, 24-31° slight edge at high power/top end, differences marginal. 3" motor — Auto falls slightly behind fixed settings; 8° clearly worst; little difference among 16°/24°/31°.
- Top-end thrust: 5" motor — 24°/31° give slightly more than lower settings. 3" motor — 16°/24° give the max, 31° actually falls off slightly.
- **Recommendations:** 5" Supernova — **16° if prioritizing cruise efficiency**, or **24° if prioritizing top-end power/efficiency at high throttle** (minimal heating penalty). 3" Sky Stars (12-pole/9-coil) — **24° recommended outright**, clear benefits in responsiveness, efficiency, and top-end thrust vs. lower settings, no benefit going higher.

## PWM Frequency (BLHeli_32, "16kHz to 128kHz" + By RPM; baseline test config was 16° timing / 24kHz, "the default for BL heli 32")
- 5" Supernova: acceleration nearly identical across all frequencies **except 128kHz**, which "does seem to significantly reduce the acceleration of the motor." Marginal edges elsewhere (32/48kHz at low RPM, maybe 96kHz at higher RPM) — differences called out as very marginal.
- 3" Sky Stars: **much bigger effect** — smaller motor/lower inductance. 16kHz and 24kHz fastest-accelerating; 32kHz and 48kHz progressively slower; **above 48kHz (96kHz, 128kHz) the motor would not accelerate cleanly at all** — audibly "jerking," sometimes failing to spin up, no usable data collected at those settings.
- Deceleration/braking: **monotonic** on both motors — higher PWM frequency = weaker/slower braking, audible at high frequencies. "You can't really recommend running a fixed PWM much higher than about 48kHz because of that huge impact on motor braking."
- Efficiency: small improvement 16kHz→24kHz on both motors, then "very very little benefit" increasing further on either motor — directly contradicts the common claim that higher PWM meaningfully improves efficiency.
- Top-end thrust/power: no measurable effect of PWM frequency at full throttle on either motor (FETs are simply fully on, not switching). Throttle-curve shape shifts slightly with frequency but "pretty marginal," "probably wouldn't even notice it."
- **Recommendation: 24kHz as the sweet spot for BOTH 5" and 3" motors** — "best balance of motor responsiveness and also efficiency." 16kHz gives a little more responsiveness at some efficiency cost.
- **Variable PWM (by-throttle, e.g. low=24kHz/high=48kHz, linear with throttle position): "doesn't offer a lot of benefit" per his testing** — "there isn't much benefit of higher pwm frequencies at higher throttle positions."
- **PWM By-RPM**: does have a real use case — fixes "notchy"/jumpy throttle caused by PWM frequency coinciding with commutation frequency. Frequency only rises modestly with By-RPM (~24kHz up to ~40kHz max), staying within the good low-frequency range for braking/efficiency. Speaker's own quads run **24kHz fixed** — "I always run 24k fixed pwm" — and only suggests switching to By-RPM if you specifically notice throttle notchiness.

## Not covered / no data
- **No mention of Demag Compensation anywhere in this video** — explicitly states other panel settings weren't covered because they showed no measurable performance effect; Demag isn't named as one tested or excluded, it's simply absent from the video entirely. Neither confirms nor contradicts the KB's existing Demag guidance.
