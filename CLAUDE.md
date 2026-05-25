# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ Это vendored upstream — не патчим без необходимости

Этот каталог — third-party RotorHazard. Он используется как backend для платформы ATMOS, которая живёт в `../platform/`. **ATMOS-специфичная логика идёт в плагин** `../platform/rh-plugin/atmos_sync/`, а не в `src/`.

Прежде чем редактировать что-либо под `src/`:
1. Проверь, можно ли решить задачу через RHAPI в плагине `atmos_sync` (см. `doc/RHAPI.md`, `doc/Plugins.md`).
2. Если кажется что без правки ядра не обойтись — **спроси у пользователя**, прежде чем менять файл. Опиши, что пробовал в плагине и почему не получилось.
3. Каждая правка под `src/` — потенциальный merge-конфликт с апстримом и она не попадает в `atmos_sync.zip`.

Это правило также записано в `../CLAUDE.md`.

## Project Overview

RotorHazard is an open-source FPV drone racing timing and event management system. It uses RSSI (Received Signal Strength Indicator) monitoring from RX5808 video receivers to detect lap crossings and manages full race events. Supports up to 16 simultaneous racers across multiple hardware nodes (STM32 Blue Pill or Arduino).

## Python Virtual Environment

The project uses a venv at `src/server/venv/`. **Always use the venv Python** for running the server, tests, and installing dependencies:
```bash
src/server/venv/bin/python    # Python interpreter
src/server/venv/bin/pip       # pip
```

The system Python has older package versions (e.g. SQLAlchemy 1.4) that are incompatible with the project (requires SQLAlchemy 2.0). Using the system Python will produce misleading errors.

## Build & Test Commands

### Server tests (Python)
```bash
cd src/tests && ../server/venv/bin/python -m unittest discover
```

### Run a single test file
```bash
cd src/tests && ../server/venv/bin/python -m unittest test_server
cd src/tests && ../server/venv/bin/python -m unittest test_Sensors
```

### Install server dependencies
```bash
src/server/venv/bin/pip install -r src/server/requirements.txt
```

### Node firmware tests (C++/Arduino CI)
```bash
cd src/node && bundle install && bundle exec arduino_ci.rb
```

### Run the server
```bash
cd src/server && venv/bin/python server.py
```

## Architecture

### Three-tier system

1. **Embedded Firmware** (`src/node/`) — C++/Arduino running on STM32/Arduino microcontrollers. Handles low-level RSSI signal acquisition, filtering, peak/nadir tracking, and lap crossing detection. Communicates with the server via I2C or serial (USB).

2. **Python Server** (`src/server/`) — Flask + Socket.IO application. Core modules:
   - `server.py` — Main entry point, Flask app, WebSocket endpoints, HTTP routes
   - `RHRace.py` — Race state machine, lap timing logic, race lifecycle
   - `RHData.py` — Data abstraction layer with caching over the SQLAlchemy models
   - `Database.py` — SQLAlchemy ORM models (pilots, heats, laps, race classes, formats)
   - `Results.py` — Leaderboard calculations and rankings
   - `RHAPI.py` — Plugin API surface (documented in `doc/RHAPI.md`)
   - `RHUI.py` — Dynamic UI panel registration system
   - `Config.py` — Configuration persistence
   - `eventmanager.py` — Pub/sub event system (`Evt` events, `Flt` filters)
   - `ClusterNodeSet.py` — Multi-node/processor clustering coordination

3. **Web Frontend** (`src/server/static/`) — HTML templates + JavaScript with Socket.IO for real-time race updates.

### Hardware Interface Layer (`src/interface/`)
- `RHInterface.py` — Base communication class for I2C/serial with node devices
- `serial_node.py` / `i2c_node.py` — Protocol-specific implementations
- `Node.py` — Single node abstraction
- `MockInterface.py` — Mock for testing without hardware
- `*_sensor.py` — Sensor drivers (BME280, INA219, etc.)

### Plugin System
Plugins extend RotorHazard via the event hook system. Two locations:
- **Bundled plugins**: `src/server/bundled_plugins/` — shipped with the project (ranking algorithms, data export, LED handlers, heat generators, scoring)
- **User plugins**: loaded from a configurable data directory (default `~/rh-data/plugins/`)

Each plugin has `__init__.py` and `manifest.json`. Plugin API is documented in `doc/RHAPI.md` and `doc/Plugins.md`.

### Communication Protocols
- **Server <-> Nodes**: I2C (addressed via `6 + NODE_NUMBER*2`) or serial USB (921600/115200 baud). Command bytes 0x00-0x7F with checksum validation.
- **Frontend <-> Server**: HTTP REST + Socket.IO WebSockets for real-time updates.

### Data Storage
- SQLite database (`database.db`) via SQLAlchemy ORM
- Runtime config in `config.json` (gitignored)
- Results are cached and invalidated on data changes

## User Data Directory (`~/rh-data/`)

The server stores runtime data outside the repo in `~/rh-data/`:
- `config.json` — server runtime configuration
- `database.db` — race database
- `logs/` — server log files (`rh_YYYYMMDD_HHMMSS.log`)
- `plugins/` — user-developed plugins (not in this repo)
- `db_bkp/`, `cfg_bkp/` — automatic backups

### Existing user plugins (reference for developing new plugins)
The `~/rh-data/plugins/` directory contains several developed plugins that serve as practical examples:
- `whooptrack_flags` — custom flag/status displays
- `whooptrack_gates` — gate management
- `whooptrack_generators` — custom heat generators
- `whooptrack_reports` — report generation with templates and custom pages
- `whooptrack_obs` — OBS integration
- `wt_overlays` — stream overlay pages
- `wt_screens` — custom screen/UI pages
- `rh_lap_annotations` — lap annotation system

These plugins demonstrate patterns for UI panels, custom pages, static assets, templates, and RHAPI usage.

## Key API Versions
- `SERVER_API`: 52
- `NODE_API_BEST`: 36

## Language & Localization
- Multi-language support via `src/server/language.json` (256+ KB of translation strings)
- Interface language configurable at runtime

## CI
- Travis CI with Python 3.9, codecov integration
- Tests: Python unittest discovery + Arduino CI for firmware
