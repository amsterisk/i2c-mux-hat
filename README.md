# I2C Mux HAT

An open-hardware 8-channel I2C multiplexer board for single-board computers with a
Raspberry Pi-compatible 40-pin header (Raspberry Pi, Orange Pi, Banana Pi, ...).

Status: **work in progress** — schematic done (ERC clean), PCB layout not started.

## Features

- **TCA9548A** 8-channel I2C switch, address 0x70–0x77 (solder jumpers, default 0x70).
- **Per-channel voltage**: a 3-pin jumper selects 3.3 V or 5 V for that channel's pull-ups
  and connector power. The mux runs at 3.3 V and translates levels, so 5 V devices never
  reach the host's GPIO.
- **Per-channel connectors**: keyed JST XH 4-pin (B4B-XH-A, vertical) *and* a 0.1" 4-pin header,
  same pinout on both.
- 2.2 kΩ pull-ups per channel (removable via solder jumper), suitable for a few metres of cable.
- Per-channel resettable fuse on connector power and ESD clamp (USBLC6-2SC6) on SDA/SCL/V.
- Optional mux reset from GPIO4 (solder jumper).
- Optional Raspberry Pi HAT ID EEPROM footprint (not fitted by default).
- HAT form factor, 65 × 56.5 mm.

Example use: DDC/CI control of monitors (input switching, brightness) over HDMI's DDC lines,
with one channel per monitor — every monitor answers on the same addresses (0x37/0x50), so
each needs its own bus.

## Connector pinout

JST XH (J*x*01) and 0.1" header (J*x*02) share one pinout:

| Pin | Signal |
|---|---|
| 1 | GND |
| 2 | V (3.3 or 5 V, per channel jumper) |
| 3 | SDA |
| 4 | SCL |

### JST XH → HDMI (DDC) cable

| XH pin | HDMI pin |
|---|---|
| 1 GND | 17 (bridge to 20 / shell) |
| 2 V | 18 (+5 V) |
| 3 SDA | 16 |
| 4 SCL | 15 |

Set that channel to **5 V**. Leave all other HDMI pins unconnected.

## Jumpers

| Ref | Default | Function |
|---|---|---|
| JPx01 (header) | no shunt | Channel x voltage: 1-2 = 3.3 V, 2-3 = 5 V, none = off |
| JPx02 (solder) | bridged | Channel x pull-ups; cut to remove |
| JP1 (solder) | open | Bridge to connect mux ~RESET to GPIO4 (pin 7) |
| JP2–JP4 (solder) | GND | Address A0–A2; move to 3V3 for 0x71–0x77 |
| JP5 (solder) | open | HAT EEPROM write-protect (only with EEPROM fitted) |

## Software

The board appears as one I2C device (0x70) on the host's bus on pins 3/5.

- **Raspberry Pi OS**: `dtoverlay=i2c-mux,pca9548,addr=0x70` in `config.txt` gives one
  `/dev/i2c-N` per channel. Keep the bus at ≤100 kHz for long cables / DDC
  (`dtparam=i2c_baudrate=100000`); the TCA9548A is rated to 400 kHz.
- **Other boards**: use a device-tree overlay for `i2c-mux-pca954x`, or select channels
  from userspace by writing the channel bitmask to 0x70 before each transaction.

## Building fabrication files

```sh
tools/export.sh            # version from `git describe`; refuses to build if ERC/DRC fail
tools/export.sh v0.1       # explicit version
tools/export.sh --allow-violations   # work-in-progress builds
```

Outputs land in `production/<version>/` (git-ignored; attach to a GitHub release):
Gerber + drill zip (upload as-is to JLCPCB/PCBWay; the stencil is made from `F.Paste`),
grouped BOM, JLCPCB-format BOM + placement file, schematic PDF, top/bottom 3D renders,
and the ERC/DRC reports. Needs KiCad 9 (`kicad-cli`, or `kicad.kicad-cli` from the snap).

Design rules target JLCPCB's standard 2-layer process (Board Setup → Constraints, plus
`hardware/i2c-mux-hat.kicad_dru`): 0.15 mm track/clearance minimum, 0.6/0.3 mm vias,
0.3 mm copper-to-edge. Net classes: *Default* 0.25 mm tracks; *Power* 0.5 mm for
+5V, +3.3V, GND and each channel's VCH/VOUT/VPU.

## Repository layout

- `hardware/` — KiCad 9 project (`i2c-mux-hat.kicad_pro`); the 8 channels are one hierarchical
  sheet (`channel.kicad_sch`) instanced 8 times.
- `tools/export.sh` — builds fabrication and documentation outputs.
- `LICENSE` — CERN-OHL-S-2.0.

## Licence

Copyright the contributors. Licensed under the **CERN Open Hardware Licence Version 2 –
Strongly Reciprocal** (CERN-OHL-S-2.0). See `LICENSE`.
