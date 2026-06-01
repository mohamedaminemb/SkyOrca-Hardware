# Design Decisions & Rationale

This document captures the key cross-board design decisions made during the SkyOrca hardware development, explaining *why* each choice was made rather than just *what* was chosen.

---

## 1. Hardware/Software Co-Design Approach

SkyOrca was developed using a **HW/SW co-design** methodology — hardware and software decisions were made jointly and iteratively, rather than sequentially.

This was essential because:

- The IMU interface (SPI vs I²C) directly determines whether the firmware can achieve a 400 Hz control loop via DMA, or must fall back to polling at lower rates.
- The number of hardware UARTs on the SkyOrcaRX MCU determines whether telemetry can be handled without software multiplexing — which would introduce non-deterministic latency.
- PCB trace lengths and layer assignment affect which sensors can share buses, directly constraining firmware architecture.

Every component table in this repository reflects this joint reasoning.

---

## 2. Why 4-Layer PCBs for All Three Boards

Even the smaller SkyOrcaTX and SkyOrcaRX boards were routed on 4 layers, despite lower component density. The reasons:

- **Continuous ground plane (In1.Cu)** — essential for XBee RF reliability. A fragmented ground plane raises return path impedance and creates unintentional loop antennas, degrading link margin.
- **Dedicated power plane (In2.Cu)** — isolates DC-DC switching noise from signal layers.
- **Signal integrity** — high-frequency SPI signals (up to 20 MHz on the IMU) require a reference plane directly beneath the trace to maintain controlled impedance and minimize crosstalk.

2-layer boards were evaluated and rejected: the fragmented ground plane would have created inductive coupling between the 8 kHz ESC PWM signals and the SPI bus serving the IMU.

---

## 3. SkyOrcaFC Revision 1 → Revision 2

The first PCB revision (50×50 mm) was rejected after layout review with LAB619. Two structural problems were found:

**Problem A — GPS near DC-DC converters**

The TPS5430 converters switch at approximately 500 kHz. Their harmonics extend into the tens of MHz range, and the common-mode noise they inject into the power plane can degrade the GPS receiver sensitivity (signal level ≈ −130 dBm). In revision 1, the GPS connector was less than 5 mm from the converter components with no keepout zone around the antenna.

Fix in revision 2:
- GPS block moved to the opposite corner from the DC-DC island (> 15 mm separation).
- 10 mm RF keepout zone added around the GPS antenna footprint — no copper pours allowed inside.

**Problem B — Thermal density around IMU**

Both converters were placed close to the MPU-6000. A temperature rise of even +10 °C on the IMU introduces a gyroscope bias that is not compensated by the complementary filter and directly degrades attitude estimation accuracy.

Fix in revision 2:
- DC-DC converters grouped in a dedicated thermal corner with their passives (inductor, diode, bulk caps).
- STM32F405 placed at the board center, minimising maximum trace length to all peripherals.

The 20% area increase (50×50 → 60×50 mm) was accepted as fully justified by the elimination of these two failure modes.

---

## 4. Logic Level Unification at 3.3V

All three boards operate at **3.3V logic** — the STM32F405 is natively 3.3V, and the ATmega328P / ATmega2560 are powered at 3.3V as well (within their operating range at 8 MHz).

This eliminates the need for any level shifters on the inter-board UART links (`UART2` between SkyOrcaRX and SkyOrcaFC, `MSP_TX_MCU` telemetry path), reducing component count and removing a potential failure point.

---

## 5. XBee Over nRF24L01 / LoRa

| Criterion | nRF24L01 | LoRa SX1276 | XBee 802.15.4 |
|-----------|----------|-------------|---------------|
| MCU interface | SPI | SPI / UART | UART |
| Bidirectional | Yes | Partial | Native |
| Firmware complexity | High | High | Low |
| Range | ~100 m | Several km | ~1.6 km |
| Telemetry support | No | Partial | Yes |

The nRF24L01 was rejected because it requires the firmware to implement the full packet management layer (auto-ACK, pipe addressing, retransmit logic) over SPI — adding significant firmware complexity on a board (SkyOrcaRX) that has no FPU and limited flash.

LoRa was rejected because its long-range capability introduces latency (Time-on-Air) incompatible with real-time control link requirements at typical drone operating distances (< 500 m).

XBee operates over IEEE 802.15.4 and presents a simple UART interface to the MCU — the module handles the entire MAC layer internally.

---

## 6. Flash Memory for Flight Data Logging

The W25Q128JVS (128 Mbit NOR Flash) is connected on a **dedicated SPI2 bus** on the SkyOrcaFC, separate from the IMU's SPI3 bus. This ensures that asynchronous write operations to the flash (triggered at a lower rate than the 400 Hz control loop) never create bus contention that could delay an IMU read.

Quad-SPI capability (133 MHz) allows burst writes during low-activity phases without blocking the control loop.

---

## 7. Partnership with LAB619

The collaboration with LAB619 Engineering & Consulting Services was not optional — it was a technical necessity for two reasons:

1. **4-layer PCB manufacturing** — No Tunisian PCB fabricator offers 4-layer boards at academic project scale. All boards were manufactured by JLCPCB (Shenzhen) and imported through LAB619's registered commercial entity.
2. **Component sourcing** — The Tunisian electronics distribution market is limited to basic passives and entry-level microcontrollers. Every active component in SkyOrca (STM32F405, MPU-6000, MS5611, MAX-M10S, XBee, TPS5430) was unavailable locally and required international procurement.

LAB619 also contributed PCB design review (DRC validation, signal integrity analysis) and access to professional assembly equipment (reflow station, oscilloscopes, logic analyzers).
