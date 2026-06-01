# SkyOrcaFC — Flight Controller Board

## Overview

The SkyOrcaFC is the central board of the SkyOrca drone. It is responsible for reading all onboard sensors, running the attitude estimation and stabilization algorithms, and generating the PWM commands sent to the four ESCs.

- **MCU:** STM32F405RGTx (Cortex-M4F @ 168 MHz, hardware FPU)
- **PCB:** 4 layers, 60 × 50 mm (revision 2)
- **Power input:** 12V from main LiPo battery

---

## Component Selection

### Microcontroller — STM32F405RGTx

| Family | Core | FPU | Freq | SPI buses |
|--------|------|-----|------|-----------|
| STM32F1 | Cortex-M3 | ✗ | 72 MHz | 2 |
| STM32F3 | Cortex-M4 | ✓ | 144 MHz | 3 |
| **STM32F405** | **Cortex-M4F** | **✓** | **168 MHz** | **3** |
| STM32F7 | Cortex-M7 | ✓ | 280 MHz | 6 |

The F405 was chosen for its hardware FPU (quaternion math and complementary filter at 400 Hz), two independent SPI buses (one dedicated to the IMU, one to flash memory), and advanced timers for high-frequency PWM generation on ESC outputs.

### IMU — MPU-6000

The MPU-6000 was preferred over the MPU-6050 (I²C only) due to its SPI interface running up to 20 MHz, combined with a dedicated hardware interrupt line (`GYRO_1_EXTI`). This allows the firmware to trigger DMA reads directly from the EXTI interrupt, achieving a deterministic 400 Hz control loop without any polling.

Gyroscope noise floor: 0.005 °/s/√Hz.

### Barometer — MS5611-01BA

10 cm altitude resolution, 1 µA standby current, I²C interface shared with the magnetometer on `I2C1`. The MS5611 was selected over the BMP280 (≈1 m resolution) because altitude-hold at centimeter precision is a key requirement.

### Magnetometer — HMC5883L

I²C interface, shared on `I2C1` with the barometer. The QMC5883L was rejected due to poor EMI immunity. A migration path to the HMC5983 (adds SPI) is planned for revision 3.

### GPS — u-blox MAX-M10S

Multi-constellation (GPS / GLONASS / Galileo / BeiDou), −167 dBm sensitivity, 11 mA active current. Connected to the STM32 via UART. The ultra-compact LCC package makes it well suited for this form factor.

### Flash — W25Q128JVS

128 Mbit serial NOR Flash, SPI up to 133 MHz, quad-SPI capable. Used for flight data logging and calibration parameter storage. The dedicated `SPI2` bus ensures no contention with the IMU on `SPI3`.

### Power — TPS5430DDA (×2)

| Stage | Input | Output | Load |
|-------|-------|--------|------|
| U5 | 12V | 5V | ESCs, 5V peripherals |
| U7 | 12V | 3.3V | STM32, sensors, GPS, flash |

Both converters use a feedback divider sized for exact output voltage, a 10 µH shielded inductor, Schottky diode, and bulk output capacitance. The DC-DC blocks are placed in a thermal island in one corner of the PCB, isolated from sensor components.

---

## Schematic Notes

The schematic is split across two sheets.

**Sheet 1 — MCU & peripherals:**
- `SPI3` → IMU (MOSI/MISO/SCK + `GYRO_1_CS`)
- `SPI2` → Flash W25Q128JVS (`CS_FLASH`)
- `I2C1` → Magnetometer + Barometer (shared bus, 4.7 kΩ pull-ups)
- `GYRO_1_EXTI` → IMU INT pin → STM32 EXTI for DMA trigger
- `UART` → GPS (`GPS_TX`, `GPS_RX`)
- `UART` → SkyOrcaRX (`IBUS_RX_MCU`, `MSP_TX_MCU`)
- `TIM` outputs → `Signal_ESC1` … `Signal_ESC4` (dedicated PWM timer channels)
- USB Micro-B with ESD protection, 22 Ω series resistors on D+/D−
- SWD connector (SWDIO / SWDCLK / NRST)

**Sheet 2 — Sensors, power, flash:**
- Each sensor has a dedicated 100 nF decoupling cap placed within 2 mm of the VDD pin.
- VDDA separated from VDDD by a 39 nH ferrite bead + 4.7 nF + 100 nF.
- Battery voltage divider → ADC pin (`Niveaux_Battrie`).

---

## PCB Revisions

### Revision 1 — 50 × 50 mm

Initial compact layout. Two critical problems were identified:

1. **GPS block adjacent to DC-DC converters** — Switching harmonics (fsw ≈ 500 kHz, harmonics up to tens of MHz) at risk of coupling into the GPS antenna path. GPS signal level is approximately −130 dBm; any added noise is significant.
2. **Thermal density too high** — Both TPS5430 converters were within thermal range of the IMU. A +10 °C temperature rise on the MPU-6000 introduces a measurable gyroscope bias offset, degrading attitude estimation.

### Revision 2 — 60 × 50 mm ✅ (production)

20% area increase resolves both issues:

| Criterion | Rev 1 | Rev 2 |
|-----------|-------|-------|
| GPS–DC-DC distance | < 5 mm | > 15 mm |
| RF keepout zone | Absent | 10 mm around GPS |
| Thermal isolation | None | DC-DC in dedicated corner |
| SPI trace length | Constrained | Optimised |
| Connector accessibility | Difficult | Good |

The STM32F405 is centred on the board to minimise maximum trace length to all peripherals.

---

## Routing Rules (JLCPCB 4-layer)

| Parameter | Value |
|-----------|-------|
| Signal trace width | 0.2 mm |
| Power trace width | 0.8–1.5 mm |
| ESC PWM trace | 0.5 mm |
| Minimum clearance | 0.15 mm |
| Via diameter | 0.6 mm (0.3 mm drill) |
| GPS antenna keepout | 10 mm |
| Decoupling placement | < 2 mm from VDD pin |
| GPS antenna trace | < 1/12 wavelength (9 mm) |

Layer stackup: `F.Cu (signal)` / `In1.Cu (GND)` / `In2.Cu (PWR)` / `B.Cu (signal)`
