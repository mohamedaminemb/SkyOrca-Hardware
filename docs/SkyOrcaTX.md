# SkyOrcaTX — Remote Controller Board

## Overview

The SkyOrcaTX is the hand-held remote controller. It reads pilot inputs from two analog joysticks and two buttons, transmits commands wirelessly to the drone via XBee, and displays telemetry data received back from the drone on an OLED screen.

- **MCU:** ATmega328P-P @ 8 MHz (DIP-28)
- **Radio:** XBee 802.15.4
- **PCB:** 4 layers, 50 × 35 mm
- **Power input:** 7.4V LiPo (2S)

---

## Component Selection

### Microcontroller — ATmega328P-P

The ATmega328P was chosen for its 6 ADC input channels (reading 4 joystick axes on PC0–PC3 + 2 spare), 1 hardware UART (XBee communication), DIP-28 package for prototyping ease, and low cost. The ATtiny85 was ruled out due to insufficient UART and ADC count; the ATmega2560 was overkill for this role.

Clocked at 8 MHz by an external crystal with 20 pF load capacitors.

### Radio — XBee 802.15.4

XBee was selected over nRF24L01 and LoRa SX1276 for three reasons:
1. **UART-native interface** — no low-level SPI protocol management in firmware; the module handles framing, addressing, and retransmission internally.
2. **Native bidirectional link** — the same module handles both TX (commands) and RX (telemetry) without additional configuration.
3. **Proven RF robustness** — the 802.15.4 PHY layer provides reliable links in RF-noisy environments typical of outdoor drone operation.

Range: up to 1.6 km line-of-sight.

Two XBee footprints (10-pin connectors) are provided for redundancy / future upgrade path.

### Power — TPS5430DDA (7.4V → 3.3V)

Same converter family as the SkyOrcaFC for supply chain simplicity. Feedback divider: R7 (10 kΩ) / R8 (5.9 kΩ).

---

## Schematic Notes

**Sheet 1 — ATmega328P and interfaces:**
- `VRXG`, `VRYG` → left joystick X/Y on `ADC0`, `ADC1`
- `VRXD`, `VRYD` → right joystick X/Y on `ADC2`, `ADC3`
- `RXD_MCU_xbee` / `TXD_MCU_xbee` → XBee UART
- I²C connector available for OLED display (`SSD1306` or equivalent)
- ICSP/SWD connector for in-circuit programming
- Reset button with 10 kΩ pull-up and 100 nF debounce capacitor
- 100 nF decoupling on VCC and AVCC pins

**Sheet 2 — Power:**
- TPS5430DDA in 7.4V→3.3V configuration
- 10 µH inductor + Schottky diode + 560 µF + 4.7 µF bulk output capacitors
- Four M3 mounting holes for enclosure integration

---

## Joystick Mapping

| Signal | ATmega Pin | Axis |
|--------|-----------|------|
| VRXG | PC0 / ADC0 | Left joystick — X (Roll) |
| VRYG | PC1 / ADC1 | Left joystick — Y (Throttle) |
| VRXD | PC2 / ADC2 | Right joystick — X (Yaw) |
| VRYD | PC3 / ADC3 | Right joystick — Y (Pitch) |
