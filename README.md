# I2C Mux HAT

An open-hardware 8-channel I2C multiplexer board for single-board computers with a
Raspberry Pi-compatible 40-pin header (Raspberry Pi, Orange Pi, Banana Pi, ...).

Status: **work in progress** — schematic done (ERC clean), PCB routed (DRC clean), not yet reviewed or fabricated.

## Features

- **TCA9548A** 8-channel I2C switch, address 0x70–0x77 (3-way DIP switch, default 0x70).
- **Per-channel voltage**: a 3-pin jumper selects 3.3 V or 5 V for that channel's pull-ups
  and connector power. The mux runs at 3.3 V and translates levels, so 5 V devices never
  reach the host's GPIO.
- **Per-channel connector**: keyed JST XH 4-pin (B4B-XH-A, vertical).
- 2.2 kΩ pull-ups per channel (enabled by a shunt on a 2-pin header), suitable for a few metres of cable.
- Per-channel resettable fuse on connector power and ESD clamp (USBLC6-2SC6) on SDA/SCL/V.
- Optional mux reset from GPIO4 (2-pin header).
- HAT form factor, 65 × 56.5 mm (no ID EEPROM; pins 27/28 unused).
- **2×20 stacking header**: all 40 pins pass through to another board above.

Example use: DDC/CI control of monitors (input switching, brightness) over HDMI's DDC lines,
with one channel per monitor — every monitor answers on the same addresses (0x37/0x50), so
each needs its own bus.

## Connector pinout

JST XH (J*x*01), Grove pin order:

| Pin | Signal |
|---|---|
| 1 | SCL |
| 2 | SDA |
| 3 | V (3.3 or 5 V, per channel jumper) |
| 4 | GND |

### JST XH → HDMI (DDC) cable

| XH pin | HDMI pin |
|---|---|
| 1 SCL | 15 |
| 2 SDA | 16 |
| 3 V | 18 (+5 V) |
| 4 GND | 17 (bridge to 20 / shell) |

Set that channel to **5 V**. Leave all other HDMI pins unconnected.

## Jumpers

| Ref | Default | Function |
|---|---|---|
| JPx01 (1×3 header) | no shunt | Channel x voltage: 1-2 = 3.3 V, 2-3 = 5 V, none = off |
| JPx02 (1×2 header) | shunt fitted | Channel x pull-ups; remove the shunt to disable |
| JP1 (1×2 header) | open | Fit a shunt to connect mux ~RESET to GPIO4 (pin 7) |
| SW1 (DIP-3, 1.27 mm) | all OFF | Address A0–A2; ON = 1, so 0x70 + A (0x70–0x77) |

All jumpers are 2.54 mm pin headers taking standard shunts.

## Stacking

J1 is a 2×20 stacking header (female, long pins through the board), so every GPIO passes
through. A board above can share I2C (pins 3/5) as long as it avoids addresses 0x70–0x77
(or move this board's address with SW1); GPIO4 is only used if JP1 is bridged; pins
27/28 are free for the upper board's ID EEPROM. Mechanically, the vertical XH plugs and
their cables exit upwards, so a board above needs a tall spacer (about 20 mm or more);
otherwise put this board at the top of the stack.

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
