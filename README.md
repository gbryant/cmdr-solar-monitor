# cmdr-solar-monitor

A [commander](https://github.com/gbryant/commander) consumer: an **ESP32-S3**
(N16R8) that measures a solar panel with an **INA219** current/power monitor over
I²C and serves the readings over WiFi/telnet. A host script polls it on an interval
and logs to SQLite; a second script graphs the result.

This is the **ESP-IDF / CMake** flavor of a commander consumer (the ESP32 builds
against `runners/esp32` via FetchContent).

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

Prereqs: ESP-IDF v5 and the `cmdr` tool — commander's
[getting-started guide](https://github.com/gbryant/commander/blob/main/docs/getting-started.md)
covers installing both.

```bash
cp secrets.h.example secrets.h     # then fill in your WiFi
cmdr regen                         # re-emit the dev scripts (bum/build/upload/monitor are gitignored)
./bum                              # build + flash + monitor (the scripts self-source ESP-IDF)
```

First build runs `idf.py set-target esp32s3` automatically.

## Logging & graphing

The device serves readings over telnet (`cmdr-solar-monitor.local:23`). The host
scripts (graphing needs `pip install matplotlib`):

```bash
python3 scripts/poll_solar.py            # poll every 60 s → solar.db (Ctrl-C to stop)
python3 scripts/poll_solar.py --interval 30 --host 192.168.1.50
python3 scripts/graph_solar.py           # plot all data
python3 scripts/graph_solar.py --hours 24
```

`poll_solar.py` reads `ina <ch> stats` (atomic voltage+current) and writes to the
gitignored `solar.db`; it re-inits the INA219 if readings stick at 0 mA.

![Two weeks of solar panel data in three stacked plots — voltage, current and
power — showing a clear daily cycle, with one visibly overcast day.](docs/img/solar-output.png)

Two weeks of real output from `graph_solar.py`. The panel's daily cycle is obvious,
and so is the weather: 05/26 was overcast, 05/31 the best day recorded.

## Updating the commander framework

This project pins commander to a release tag (the `GIT_TAG` in `CMakeLists.txt`).
`cmdr pull` re-fetches that same tag, so on its own it changes nothing — to adopt a
newer release, **`cmdr pin <tag>` then `cmdr pull`** (rebuild after). `cmdr unpin`
floats on `main` instead. Don't depend on a local commander checkout as a normal
workflow.
