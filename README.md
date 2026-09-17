# BLE Indoor Positioning — beacon

A Bluetooth Low Energy beacon for indoor positioning: custom two-layer PCB,
firmware for two different radio chips, and the RSSI measurements that make the
distance estimate mean anything.

Built as the hardware half of a team wayfinding system for a hospital, where
GPS does not work indoors. See [docs/integration.md](docs/integration.md) for
what I built and what teammates built.

| | |
|---|---|
| Board | 35.05 × 35.05 mm, 2 layers, KiCad 10.0.3 — no development kit |
| Radio | EBYTE E104-BT53A1 (EFR32BG22), ported from nRF52840 |
| RSSI samples measured | **370**, two independent sessions |
| txPower calibration | **−59 → −69 dBm** |
| Flash budget | 256 KB used of 352 KB |

---

## The calibration that mattered most

Byte 11 of the advertising payload is `txPower`: the RSSI this hardware
actually produces at one metre. An app turns a measured RSSI into a distance
with it:

```
distance = 10 ^ ((txPower − rssi) / 20)
```

The firmware shipped with **−59 dBm** — the generic iBeacon default, copied
across from the older nRF52840 build and never measured on this hardware. The
measured value is **−69 dBm**.

Ten decibels does not produce a slightly worse estimate. Standing exactly one
metre from the beacon, the app computed:

```
10 ^ ((−59 − (−69)) / 20) = 10 ^ 0.5 = 3.16 m
```

Every distance in the system was wrong by the same factor of 3.2, at every
range, which is the kind of error that looks like a bad smoothing algorithm
until someone measures the constant it is built on.

The second thing the measurements showed: RSSI swung from **−59 to −98 dBm
while the beacon sat still on a table**. Roughly 39 dB of spread with nothing
moving — multipath, bodies blocking the path, antenna orientation. That is the
argument for smoothing, written in data rather than asserted.

Full numbers, coverage-radius tables and beacon spacing:
[docs/rf-coverage.md](docs/rf-coverage.md).

---

## Porting nRF52840 → EFR32BG22

The beacon started on a Nordic nRF52840 and moved to a Silicon Labs EFR32BG22.
Different vendor, different SDK, different toolchain, different flash
programmer — and the **advertising payload format was kept byte-for-byte
identical**, so the Android app that reads beacons did not change one line.

Both builds are kept here:

| Directory | Chip | Toolchain |
|---|---|---|
| [`firmware/bt53/`](firmware/bt53) | EFR32BG22C112F352GM32 | Simplicity SDK, GCC, pyOCD |
| [`firmware/e73/`](firmware/e73) | nRF52840 | Arduino core, Adafruit nRF52 bootloader |

Flashing instructions: [docs/flashing-bt53.md](docs/flashing-bt53.md) ·
[docs/flashing-e73.md](docs/flashing-e73.md).

---

## Flash layout

```
0x00000 ┌──────────────────────────────┐
        │ Gecko Bootloader + AppLoader │  ~72 KB   (OTA over BLE)
        ├──────────────────────────────┤
        │ Application                  │  ~176 KB
        ├──────────────────────────────┤
        │ NVM3 — beacon ID             │  end of flash
0x58000 └──────────────────────────────┘
```

The beacon ID lives in NVM3 rather than in the application image. Two
consequences, both of which matter once a beacon is screwed to a wall:

- the ID survives a power cut
- firmware can be updated over BLE without the beacon losing its identity, so
  nobody has to take it down to reflash it

---

## Repository map

| Path | What is in it |
|---|---|
| [`hardware/`](hardware) | Board specs, bill of materials, SWD pinout, gerbers |
| [`firmware/bt53/`](firmware/bt53) | Current firmware, EFR32BG22 |
| [`firmware/e73/`](firmware/e73) | Earlier firmware, nRF52840 |
| [`docs/rf-coverage.md`](docs/rf-coverage.md) | Measurements, calibration, coverage radius |
| [`docs/integration.md`](docs/integration.md) | How the beacon feeds the wayfinding app |
| [`docs/flashing-bt53.md`](docs/flashing-bt53.md) | Flashing the EFR32BG22, including the pitfalls |

The working notes under `docs/` are written in Vietnamese, since that is the
language they were taken in.

---

## Known limits

- `RSSI@1m = −69 dBm` was measured with the module lying on a bench next to its
  programming cable. Once beacons are mounted on a wall the value should be
  re-measured; mounting height and surrounding material move it by a few dB.
  Correcting it means rewriting one byte of the payload, not reflashing.
- The coverage radii are computed from a log-distance path-loss model with an
  assumed environment factor, not measured end to end at each distance.
