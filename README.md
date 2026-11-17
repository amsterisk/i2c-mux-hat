# I2C Mux HAT

An open-hardware 8-channel I2C multiplexer board for single-board computers with a
Raspberry Pi-compatible 40-pin header (Raspberry Pi, Orange Pi, Banana Pi, ...).

Status: **work in progress** — schematic done (ERC clean), PCB layout not started.

## Features

- **TCA9548A** 8-channel I2C switch, address 0x70–0x77 (solder jumpers, default 0x70).
- **Per-channel voltage**: a 3-pin jumper selects 3.3 V or 5 V for that channel's pull-ups
  and connector power. The mux runs at 3.3 V and translates levels, so 5 V devices never
  reach the host's GPIO.
- **Per-channel connectors**: RJ9 (4P4C) jack *and* a 0.1" 4-pin header footprint.
- 2.2 kΩ pull-ups per channel (removable via solder jumper), suitable for a few metres of cable.
- Per-channel resettable fuse on connector power and ESD clamp (USBLC6-2SC6) on SDA/SCL/V.
- Optional mux reset from GPIO4 (solder jumper).
- Optional Raspberry Pi HAT ID EEPROM footprint (not fitted by default).
- HAT form factor, 65 × 56.5 mm.

Example use: DDC/CI control of monitors (input switching, brightness) over HDMI's DDC lines,
with one channel per monitor — every monitor answers on the same addresses (0x37/0x50), so
each needs its own bus.

## Connector pinouts

| RJ9 contact | Signal | | 0.1" header pin | Signal |
|---|---|---|---|---|
| 1 | SCL | | 1 | GND |
| 2 | GND | | 2 | V (3.3/5 V) |
| 3 | V (3.3/5 V) | | 3 | SDA |
| 4 | SDA | | 4 | SCL |

RJ9 contact 1: plug held with latch underneath, contacts up, cable toward you → leftmost.
**The RJ9 jacks are not telephone ports. Never connect them to a phone line.**

### RJ9 → HDMI (DDC) cable

| RJ9 | HDMI pin |
|---|---|
| 1 SCL | 15 |
| 2 GND | 17 (bridge to 20 / shell) |
| 3 V | 18 (+5 V) |
| 4 SDA | 16 |

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

## Repository layout

- `hardware/` — KiCad 9 project (`i2c-mux-hat.kicad_pro`); the 8 channels are one hierarchical
  sheet (`channel.kicad_sch`) instanced 8 times. Project footprints in `i2c-mux-hat.pretty/`.
- `LICENSE` — CERN-OHL-S-2.0.

## Licence

Copyright the contributors. Licensed under the **CERN Open Hardware Licence Version 2 –
Strongly Reciprocal** (CERN-OHL-S-2.0). See `LICENSE`.
