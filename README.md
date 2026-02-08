# CoreS3 RS232 Serial Server

ESPHome-based serial-to-TCP bridge running on [M5Stack CoreS3](https://docs.m5stack.com/en/core/CoreS3) with [RS232 Module 13.2](https://docs.m5stack.com/en/module/RS232M%20Module%2013.2).

Exposes an RS232 serial port as a TCP socket using [esphome-stream-server](https://github.com/oxan/esphome-stream-server). Clients connect to the device's IP on port 6638 and get transparent bidirectional serial data relay.

## Hardware

- **M5Stack CoreS3** (ESP32-S3, 240MHz, 16MB flash, PSRAM)
- **RS232 Module 13.2** stacked on the CoreS3 bus
  - TX: GPIO17
  - RX: GPIO18
  - Baud rate: 115200 (default, changeable at runtime)

## Prerequisites

- [pixi](https://pixi.sh) package manager

## Building and Flashing

```bash
# Install dependencies
pixi install

# Compile firmware
pixi run compile

# Compile, upload and show logs (device connected via USB)
pixi run run

# Upload pre-compiled firmware
pixi run upload

# View device logs
pixi run logs

# Clean build artifacts
pixi run clean
```

## WiFi Setup

On first boot (or when no saved WiFi network is available), the device starts a captive portal AP. Connect to it with your phone or laptop, then configure your WiFi credentials through the portal. The credentials are saved and used on subsequent boots.

## Connecting to the Serial Port

Once the device connects to WiFi, connect any TCP client to `<device-ip>:6638`. Data sent over TCP is forwarded to the RS232 port and vice versa.

Some settings related to processing of the data (like echo, line endings, special characters, clipboard handling, etc.) depend on the device connected to the serial port of the RS232 port and the client software used to connect to the TCP port. The serial server itself is a transparent bridge and does not modify the data in any way.

Below are example connection instructions for common TCP client software and usual connected devices (like serial consoles of network equipment, microcontrollers, etc.):

### socat

```bash
# escape allows terminating the session with Ctrl+D
socat -,rawer,escape=0x04 TCP:<device-ip>:6638
```

### PuTTY / KiTTY

1. Set **Connection type** to **Raw**
2. Enter the device IP as **Host Name**
3. Set **Port** to **6638**
4. Go to **Terminal** category and change **Local echo** and **Local line editing** to **Force off**
5. Click **Open**

## Web Server

The device runs a web server on port 80 (`http://<device-ip>/`). It provides:

- Device status overview
- **Baud rate** selector — change the RS232 baud rate at runtime (2400, 4800, 9600, 19200, 38400, 57600, 115200). The selected value is saved and persists across reboots.
- **Client connected** — whether a TCP client is currently connected to the serial server
- **Number of connections** — count of active TCP connections
- WiFi signal strength, IP address, SSID, and MAC address
- Device uptime
- Debug info (heap free, loop time, internal temperature)
- OTA firmware update capability

## Configuration

Main config: `cores3-serialserver.yaml`

Shared packages in `packages/common/`: `wifi.yaml`, `uptime.yaml`, `debug.yaml`
