# How the beacon fits into the wayfinding system

This repository holds one part of a larger team project: an indoor wayfinding
system for a hospital, where GPS is unusable indoors. The beacon is the piece
that makes position measurable at all.

## The chain

```
beacon (this repo)  ──BLE advertising──▶  Android app  ──▶  3D map
   firmware                                 scans RSSI          draws position
   custom PCB                               smooths it
   txPower calibration                      converts to distance
```

The beacon advertises a fixed payload that carries its own identifier and, in
byte 11, the `txPower` value — the RSSI this specific hardware produces at one
metre. The app needs that byte to turn a measured RSSI into a distance:

```
distance = 10 ^ ((txPower − rssi) / 20)
```

Which is why the calibration described in
[rf-coverage.md](rf-coverage.md) matters more than it first appears: a wrong
`txPower` does not degrade the estimate gracefully, it scales every distance in
the system by a constant factor.

## Who built what

**Mine:**

- the beacon firmware, on both the nRF52840 and the EFR32BG22 builds
- the PCB: schematic, hand-drawn symbol and footprint for the radio module,
  layout, and the gerbers in `hardware/gerber/`
- the RSSI field measurements and the `txPower` calibration that came out of
  them, 370 samples over two independent sessions
- the admin panel that assigns a beacon to a physical place

**Other members of the team:**

- the 3D map
- the Android wayfinding application that consumes the advertisements

The application and map live in the team's own repository, which is private.
What is published here is the hardware and firmware side, plus the measurement
notes, because those are the parts I can speak for.

## The boundary that was found by measuring

The coverage tables in [rf-coverage.md](rf-coverage.md) end on a conclusion
worth repeating here, because it changed where the team looked for a fix:

The radio already reaches far enough. At +4 dBm with a calibrated
`RSSI@1m = −69 dBm`, a phone still hears the beacon at roughly 11 m in an
ordinary indoor environment. The limit is the app's `minRssi = −85` constant,
which discards everything weaker and throws away more than half the range the
radio produced. Widening that threshold costs nothing; raising transmit power
would have cost battery life for no gain.
