> q&A: what bluejay timing/demag for 5 inch, 2207, 6S ?

  For a 5-inch, 2207 stator, 6S build running Bluejay /bluejay.md, here are the recommended settings depending on your flight style:
  ──────
  ### 1. Recommended Settings (Freestyle / General Flight)

  • Motor Timing: Medium-High (or 22.5° / High depending on configurator display)
      • Why: Avoid Auto. A fixed Medium-High / High timing setting provides the exact phase lead needed for 2207 stators on 6S, eliminating tracking lag and desyncs during snap rolls, inverted yaw
      spins, and 0%–100% full-throttle punchouts.
  • Demag Compensation: High
      • Why: 2207 motors have larger magnets and higher inductive flyback energy than smaller micro motors. Setting Demag to High prevents magnetic field collapse from blinding the Back-EMF zero-
      crossing comparator during rapid direction reversals and aggressive prop loading.
  • PWM Frequency: 24kHz or 48kHz (Do not use 96kHz on 5" 6S)
      • 24kHz: Maximum active braking torque (damped light), best low-end grip, and lowest desync risk.
      • 48kHz: Smoother throttle and slightly longer flight times with minimal motor whine.
  • Startup Power:
      • Startup Min Power: 1050 – 1100
      • Startup Max Power: 1150 – 1200 (Never max these out)

  ──────
  ### 2. Dedicated Racing Setup (Maximum Top-End RPM)

  If you are running high-Kv 6S racing motors (e.g. 2100Kv–2150Kv) on a clean, low-vibration build with a fresh Low-ESR capacitor:

  • Motor Timing: High
  • Demag Compensation: Low (or Off)
      • Why: Setting Demag to Low/Off eliminates artificial phase delays, recovering ~5,000 to 7,000 top-end RPM (up to 15–20% power gain) on high-Kv 6S packs.
      • Condition: Only use Low Demag if your motor screws are properly sized, frame resonance is low, and you have a 35V–50V 1000µF Low-ESR capacitor soldered directly to the ESC pads.

  ──────
  ### 3. Corresponding Betaflight Settings

  • Protocol: DShot300 with 4.0 kHz PID loop (Optimal for 8-bit Bluejay MCUs to prevent packet jitter/errors).
  • Dynamic Idle: set idle_min_rpm = 32 (~3,200 RPM) to keep the motor above the zero-crossing noise floor during zero-throttle maneuvers.
