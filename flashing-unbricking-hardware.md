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
# Using STM32_Programmer_CLI (Official ST Tool) — verified against ST's own CLI syntax
STM32_Programmer_CLI -c port=SWD -rdu
# -rdu: Readout Unprotect (Mass erases flash memory and drops RDP from Level 1 to Level 0)

# Flashing the AM32 Bootloader & Firmware
STM32_Programmer_CLI -c port=SWD -w AM32_TARGET_NAME.hex -v -rst
```

> `mode=UR` is not a real option for `-c port=SWD` — do not include it.

```bash
# Using OpenOCD — RDP is an OPTION BYTE, not the same thing as flash write-protection.
# The generic "<driver> lock / unlock" commands only touch FLASH_CR write-protection and
# will NOT remove RDP.
# Command pattern sourced from ST's own community thread on this exact STM32G0x + OpenOCD problem
# (community.st.com "OpenOCD and RDP protection [STM32g0x]"); the stm32l4x driver also covers G0/G4.
openocd -f interface/stlink.cfg -f target/stm32g0x.cfg \
  -c "init" \
  -c "reset halt" \
  -c "stm32l4x option_write 0 0x20 0xaa 0xaa" \
  -c "stm32l4x option_load 0" \
  -c "reset" \
  -c "shutdown"
```

> **Caution:** the register offset (`0x20`) and value (`0xaa` = RDP Level 0) above matched the specific STM32G0x case discussed in the sourced thread — confirm the correct option-byte offset for your exact target/OpenOCD version before running this, since it mass-erases flash. When in doubt, prefer `STM32_Programmer_CLI` above — it's the officially documented tool for this operation and doesn't require guessing a register offset.

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
