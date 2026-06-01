# SkyOrcaRX — Receiver / Communication Module

## Overview

The SkyOrcaRX sits on the drone between the XBee radio link and the SkyOrcaFC flight controller. It decodes incoming command frames from the remote controller and forwards them to the FC over UART, while also relaying telemetry data from the FC back to the transmitter.

- **MCU:** ATmega2560-16A @ 8 MHz
- **Radio:** XBee 802.15.4
- **PCB:** 4 layers, 30 × 25 mm
- **Power:** 3.3V supplied by SkyOrcaFC (no on-board regulator)

---

## Component Selection

### Microcontroller — ATmega2560-16A

The ATmega2560 was chosen specifically for its **4 independent hardware UART channels**:

| UART | Assignment |
|------|-----------|
| UART3 | XBee radio (receive commands, transmit telemetry) |
| UART2 | SkyOrcaFC link — `IBUS_RX_MCU` signal |
| UART0 | Debug / serial monitor |
| UART1 | Reserved for future extensions |

The ATmega328P (1 UART) and STM32F103 (3 UARTs) were ruled out due to insufficient UART count without multiplexing, which would introduce latency and firmware complexity in a time-critical link.

---

## Schematic Notes

Single-sheet design. Key circuits:

- **Quartz:** 8 MHz with 22 pF load capacitors
- **UART3 XBee:** dual 10-pin connectors (`UART3_XBEE1`, `UART3_XBEE2`)
- **UART2:** 2-pin connector → `IBUS_RX_MCU` on SkyOrcaFC
- **UART0:** 3-pin debug connector
- **ICSP/SPI:** MOSI / MISO / SCK / RESET for ATmega2560 programming
- **Extensions:** J3 — GPIO1, GPIO2, GPIO3/PWM; I²C connector (SDA/SCL)
- **Power:** `POWER1` 3-pin connector — 3.3V input from SkyOrcaFC; no local regulator
- **Reset:** SW1 with 10 kΩ pull-up
- **Decoupling:** 100 nF on each VCC pin; 22 pF on AREF

---

## Data Flow

```
SkyOrcaTX (XBee TX)
       │  RF 2.4 GHz
       ▼
SkyOrcaRX — XBee RX
       │  UART3 receive
       ▼
ATmega2560 — decode & forward
       │  UART2 → IBUS_RX_MCU
       ▼
SkyOrcaFC — flight control loop

◄──────────────────────────────
  Telemetry path (reverse):
  SkyOrcaFC MSP_TX_MCU → UART2 → ATmega2560 → UART3 → XBee TX → SkyOrcaTX
```
