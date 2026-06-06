# cmdr-solar-monitor

A [commander](https://github.com/gbryant/commander) consumer: an **ESP32-S3**
(N16R8) that measures a solar panel with an **INA219** current/power monitor over
I²C and serves the readings over WiFi/telnet. A host script polls it on an interval
and logs to SQLite; a second script graphs the result.

This is the **ESP-IDF / CMake** flavor of a commander consumer (the ESP32 builds
against `runners/esp32` via FetchContent). Naming: a `cmdr-` prefix marks commander
consumers so they don't need an umbrella folder.

## Hardware

- ESP32-S3-N16R8 (16 MB flash, 8 MB OPI PSRAM), native USB Serial/JTAG console.
- INA219 on I²C **SDA=GPIO8 / SCL=GPIO9**, address **0x40** (channel `a`).
  Calibrated for a 0.1 Ω shunt. A second channel can be added later with
  `cmdr module enable ina219` (e.g. `a:0x40,b:0x45`).

## Modules

`cmdr module list` shows what's enabled: `system` (help/version), `i2c` (bus
diagnostics — `i2c scan` to confirm the INA219 ACKs at 0x40), and `ina219`
(`ina` to list, `ina a volt|amp|watt|stats|init`, `ina stats` for CSV).

## Setup

```bash
cp secrets.h.example secrets.h     # then fill in your WiFi
esp                                # load the ESP-IDF environment (once per shell)
./bum                              # build + flash + monitor (build/upload/monitor are gitignored)
```

First build runs `idf.py set-target esp32s3` automatically.

## Logging & graphing

The device serves readings over telnet (`cmdr-solar-monitor.local:23`). The host
scripts (need `pip install pyserial matplotlib`):

```bash
python3 scripts/poll_solar.py            # poll every 60 s → solar.db (Ctrl-C to stop)
python3 scripts/poll_solar.py --interval 30 --host 192.168.1.50
python3 scripts/graph_solar.py           # plot all data
python3 scripts/graph_solar.py --hours 24
```

`poll_solar.py` reads `ina <ch> stats` (atomic voltage+current) and writes to the
gitignored `solar.db`; it re-inits the INA219 if readings stick at 0 mA.

## Updating the commander framework

Framework changes live in the commander repo; adopt the latest with **`cmdr pull`**
(rebuild after). The commander version is the `GIT_TAG` in `CMakeLists.txt`
(`main` = latest default branch; a release tag like `v1.2.0` pins it). Don't depend
on a local commander checkout as a normal workflow.
