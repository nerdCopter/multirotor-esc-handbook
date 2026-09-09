# Advanced ESC Features, 3D Flight, Turtle Mode & Sound Customization

Modern ESC firmwares incorporate specialized features beyond standard flight commutation, including bidirectional crash recovery, bi-directional 3D aerobatics, active telemetry streaming, and programmable startup melodies.

---

## 1. Flip Over After Crash (Turtle Mode / Crash Flip)

**Turtle Mode** allows an inverted multirotor that has crashed upside down to flip itself over without manual retrieval.

```
       [ Drone Inverted Upside Down ]
                    │
   [ Pilot Engages Turtle Mode Switch on Radio ]
                    │
   [ FC mixer reverses selected motors via DSHOT_CMD_SPIN_DIRECTION,
     then drives them at a fixed crashflip_motor_percent — throttle
     stick is ignored, pitch/roll stick picks the flip direction ]
                    │
   [ Selected Motor Pair Spins Reversed ] ──► Frame Pivots & Flips Upright
```

There is no single dedicated "turtle mode" DShot command. Betaflight's crash-flip logic
lives in the mixer: it reverses the spin direction of the motors needed for the chosen flip
direction (via the standard `DSHOT_CMD_SPIN_DIRECTION_*` commands — requires bidirectional
DShot) and drives them at `crashflip_motor_percent` (default `25`), independent of the
throttle channel. See [Protocols & Telemetry](protocols-telemetry.md) for the real command set.

### Technical & Electrical Considerations:
* **Current Spikes:** Spinning props directly into grass or dirt creates near locked-rotor conditions. Peak current can exceed 40A–60A per motor instantly.
* **Firmware Safety:** Modern firmwares (AM32, Bluejay, BLHeli_32) limit the maximum duty cycle and current during Turtle Mode to prevent burning MOSFETs.
* **Tuning Tip:** If the quad struggles to flip on thick grass, raise `crashflip_motor_percent` in Betaflight (PID Tuning tab / CLI).

---

## 2. 3D Aerobatic Flight (Reversible Motor Direction)

In 3D flight mode, the ESC can reverse motor rotation direction in real-time mid-flight based on stick input:
* **Stick Center (1500µs / DShot value 1048):** Motor stopped (0 RPM).
* **Stick Up (1500–2000µs):** Motor rotates forward for normal positive thrust.
* **Stick Down (1000–1500µs):** Motor actively brakes to zero and accelerates in reverse for inverted negative thrust.

### Electrical Dynamics:
* Reversing rotation at high RPM injects huge regenerative energy back into the power bus.
* **Requirements:** High-capacity Low-ESR capacitors (1000µF+) and TVS diodes are strictly mandatory to absorb reverse commutation voltage flyback.

---

## 3. Custom Startup Melodies & Tone Customization

ESCs can oscillate motor stator windings at audible frequencies (500Hz–5kHz) using low-power PWM pulses, turning the brushless motors into acoustic speakers.

### Firmware Tools:
* **Bluejay & AM32:** Direct integration with online RTTTL (Ring Tone Text Transfer Language) converters via [esc-configurator.com](https://esc-configurator.com/) and [am32.ca](https://am32.ca/).
* **BLHeli_32:** Integrated melody editor in BLHeliSuite32.

---

## 4. Current Limiting, Thermal Protection & Sensor Telemetry

1. **Hardware Overcurrent Protection (OCP):**
   * Uses fast internal analog comparators connected to the shunt resistor. If current exceeds a hard limit (e.g. 80A), gate drivers are cut within nanoseconds.
2. **Thermal Throttling:**
   * Reads onboard NTC thermistors or internal MCU temperature sensors. If temperature exceeds 110°C–120°C, the firmware progressively reduces maximum duty cycle to prevent MOSFET delamination.

---

*Related Documentation:*
* [Theory of Operation & Hardware](theory-operation-hardware.md)
* [Protocols & Telemetry](protocols-telemetry.md)
* [Desyncs & Commutation Troubleshooting](desyncs.md)
* [Back to Knowledge Base Index](README.md)
