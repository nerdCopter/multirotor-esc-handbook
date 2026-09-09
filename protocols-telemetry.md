# ESC Communication Protocols, DShot & Telemetry Guide

The communication link between the Flight Controller (FC) and the Electronic Speed Controller (ESC) dictates throttle latency, packet jitter, noise rejection, and telemetry capability.

---

## 1. Evolution of ESC Protocols

```
PWM (50Hz - 400Hz) ──► OneShot125 (125-250µs) ──► OneShot42 ──► MultiShot (5-25µs) ──► Digital DShot
  [Analog / High Latency / Jitter]                                    [Pure Digital / CRC Check]
```

### Analog vs Digital Comparison

| Protocol | Signal Type | Update Period / Bit Rate | Throttle Resolution | Calibration Required? | Telemetry Return? |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Standard PWM** | Analog Pulse | 20ms (50Hz) - 2.5ms (400Hz) | ~1000 steps | Yes | No |
| **OneShot125** | Analog Pulse | 125µs – 250µs | ~1000 steps | Yes | No |
| **MultiShot** | Analog Pulse | 5µs – 25µs | ~2048 steps | Yes | No |
| **DShot150** | Digital Stream | 150 kbit/s (106.7µs frame) | 2048 steps (11-bit) | **No** | No (or via separate wire) |
| **DShot300** | Digital Stream | 300 kbit/s (53.3µs frame) | 2048 steps (11-bit) | **No** | Yes (Bidirectional) |
| **DShot600** | Digital Stream | 600 kbit/s (26.7µs frame) | 2048 steps (11-bit) | **No** | Yes (Bidirectional) |
| **DShot1200** | Digital Stream | 1200 kbit/s (13.3µs frame) | 2048 steps (11-bit) | **No** | Yes (Bidirectional) |

---

## 2. Digital DShot Protocol Structure

Each DShot packet sent from FC to ESC consists of **16 bits**:

```
[ Bit 0 - 10: Throttle Value (11 bits) ] [ Bit 11: Telemetry Request (1 bit) ] [ Bit 12 - 15: CRC Checksum (4 bits) ]
  • Values 0: Disarmed                     • 1 = Request serial telemetry packet   • CRC = nibble0 ^ nibble1 ^ nibble2 of Bits[0-11]
  • Values 1 - 47: Special commands        • 0 = Standard throttle command           (inverted with bitwise NOT when bidirectional/telemetry mode is active)
  • Values 48 - 2047: Throttle (0% - 100%)
```

### DShot Special Commands (Disarmed, values 1-47)
* `CMD 1-5`: `DSHOT_CMD_BEACON1`-`BEACON5` — Beep tones 1 through 5 (lost-model beacon)
* `CMD 6`: `DSHOT_CMD_ESC_INFO` — Request ESC info packet
* `CMD 7 / 8`: `DSHOT_CMD_SPIN_DIRECTION_1` / `_2` — Legacy spin-direction select (requires repeated sends)
* `CMD 9 / 10`: `DSHOT_CMD_3D_MODE_OFF` / `_ON` — Disable/enable bidirectional 3D mode
* `CMD 11`: `DSHOT_CMD_SETTINGS_REQUEST`
* `CMD 12`: `DSHOT_CMD_SAVE_SETTINGS` — Save settings to EEPROM
* `CMD 13 / 14`: `DSHOT_CMD_EXTENDED_TELEMETRY_ENABLE` / `_DISABLE`
* `CMD 20 / 21`: `DSHOT_CMD_SPIN_DIRECTION_NORMAL` / `_REVERSED` — Modern persistent spin-direction select. **Not** a turtle-mode command; see [Advanced ESC Features](advanced-esc-features.md) for how crash-flip actually reverses motors.
* `CMD 22-29`: `DSHOT_CMD_LED0_ON`-`LED3_ON`/`_OFF` (BLHeli_32 only)
* `CMD 30`: `DSHOT_CMD_AUDIO_STREAM_MODE_ON_OFF` (KISS)
* `CMD 31`: `DSHOT_CMD_SILENT_MODE_ON_OFF` (KISS)

Source: [betaflight/src/main/drivers/dshot_command.h](https://github.com/betaflight/betaflight/blob/master/src/main/drivers/dshot_command.h).

---

## 3. Bidirectional DShot & RPM Filtering

Bidirectional DShot allows the FC to transmit throttle commands down the motor signal wire, and the ESC immediately responds back across the **same single wire** with an inverted **e-RPM telemetry packet (EDT - Extended DShot Telemetry)**.

```
FC ───── [16-bit DShot Command Packet] ─────► ESC (Commutates Motor)
FC ◄──── [16-bit Inverted e-RPM / CRC] ────── ESC (Reports instantaneous RPM)
```

### Why RPM Filtering is Critical in Betaflight / QuickPID:
1. **Dynamic Harmonic Notch Filters:** The FC calculates the exact acoustic and mechanical motor noise frequencies:
   $$\text{Noise Frequency (Hz)} = \frac{\text{e-RPM}}{60} \times \text{Harmonic Number}$$
2. **Phase Delay Elimination:** Dynamic notch filters track motor speed precisely, eliminating the need for aggressive static low-pass gyro filters, drastically reducing control loop latency and eliminating propwash oscillations.

---

## 4. ESC Telemetry Interfaces

1. **Bidirectional DShot (Single Wire):** Primary method for ultra-fast RPM telemetry (every PID loop cycle).
2. **Dedicated ESC Telemetry (KISS / BLHeli_32 / AM32 Serial Wire):**
   * Transmits operational telemetry at ~50Hz–100Hz:
     * Motor RPM
     * Real-time Voltage ($V_{bat}$)
     * Real-time Current ($I_{amps}$)
     * Cumulative mAh Consumed
     * ESC MOSFET Internal Temperature ($^\circ\text{C}$)

---

*Related Documentation:*
* [Theory of Operation & Hardware](theory-operation-hardware.md)
* [Betaflight ESC Integration](betaflight-esc-tuning.md)
* [BLHeli_32 Guide](blheli32.md)
* [Bluejay Guide](bluejay.md)
* [AM32 Guide](am32.md)
* [Back to Knowledge Base Index](README.md)
