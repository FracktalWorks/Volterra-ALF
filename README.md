# Volterra-ALF

**Volterra Adaptive Layer Fusion** - Klipper Configuration for Large Format CoreXY 3D Printer

[![Klipper](https://img.shields.io/badge/Firmware-Klipper-red)](https://www.klipper3d.org/)
[![Mainsail](https://img.shields.io/badge/Interface-Mainsail-blue)](https://docs.mainsail.xyz/)
[![License](https://img.shields.io/badge/License-GPLv3-green.svg)](https://www.gnu.org/licenses/gpl-3.0)

## Overview

This repository contains the Klipper firmware configuration files for the **Volterra-ALF** (Adaptive Layer Fusion) 3D printer by [Fracktal Works](https://github.com/FracktalWorks). The Volterra-ALF is a large-format industrial CoreXY printer designed for high-temperature materials with advanced thermal management features.

## Printer Specifications

| Specification | Value |
|---------------|-------|
| **Kinematics** | CoreXY |
| **Build Volume** | 400 × 400 × 435 mm |
| **Max Velocity** | 200 mm/s |
| **Max Acceleration** | 1500 mm/s² |
| **Max Z Velocity** | 20 mm/s |
| **Nozzle Diameter** | 0.4 mm |
| **Max Hotend Temp** | 480°C |
| **Max Bed Temp** | 150°C |
| **Max Chamber Temp** | 160°C |

## Hardware Configuration

### Motion System
- **Stepper Drivers**: TMC5160 on all axes (SPI mode with StealthChop)
- **X/Y Motors**: 1.2A run current, 16 microsteps
- **Z Motor**: 2.8A run current, 8 microsteps
- **Extruder**: Dual extrusion system with primary and side extruder steppers

### Control Board
- **MCU**: STM32H723xx (BTT Manta M8P or similar)
- **Serial**: USB connection via `/dev/serial/by-id/`

### Thermal System
- **Hotend**: PT1000 sensor, PID controlled (up to 480°C)
- **Heated Bed**: ATC Semitec 104GT-2 thermistor, watermark control
- **Chamber Heater**: PID controlled with dedicated heater
- **Filament Heater**: Separate heating zone for filament preheating
- **AOF Fan**: Always-on fan for hotend cooling

### Sensors & Probing
- **Z Probe**: Bed-mounted probe with automatic mesh leveling
- **Bed Mesh**: 5×5 probe grid with bicubic interpolation
- **Bed Screws**: 3-point manual leveling (Front Left, Front Right, Rear Centre)

## File Structure

```
Volterra-ALF/
├── printer.cfg              # Main printer configuration
├── CORE_GCODE_MACROS.cfg    # Core G-code macros for Marlin compatibility
├── macro.cfg                # Additional custom macros
├── mainsail.cfg             # Mainsail web interface configuration
├── moonraker.conf           # Moonraker API server configuration
├── KlipperScreen.conf       # KlipperScreen touch interface settings
└── config_backups/          # Timestamped configuration backups
```

## Key Features

### G-code Compatibility
The configuration includes Marlin-compatible G-code macros for seamless operation:

| Macro | Description |
|-------|-------------|
| `G28` | Custom homing override (Z→Y→X sequence) |
| `G29` | Automatic bed mesh calibration |
| `M106` | Part cooling fan control (inverted PWM) |
| `M114` | Position reporting |
| `M190` | Heated bed temperature wait with tolerance |
| `M218` | Tool offset configuration |
| `M290` | Babystepping with persistent storage |
| `M420` | Bed mesh load/clear |
| `M500` | Save configuration |
| `M503` | Report settings |
| `M851` | Z probe offset adjustment |
| `M900` | Pressure advance configuration |

### Filament Sensors
- **Runout Detection**: Switch-based filament sensors (T0/T1)
- **Jam Detection**: Motion-based encoder sensors
- **Macros**: `SET_FILAMENT_RUNOUT_SENSOR`, `SET_FILAMENT_JAM_SENSOR`

### Thermal Management
- Multi-zone heating (bed, chamber, filament)
- Automatic idle timeout with temperature management
- Configurable verify_heater settings for reliable operation

## Installation

### Prerequisites
- Raspberry Pi (or similar SBC) running Raspberry Pi OS
- Klipper firmware installed
- Moonraker API server
- Mainsail or Fluidd web interface

### Setup Steps

1. **Clone the repository**:
   ```bash
   cd ~/printer_data/config
   git clone https://github.com/FracktalWorks/Volterra-ALF.git .
   ```

2. **Flash Klipper to MCU**:
   ```bash
   cd ~/klipper
   make menuconfig
   # Select: STM32H723, 128KiB bootloader, USB communication
   make
   make flash FLASH_DEVICE=/dev/serial/by-id/usb-Klipper_stm32h723xx_XXXXX
   ```

3. **Configure Moonraker**:
   - Update `moonraker.conf` with your network settings
   - Add trusted clients for your network

4. **Restart Services**:
   ```bash
   sudo systemctl restart klipper
   sudo systemctl restart moonraker
   ```

## Usage

### Initial Setup
1. **Home the printer**: `G28`
2. **Level the bed screws**: `SCREW_ADJUST`
3. **Run bed mesh calibration**: `G29`
4. **Adjust Z probe offset**: `M851 Z=<offset>`
5. **Save configuration**: `M500`

### Print Preparation
```gcode
; Preheat
M140 S60          ; Bed temperature
M104 S200         ; Hotend temperature
SET_HEATER_TEMPERATURE HEATER=Chamber TARGET=50  ; Chamber

; Home and level
G28               ; Home all axes
M420 S1           ; Load bed mesh

; Start printing
```

### Babystepping During Print
```gcode
M290 Z0.05        ; Raise nozzle by 0.05mm
M290 Z-0.05       ; Lower nozzle by 0.05mm
```

## Web Interface

Access the printer through **Mainsail** at:
- Local: `http://<raspberry-pi-ip>`
- Default port: 80 (web) / 7125 (API)

### Supported Interfaces
- [Mainsail](https://docs.mainsail.xyz/) - Primary web interface
- [KlipperScreen](https://klipperscreen.readthedocs.io/) - Touch screen interface
- [OctoPrint](https://octoprint.org/) - Compatible via `octoprint_compat`

## Configuration Backups

Configuration backups are automatically saved in `config_backups/` with timestamps:
- Format: `printer-YYYYMMDD_HHMMSS.cfg`
- Useful for reverting to previous working configurations

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Homing fails | Check endstop connections, verify `position_endstop` values |
| Probe not triggering | Verify probe pin configuration and wiring |
| Temperature errors | Adjust `verify_heater` settings, check thermistor connections |
| Stepper motor issues | Check TMC5160 SPI connections and `run_current` values |

### Useful Commands
```gcode
QUERY_ENDSTOPS           ; Check endstop states
QUERY_PROBE              ; Check probe state
QUERY_FILAMENT_RUNOUT_SENSOR  ; Check filament sensor status
M503                     ; Report current settings
```

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request with detailed description

## License

This project is licensed under the **GNU General Public License v3.0** - see the included mainsail.cfg header for details.

## Authors

- **Vijay Raghav Varada** - Fracktal Works
- **Yashas B Padmashali** - Fracktal Works

---

*For more information about Fracktal Works printers, visit [fracktal.in](https://fracktal.in)*
![alt text](<BIGTREETECH MANTA M8P V2.0 PinOut (1).png>)