# Beacon hardware — beacon_v1.0

Custom two-layer PCB designed in KiCad 10.0.3. Every schematic symbol and
footprint for the radio module was drawn by hand: the module is not a
development kit, it is a bare E104-BT53A1 soldered onto this board and flashed
over SWD.

| Spec | Value |
|---|---|
| Board size | 35.05 × 35.05 mm |
| Layers | 2 (F.Cu / B.Cu) |
| Thickness | 1.6 mm |
| Designed in | KiCad 10.0.3 |
| Radio module | EBYTE E104-BT53A1 (Silicon Labs EFR32BG22C112F352GM32) |
| Supply | 2 × AA direct, 1.9–3.6 V — no regulator needed |
| TX power | +4 dBm (chip maximum +6 dBm) |

Every figure above is read back from `gerber/beacon_v1.0-job.gbrjob`, which is
the file the board house was sent.

## Bill of materials

| Ref | Part | Note |
|---|---|---|
| U1 | E104-BT53A1 | BLE 5.3 module, hand-drawn symbol + footprint |
| C1, C2 | Decoupling | placed at the module supply pins |
| J1 | Battery input | 2 × AA |
| J2 | SWD programming header | see pinout below |

## SWD programming pinout

| J-Link | Module pin | Note |
|---|---|---|
| VCC 3.3 V | 14 or 15 (VCC) | never 5 V |
| GND | 1 / 9 / 10 / 16 / 17 | |
| SWCLK | 6 (PA01) | |
| SWDIO | 7 (PA02) | |
| RESET | 26 (RST) | strongly recommended — without it a bad flash means power-cycling by hand |

## Manufacturing

`gerber/` holds the files sent to the board house: 2 copper layers, 2 solder
mask, 2 silkscreen, edge cuts, and PTH/NPTH drill files.

## Why not a dev kit

A dev kit carries a USB bridge, regulators and LEDs that a wall-mounted beacon
never uses. They cost current the two AA cells have to pay for, and area the
enclosure does not have. Running the bare module means the board holds only
what the beacon needs.
