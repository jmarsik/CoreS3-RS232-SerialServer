# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ESPHome firmware for an M5Stack CoreS3 (ESP32-S3) with RS232 Module 13.2 that acts as a serial-to-TCP bridge. Clients connect via TCP to port 6638 for transparent bidirectional RS232 data relay.

## Build Commands

All commands use [pixi](https://pixi.sh) as the package manager (ESPHome 2025.12.7 via PyPI):

```bash
pixi install        # Install dependencies (first time only)
pixi run compile   # Compile firmware
pixi run run       # Compile + upload + show logs (USB connected)
pixi run upload    # Upload pre-compiled firmware
pixi run logs      # Stream device logs
pixi run clean     # Clean build artifacts
```

After any YAML change, run `pixi run compile` to verify correctness.

## Architecture

**Main config:** `cores3-serialserver.yaml` — contains all device-specific configuration.

**Reusable packages** in `packages/common/` are included via `!include`:
- `wifi.yaml` — WiFi info sensors (IP, SSID, MAC, signal strength, scan results)
- `uptime.yaml` — uptime sensor with human-readable text conversion
- `debug.yaml` — heap free, loop time, internal temperature, ESPHome version

**External components** (fetched from GitHub at compile time):
- `oxan/esphome-stream-server` — TCP-to-UART bridge (stream_server + sensors)
- `m5stack/M5CoreS3-Esphome` — CoreS3 board init component (`board_m5cores3` handles AXP2101 power management)

## Integrations

- **web_server** (port 80) — device dashboard with status, baud rate selector, and OTA update
- **api** — Home Assistant native API (reboot timeout disabled)
- **ota** — over-the-air firmware updates via ESPHome
- **captive_portal** — WiFi provisioning AP on first boot / when no network is saved
- **improv_serial** — WiFi provisioning via USB serial (used by ESPHome web dashboard)
- **psram** — octal mode at 80MHz for additional memory

## Key Design Decisions

- **Logger uses default serial** — on ESP32-S3, the logger defaults to USB CDC serial, which does not conflict with the RS232 UART on GPIO17/18. No need to set `baud_rate: 0`.
- **No `secrets.yaml`** — WiFi is configured at runtime via captive portal, not hardcoded.
- **`name_add_mac_suffix: true`** — allows multiple devices with the same firmware.
- **Runtime baud rate** — uses a `template select` with `restore_value: true` and lambda calling `set_baud_rate()` + `load_settings()` on the UART component.
- **M5 libraries** (`M5GFX`, `M5Unified`) are included as PlatformIO libraries because `board_m5cores3` depends on them, even though display/touchscreen components are not used.
- **`CONFIG_ARDUINO_LOOP_STACK_SIZE: 32768`** — increased from the default 8KB because the combination of web server, stream server, and other components requires more stack space.

## Hardware Pin Mapping

- UART TX: GPIO17, RX: GPIO18 (RS232 Module 13.2 on CoreS3 bus)
