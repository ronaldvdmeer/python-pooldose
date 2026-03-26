# Examples

The `examples/` directory contains demonstration scripts that show how to use the python-pooldose library.

## Real Device Demo (`examples/demo.py`)

Demonstrates connecting to a real PoolDose device and accessing all types of data:

```bash
# Edit the HOST variable in the file first
python examples/demo.py
```

**Features:**

- Connects to actual hardware
- Shows device information and static values
- Displays all sensor readings, alarms, setpoints, and settings
- Demonstrates error handling

## Basic Example

```python
import asyncio
import json
from pooldose.client import PooldoseClient
from pooldose.request_status import RequestStatus

HOST = "192.168.1.100"  # Change this to your device's host or IP address
TIMEOUT = 30

async def main() -> None:
    """Demonstrate PooldoseClient usage with dictionary-based API."""
    
    # Create client instance (excludes WiFi passwords by default)
    client = PooldoseClient(host=HOST, timeout=TIMEOUT)
    
    # Optional: Include sensitive data like WiFi passwords
    # client = PooldoseClient(host=HOST, timeout=TIMEOUT, include_sensitive_data=True)
    
    # Connect to device
    status = await client.connect()
    if status != RequestStatus.SUCCESS:
        print(f"Error connecting to device: {status}")
        return
    
    print(f"Connected to {HOST}")
    print("Device Info:", json.dumps(client.device_info, indent=2))

    # --- Get static values ---
    status, static_values = client.static_values()
    if status == RequestStatus.SUCCESS:
        print(f"Device Name: {static_values.sensor_name}")
        print(f"Serial Number: {static_values.sensor_serial_number}")
        print(f"Firmware Version: {static_values.sensor_fw_version}")

    # --- Get instant values (dictionary-style) ---
    status, instant_values = await client.instant_values()
    if status != RequestStatus.SUCCESS:
        print(f"Error getting instant values: {status}")
        return

    # Dictionary-style individual access
    if "temperature" in instant_values:
        temp = instant_values["temperature"]
        print(f"Temperature: {temp[0]} {temp[1]}")

    # Get with default
    ph_value = instant_values.get("ph", "Not available")
    print(f"pH: {ph_value}")

    # --- Get structured instant values ---
    status, structured_data = await client.instant_values_structured()
    if status != RequestStatus.SUCCESS:
        print(f"Error getting structured values: {status}")
        return

    # Access sensors
    sensors = structured_data.get("sensor", {})
    print("\nSensor Values:")
    for key, sensor_data in sensors.items():
        value = sensor_data.get("value")
        unit = sensor_data.get("unit")
        if unit:
            print(f"  {key}: {value} {unit}")
        else:
            print(f"  {key}: {value}")

    # Access numbers (setpoints)
    numbers = structured_data.get("number", {})
    print("\nSetpoints:")
    for key, number_data in numbers.items():
        value = number_data.get("value")
        unit = number_data.get("unit")
        min_val = number_data.get("min")
        max_val = number_data.get("max")
        
        if unit:
            print(f"  {key}: {value} {unit} (Range: {min_val}-{max_val})")
        else:
            print(f"  {key}: {value} (Range: {min_val}-{max_val})")

    # Access switches
    switches = structured_data.get("switch", {})
    print("\nSwitches:")
    for key, switch_data in switches.items():
        value = switch_data.get("value")
        status_text = "ON" if value else "OFF"
        print(f"  {key}: {status_text}")

    # Access binary sensors (alarms/status)
    binary_sensors = structured_data.get("binary_sensor", {})
    print("\nAlarms & Status:")
    for key, sensor_data in binary_sensors.items():
        value = sensor_data.get("value")
        status_text = "ACTIVE" if value else "OK"
        print(f"  {key}: {status_text}")

    # Access selects (configuration options)
    selects = structured_data.get("select", {})
    print("\nSettings:")
    for key, select_data in selects.items():
        value = select_data.get("value")
        print(f"  {key}: {value}")

    # --- Setting values ---
    
    # Set number values (via InstantValues)
    result = await instant_values.set_number("target_ph", 7.2)
    print(f"Set pH target to 7.2: {result}")

    # Set switch values
    result = await instant_values.set_switch("stop_dosing", True)
    print(f"Set stop dosing: {result}")

    # Set select values
    result = await instant_values.set_select("water_meter_unit", "L/h")
    print(f"Set water meter unit: {result}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Advanced Usage

### Connection Management

```python
from pooldose.client import PooldoseClient
from pooldose.request_status import RequestStatus

# HTTP connection (default)
client = PooldoseClient("192.168.1.100", timeout=30)
status = await client.connect()

# HTTPS connection with SSL verification
client = PooldoseClient("192.168.1.100", timeout=30, use_ssl=True)
status = await client.connect()

# HTTPS connection with custom port and disabled verification
client = PooldoseClient("192.168.1.100", use_ssl=True, port=8443, ssl_verify=False)
status = await client.connect()

# Check connection status
if client.is_connected:
    print("Client is connected")
else:
    print("Client is not connected")
```

### Error Handling

```python
from pooldose.client import PooldoseClient

client = PooldoseClient("192.168.1.100")
status = await client.connect()

if status == RequestStatus.SUCCESS:
    print("Connected successfully")
elif status == RequestStatus.HOST_UNREACHABLE:
    print("Could not reach device")
elif status == RequestStatus.PARAMS_FETCH_FAILED:
    print("Failed to fetch device parameters")
elif status == RequestStatus.API_VERSION_UNSUPPORTED:
    print("Unsupported API version")
else:
    print(f"Other error: {status}")
```

### Working with Structured Data

```python
# Get all data types at once
status, structured_data = await client.instant_values_structured()

if status == RequestStatus.SUCCESS:
    # Check what types are available
    available_types = list(structured_data.keys())
    print("Available types:", available_types)
    
    # Process each type
    for data_type, items in structured_data.items():
        print(f"\n{data_type.title()} ({len(items)} items):")
        for key, data in items.items():
            if data_type in ["sensor", "number"]:
                value = data.get("value")
                unit = data.get("unit")
                if unit:
                    print(f"  {key}: {value} {unit}")
                else:
                    print(f"  {key}: {value}")
            elif data_type in ["switch", "binary_sensor"]:
                value = data.get("value")
                print(f"  {key}: {'ON' if value else 'OFF'}")
            elif data_type == "select":
                value = data.get("value")
                print(f"  {key}: {value}")
```

### Working with Mappings

```
Mapping Discovery Process:
┌─────────────────┐
│ Device Connect  │
└─────────────────┘
         │
         ▼
┌─────────────────┐
│ Get Model ID    │ ──────► PDZZ1H1HATEST1V1
│ Get Firmware Code     │ ──────► 654321
└─────────────────┘
         │
         ▼
┌─────────────────┐
│ Load JSON file  │ ──────► model_PDZZ1H1HATEST1V1_FW654321.json
└─────────────────┘
         │
         ▼
┌─────────────────┐
│ Type Discovery  │
│ ┌─────────────┐ │
│ │ Sensors     │ │ ──────► temperature, ph, orp, ...
│ │ Switches    │ │ ──────► stop_dosing, pump_detection, ...
│ │ Numbers     │ │ ──────► ph_target, orp_target, ...
│ │ Selects     │ │ ──────► water_meter_unit, ...
│ │ Binary Sens │ │ ──────► alarm_ph, alarm_orp, ...
│ └─────────────┘ │
└─────────────────┘
```

## Benefits of the Examples

- **Learning**: Step-by-step progression from simple to advanced usage
- **Development**: Mock client allows development without hardware
- **Testing**: JSON-based testing for CI/CD pipelines
- **Reference**: Real-world code patterns and best practices
