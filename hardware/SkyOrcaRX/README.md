# SkyOrcaRX — Receiver / Communication Module

> Full design documentation: [`docs/SkyOrcaRX.md`](../../docs/SkyOrcaRX.md)

## Contents

```
SkyOrcaRX/
├── kicad/
│   ├── SkyOrcaRX.kicad_pro
│   ├── SkyOrcaRX.kicad_sch
│   ├── SkyOrcaRX.kicad_pcb
│   └── schematic.pdf
└── bom/
    (BOM to be exported from KiCad project)
```

## Quick Specs

| Parameter | Value |
|-----------|-------|
| MCU | ATmega2560-16A @ 8 MHz |
| Dimensions | 30 × 25 mm |
| Layers | 4 |
| Power input | 3.3V from SkyOrcaFC |
| Radio | XBee 802.15.4 |
| UARTs | 4 (radio / FC link / debug / spare) |
| Revision | 1 |
