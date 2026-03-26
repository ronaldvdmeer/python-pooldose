# Command Line Usage

After installation, you can use python-pooldose directly from the command line.

> **Note:** `--host` and `--mock` are mutually exclusive — you must specify exactly one.
> `--analyze` and `--analyze-all` require `--host` (they cannot be used in mock mode).

## Connect to Real Device

```bash
# Basic connection
pooldose --host 192.168.1.100

# With HTTPS
pooldose --host 192.168.1.100 --ssl

# Custom port
pooldose --host 192.168.1.100 --ssl --port 8443

# Analyze device capabilities (discover unsupported devices)
pooldose --host 192.168.1.100 --analyze

# Show all widgets including hidden ones
pooldose --host 192.168.1.100 --analyze-all

# Show version
pooldose --version
```

## Mock Mode with JSON Files

```bash
# Use JSON file for testing
pooldose --mock path/to/your/data.json
```

## Alternative Module Execution

You can also run it as a Python module:

```bash
# Real device
python -m pooldose --host 192.168.1.100

# Device analysis
python -m pooldose --host 192.168.1.100 --analyze

# Mock mode
python -m pooldose --mock data.json

# Show help
python -m pooldose --help

# Show version
python -m pooldose --version
```

## Device Analysis for Unsupported Devices

The device analyzer is a powerful feature that helps discover and analyze PoolDose devices that are not yet officially supported. This is particularly useful for:

- **New Device Discovery**: Identifying capabilities of unknown device models
- **Device Support Development**: Gathering data needed to add support for new devices
- **Troubleshooting**: Understanding how your device exposes data and controls
- **Widget Exploration**: Discovering all available sensors, controls, and settings

### Basic Device Analysis

```bash
# Analyze a device to discover its capabilities
pooldose --host 192.168.1.100 --analyze

# Show all widgets including hidden ones
pooldose --host 192.168.1.100 --analyze-all

# Analyze with HTTPS
pooldose --host 192.168.1.100 --ssl --analyze
```

### Analysis Output

The analyzer provides comprehensive information about your device:

```
=== DEVICE ANALYSIS ===
Device: 01234567890A_DEVICE
Model ID: PDZZ1H1HATEST1V1  
Firmware Code: 654321

=== WIDGETS (Visible UI Elements) ===

SENSORS (Read-only values)
temperature: 24.5°C
ph: 7.2
orp: 720 mV

SETPOINTS (Configurable values)  
target_ph: 7.0 (Range: 6.0-8.0, Step: 0.1)
target_orp: 700 mV (Range: 400-900, Step: 10)

SWITCHES (On/Off controls)
stop_dosing: OFF
pump_detection: ON

SELECTS (Configuration options)
water_meter_unit: L/h
  Options: [L/h, m³/h, gal/h]

ALARMS (Status indicators)
alarm_ph: OK
alarm_orp: OK
```

### Using Analysis for Device Support

When you encounter an unsupported device, the analyzer helps gather the necessary information:

1. **Run Analysis**: Use `--analyze` to discover all device capabilities
2. **Document Output**: Save the analysis output to understand device structure  
3. **Check Widget Types**: Note which sensors, controls, and settings are available
4. **Identify Patterns**: Look for device model and firmware information
5. **Report Findings**: Use the analysis data to request support for your device model

### Example: Discovering New Device

```bash
# Unknown device analysis
pooldose --host 192.168.1.100 --analyze

# Output shows:
# Device: 01987654321B_DEVICE  
# Model ID: PDZZ1H1HATEST1V1     ← New model not yet supported
# Firmware Code: 654321            ← New firmware version
# 
# Widgets discovered: 15 sensors, 8 controls, 12 settings
```

With this information, you can:
- Report the new model/firmware combination  
- Share the widget structure for mapping development
- Help expand device support for the community

The device analyzer makes python-pooldose extensible and helps build support for the growing ecosystem of SEKO PoolDose or VÁGNER POOL devices.
