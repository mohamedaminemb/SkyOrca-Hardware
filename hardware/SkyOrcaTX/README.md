# SkyOrcaTX — Remote Controller Board

> Full design documentation: [`docs/SkyOrcaTX.md`](../../docs/SkyOrcaTX.md)

## Contents

```
SkyOrcaTX/
├── kicad/
│   ├── SkyOrcaTX.kicad_pro
│   ├── SkyOrcaTX.kicad_sch
│   ├── SkyOrcaTX.kicad_pcb
│   └── schematic.pdf
└── bom/
    └── ibom.html                       # Interactive BOM (open in browser)
```

## Quick Specs

| Parameter | Value |
|-----------|-------|
| MCU | ATmega328P-P @ 8 MHz |
| Dimensions | 50 × 35 mm |
| Layers | 4 |
| Power input | 7.4V (LiPo 2S) |
| Radio | XBee 802.15.4 (up to 1.6 km) |
| Revision | 1 |

## Ordering

No Gerber zip is included in this release — contact the project team or generate from the KiCad project. Use the same JLCPCB 4-layer settings as SkyOrcaFC.
