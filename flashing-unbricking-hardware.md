# ESC Flashing, ST-Link Unbricking & Hardware Modding

This guide covers advanced firmware flashing workflows, recovering corrupted/bricked ESCs via hardware debuggers (SWD / ST-Link), unlocking Readout Protection (RDP), and replacing bootloaders.

---

## 1. Web & Passthrough Flashing vs Direct SWD Flashing

```
[ Flight Controller Passthrough ] ───── (Standard) ──► Flash over Betaflight USB (4-way-if / MSP)
[ Hardware Debugger (ST-Link)  ] ───── (Recovery)  ──► Direct SWD (SWDIO, SWCLK, GND) to MCU Pads
```

### 1. FC Passthrough (Standard Workflow)
* Uses Betaflight's built-in 4-way interface or MSP pass-through to send firmware bytes directly over the single DShot signal wire.
* Tools: [esc-configurator.com](https://esc-configurator.com/) (Bluejay), [am32.ca](https://am32.ca/) (AM32), BLHeli_32 Suite.

---

## 2. Unbricking & Flashing AM32 / ESCape32 to 32-Bit ESCs via ST-Link

When flashing locked BLHeli_32 boards or recovering corrupted microcontrollers, you must connect directly to the microcontroller's Serial Wire Debug (SWD) port.

### Required Pin Connections:
* **SWDIO:** Serial Wire Data Input/Output
* **SWCLK:** Serial Wire Clock
* **GND:** Ground reference
* **VCC / Power:** 3.3V logic (or power the ESC with a current-limited DC power supply / Smoke Stopper).

### Unlocking Readout Protection (RDP) on STM32 / AT32:
Locked BLHeli_32 ESCs have RDP Level 1 enabled to prevent firmware dumping. To flash open-source firmware:

```bash
# Using STM32_Programmer_CLI (Official ST Tool)
STM32_Programmer_CLI -c port=SWD mode=UR -rdu
# -rdu: Readout Unprotect (Mass erases flash memory and unlocks chip)

# Flashing the AM32 Bootloader & Firmware
STM32_Programmer_CLI -c port=SWD -w AM32_TARGET_NAME.hex -v -rst
```

```bash
# Using OpenOCD
openocd -f interface/stlink.cfg -f target/stm32g0x.cfg \
  -c "init" \
  -c "reset halt" \
  -c "stm32g0x unlock 0" \
  -c "flash write_image erase AM32_TARGET_NAME.hex" \
  -c "reset run" \
  -c "shutdown"
```

---

## 3. Hardware Target Mapping & Pin Identification

Before flashing third-party firmware, you must match the exact hardware target pinout:
1. **Gate Driver Configuration:** Dual-FET half-bridges, separate High/Low PWM pins, or inverted gate inputs.
2. **Current Shunt ADC Pin:** Analog sense input for current telemetry.
3. **Phase Voltage BEMF Sense Pins:** Resistor divider pins tied to internal MCU comparators or ADCs.
4. **Target Naming Convention (AM32 / ESCape32):**
   * Example: `AT421_G071_FD6288_40A` specifies AT32F421 / STM32G071 MCU with Fortior FD6288 gate driver IC.

---

*Related Documentation:*
* [BLHeli_32 Technical Guide](blheli32.md)
* [AM32 Technical Guide](am32.md)
* [ESCape32 Technical Guide](escape32.md)
* [Back to Knowledge Base Index](README.md)
