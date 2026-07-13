# TuksBaja Telemetry | PCB Design

Hardware design files for the TuksBaja SAE Baja vehicle telemetry system (iteration 1).

The board takes the vehicle's 12V battery, steps it down to 5V, and hosts an ESP32 dev board that reads four A3144 hall effect wheel-speed sensors and drives a steering-wheel display.

**Firmware lives in a separate repo:** [esp32-code](https://github.com/TuksBaja-Electrical-Subteam/esp32-code)

## System Overview

```
12V Battery ──> Fuse ──> Reverse-polarity ──> TVS ──> Buck (LMR60430, 5V @ 3A)
                                                          │
                                                          ▼
              4x A3144 Hall Sensors ──> ESP32 DevKit (socketed) ──> Display (steering wheel)
```

Key design decisions:
- **ESP32 dev board socketed on female headers** — not a bare module — so the proven USB/flash/regulator circuitry stays off our board and a damaged dev board can be swapped in seconds.
- **Buck converter design generated with TI WEBENCH** (see `reference/`) and copied verbatim. Do not substitute component values without re-running WEBENCH.
- **SMD assembly by JLCPCB.** We only hand-solder through-hole parts (screw terminals, headers).
- Hall sensors are powered from 5V but pulled up to 3.3V (open-collector output) — safe for ESP32 GPIOs.

## Repo Structure

```
├── TuksBaja Electrical System.kicad_pro   # KiCad project
├── TuksBaja Electrical System.kicad_sch   # Schematic
├── TuksBaja Electrical System.kicad_pcb   # Board layout
├── libs/
│   ├── symbols/            # Project symbol libraries (e.g. LMR60430)
│   └── footprints.pretty/  # Project footprint libraries
└── reference/
    ├── WBDesign3.pdf       # WEBENCH buck converter design report (source of truth)
    └── datasheets/         # LMR60430, A3144, dev board pinout, etc.
```

## Requirements

- **KiCad 10.0 or later** — [kicad.org/download](https://www.kicad.org/download/)
- Libraries in `libs/` are registered as **project-specific** libraries using `${KIPRJMOD}` paths, so they work automatically after cloning. If a symbol/footprint shows as missing, check Preferences → Manage Symbol/Footprint Libraries → Project Specific Libraries.
- For fabrication exports: install the **Fabrication Toolkit** plugin (Tools → Plugin and Content Manager) — generates Gerbers, BOM, and CPL in JLCPCB format.

## Team Workflow (read before editing!)

KiCad files are text, but they **do not merge**. Two people editing the same file will create conflicts that are practically impossible to resolve.

1. **One person edits the schematic/PCB at a time.** Announce in the group chat before you start.
2. `git pull` **before** opening KiCad.
3. Commit and push **as soon as** you close KiCad.
4. Write meaningful commit messages ("add hall sensor input filters", not "update").
5. Never commit: autosaves, `*.lck` lock files, `*-backups/` folders, `.kicad_prl`. The `.gitignore` handles this — don't fight it.

## Fabrication (JLCPCB)

1. Run ERC (schematic) and DRC (layout) — zero errors before ordering.
2. Export with Fabrication Toolkit → Gerbers + `bom.csv` + `positions.csv`.
3. Upload Gerbers at jlcpcb.com → enable **PCB Assembly** → upload BOM + CPL.
4. **Review the 3D placement preview carefully** — check rotation of U1, diodes, and any polarized parts.
5. Order ≥5 boards. Through-hole parts (screw terminals, female headers for the dev board) are hand-soldered after delivery.

## Bring-Up Procedure

Follow in order — do not skip ahead:

1. Visual inspection; continuity check for shorts (12V↔GND, 5V↔GND).
2. Power from a **bench supply, current-limited to ~200 mA**, dev board NOT inserted. Verify 5.0V at the socket VIN pin.
3. Insert dev board (pre-flashed with a blink/hello firmware). Verify boot.
4. Connect one hall sensor; verify pulses. Then the remaining three.
5. Connect display; verify full system on the bench.
6. Only now: connect to the vehicle's 12V.

## Status

- [ ] Schematic — buck converter section
- [ ] Schematic — input protection
- [ ] Schematic — ESP32 socket, sensors, display
- [ ] ERC clean
- [ ] Layout
- [ ] DRC clean
- [ ] Rev A ordered
- [ ] Rev A bring-up