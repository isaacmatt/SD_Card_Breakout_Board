
# microSD Card Breakout (SPI, 3.3 V)

A minimal microSD card breakout board for 3.3 V microcontrollers, exposing the card over a 1-bit SPI interface on a 0.1" header. Designed as a hands-on KiCad project with a clean, manufacturable layout.

> **Status:** Schematic and PCB layout complete. ERC and DRC clean (0 violations, 0 unconnected). **Not yet fabricated or tested** — this is a design-stage project. Footprint assignment, layout, and manufacturing files are done; the board has not been ordered, assembled, or brought up.

---

## Overview

This is a passive breakout: it routes a microSD connector to a pin header with the minimum support circuitry required for reliable operation, and nothing else. No onboard microcontroller, voltage regulator, level shifter, or USB. You supply 3.3 V from a host, jumper the SPI lines across, and talk to the card using a standard SD library on the host side.

The design deliberately targets the common denominator across 3.3 V hosts:  **SPI 1-bit mode** . SDIO/4-bit is intentionally out of scope.

## Features

* microSD (TF) push-push connector with spring eject
* SPI 1-bit interface broken out to a 1×8 0.1" header
* Pull-ups on all data lines per the SD specification
* Local decoupling (100 nF + 1 µF) at the connector's VDD pin
* 2-layer board with ground pours on both layers, via-stitched
* All passives 0603 (hand-solder friendly); designed for low-cost fab + assembly

## Specifications

| Property               | Value                                                    |
| ---------------------- | -------------------------------------------------------- |
| Interface              | SPI, 1-bit                                               |
| Logic / supply voltage | 3.3 V only (**not 5 V tolerant** )                 |
| Layers                 | 2 (signals on top, GND pours both layers)                |
| Connector              | SHOU HAN TF PUSH, push-push, 8 contacts + grounded shell |
| Connector LCSC P/N     | C393941                                                  |
| Card detection         | Firmware (no hardware detect switch — see below)        |
| EDA tool               | KiCad 10                                                 |

## Supported hosts

| Host                         | Supported         | Notes                                                                                                                                     |
| ---------------------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Raspberry Pi Pico W (RP2040) | ✅ Direct         | 3.3 V SPI host                                                                                                                            |
| nRF52840                     | ✅ Direct         | 3.3 V SPI host                                                                                                                            |
| Arduino Uno (ATmega328P)     | ⚠️ Not directly | 5 V logic — its SPI outputs would overdrive a 3.3 V card. Requires an external level shifter on SCK/MOSI/CS. Not included on this board. |

microSD cards operate at 2.7–3.6 V and are not 5 V tolerant. Any 5 V-logic host requires level shifting on the host-driven lines (SCK, MOSI, CS). MISO returns 3.3 V, which a 5 V AVR reads as a valid logic high, so only the outbound lines need shifting.

## Pinout

### microSD contact → SPI mapping

The microSD pinout is standardized. In SPI mode:

| Contact | Card name | SPI role       | On-board support          |
| ------- | --------- | -------------- | ------------------------- |
| 1       | DAT2      | unused         | 10 kΩ pull-up            |
| 2       | CD/DAT3   | **CS**   | 10 kΩ pull-up            |
| 3       | CMD       | **MOSI** | 10 kΩ pull-up            |
| 4       | VDD       | power          | 100 nF + 1 µF decoupling |
| 5       | CLK       | **SCK**  | —                        |
| 6       | VSS       | ground         | —                        |
| 7       | DAT0      | **MISO** | 10 kΩ pull-up            |
| 8       | DAT1      | unused         | 10 kΩ pull-up            |
| shell   | shield    | —             | tied to GND               |

DAT1 and DAT2 are unused in SPI mode but are pulled high so the card does not float into an undefined state at power-up.

### J2 header (1×8, 2.54 mm)

| Pin | Signal |
| --- | ------ |
| 1   | 3V3    |
| 2   | GND    |
| 3   | CS     |
| 4   | SCK    |
| 5   | MOSI   |
| 6   | MISO   |
| 7   | GND    |
| 8   | GND    |

The header order was chosen to keep board routing clean; the two extra grounds improve the jumper-wire return path.

## Card detection

The chosen connector has **no mechanical card-detect switch** — only the 8 SD contacts and a grounded shell. Card presence is therefore detected in  **firmware** : attempt to initialize the card over SPI, and if it responds, a card is present. This is standard practice for SD breakouts and removes a dependency on an underdocumented mechanical feature.

## Bill of Materials

| Ref | Value             | Package     | Notes                            |
| --- | ----------------- | ----------- | -------------------------------- |
| J1  | microSD connector | SMD         | SHOU HAN TF PUSH, LCSC C393941   |
| J2  | 1×8 header       | 2.54 mm THT | Hand-soldered after SMT assembly |
| C1  | 100 nF            | 0603        | Decoupling at VDD                |
| C2  | 1 µF             | 0603        | Bulk decoupling at VDD           |
| R1  | 10 kΩ            | 0603        | CS pull-up                       |
| R2  | 10 kΩ            | 0603        | MOSI pull-up                     |
| R3  | 10 kΩ            | 0603        | MISO pull-up                     |
| R4  | 10 kΩ            | 0603        | DAT1 pull-up                     |
| R5  | 10 kΩ            | 0603        | DAT2 pull-up                     |

## Design decisions & rationale

* **3.3 V-only, passive breakout.** Keeps the board simple and lets it work with any 3.3 V SPI host. Level shifting is treated as a host-side concern, not baked in.
* **SPI 1-bit mode.** The common denominator across target hosts; also simplifies the breakout to CS/SCK/MOSI/MISO + power.
* **Pull-ups on all data lines.** Per the SD physical-layer specification, including the unused DAT1/DAT2 so they don't float; CS is pulled high so it idles correctly.
* **Decoupling placed at the connector's VDD pin.** SD cards have real current inrush; the 100 nF + 1 µF sit immediately at the power pin where they're effective.
* **Connector selected for availability.** An active, in-stock, JLCPCB-assemblable part was prioritized over a "nicer" datasheet on an out-of-stock alternative — a manufacturability-driven choice.
* **Firmware card detection.** Chosen over relying on a hardware switch the connector doesn't have.
* **Dual-layer GND pour, via-stitched.** Grounds the connector shield, provides a low-impedance return, and ties both planes together.

## Manufacturing notes

Intended fab/assembly flow (JLCPCB):

1. Order bare PCBs + **economic SMT assembly** for the connector and 0603 passives.
2. Leave the **through-hole header (J2) off** the assembly and hand-solder it afterward — it's the easiest part and avoids THT assembly fees.
3. At the assembly stage, verify the connector's **pad-1 orientation** in the placement preview — connector rotation is the most common first-time assembly mistake.

The pull-up resistors and caps are all 0603 and the only THT part is the header, so the board is straightforward to assemble either by machine or by hand.

## Not included / limitations

* No level shifting — 3.3 V hosts only.
* No ESD protection — see roadmap.
* No mounting holes are electrically connected (the connector's holes are mechanical/NPTH).
* Untested; no firmware/driver provided. Use a standard SPI SD library on the host (e.g. Arduino `SD`/`SdFat`, CircuitPython `adafruit_sdcard`, or the RP2040/nRF SDKs).

## Roadmap (v2 ideas)

* Add a low-capacitance ESD/TVS array at the connector contacts (e.g. `USBLC6-2SC6` or a multi-channel low-cap array), placed before the pull-ups. Low capacitance matters so SPI edges aren't distorted.
* Optional level-shifted variant (e.g. `74LVC125` buffer + 3.3 V LDO) to support 5 V hosts like the Arduino Uno directly.
* Silkscreen the full header pinout next to J2 for at-a-glance jumpering.

## Repository structure

```
.
├── microSD_breakout.kicad_pro     # KiCad project
├── microSD_breakout.kicad_sch     # Schematic
├── microSD_breakout.kicad_pcb     # PCB layout
├── easyeda2kicad.pretty/          # Imported connector footprint
├── easyeda2kicad.kicad_sym        # Imported connector symbol
├── fab/                           # Gerbers, BOM, CPL (when generated)
├── docs/                          # Schematic / PCB / 3D renders
└── README.md
```

## Opening this project

The connector's symbol and footprint are **bundled in this repository** (`easyeda2kicad.pretty/`, `easyeda2kicad.kicad_sym`) so the project opens without external dependencies. The library tables reference them with the project-relative path variable `${KIPRJMOD}`, so the paths resolve correctly on any machine after cloning.

If KiCad reports a missing footprint or symbol after cloning, check that the footprint and symbol library tables point to the in-repo copies (Preferences → Manage Footprint/Symbol Libraries) using `${KIPRJMOD}/easyeda2kicad.pretty` and `${KIPRJMOD}/easyeda2kicad.kicad_sym`.

### Regenerating the connector footprint

The J1 footprint was imported from LCSC using [easyeda2kicad](https://github.com/uPesy/easyeda2kicad.py). To regenerate it from scratch:

```bash
pip install easyeda2kicad
easyeda2kicad --full --lcsc_id C393941
```

Then add the generated `easyeda2kicad.pretty` and `easyeda2kicad.kicad_sym` to your KiCad library tables. Using the EasyEDA/LCSC library this way ensures the footprint geometry matches JLCPCB's pick-and-place data for assembly.

## Wiring example

Connect the J2 header to your host's SPI pins. The exact host GPIO numbers depend on your board — fill in your host's SPI bus:

| J2 pin | Signal | → Host                      |
| ------ | ------ | ---------------------------- |
| 1      | 3V3    | 3.3 V                        |
| 2      | GND    | GND                          |
| 3      | CS     | any GPIO used as chip-select |
| 4      | SCK    | SPI clock                    |
| 5      | MOSI   | SPI MOSI / TX                |
| 6      | MISO   | SPI MISO / RX                |
| 7      | GND    | GND                          |
| 8      | GND    | GND                          |

## License

[MIT](https://claude.ai/chat/LICENSE) — choose and add a `LICENSE` file. MIT is a common default for open hardware projects; adapt as you prefer.

## Author

**isaac-m** ([@isaacmatt](https://github.com/isaacmatt))

---

*Built in KiCad as a from-scratch PCB design exercise: connector selection and lifecycle/stock checking, third-party footprint import and verification, schematic capture, layout, ground-plane routing, and manufacturing-file preparation.*
