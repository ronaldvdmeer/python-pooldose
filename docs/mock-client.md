# Mock Client System

The **MockPooldoseClient** system allows using JSON files instead of real Pooldose hardware for testing and development. This is particularly useful for:

- **Development without hardware**
- **Unit tests**
- **Data analysis with real device data**
- **CI/CD pipeline tests**

## Quick Start

```python
import asyncio
from pathlib import Path
from pooldose.mock_client import MockPooldoseClient

async def simple_test():
    # Load data file
    json_file = Path("path/to/your/data.json")
    
    # Create mock client
    client = MockPooldoseClient(json_file_path=json_file)
    
    # Connect (loads mapping data)
    status = await client.connect()
    if status.name != "SUCCESS":
        print(f"Connection failed: {status}")
        return
    
    # Get sensor values
    status, instant_values = await client.instant_values()
    if status.name == "SUCCESS" and instant_values:
        print(f"Temperature: {instant_values['temperature']}")
        print(f"pH Value: {instant_values['ph']}")
        print(f"ORP: {instant_values['orp']}")
    
    # Get structured data
    status, data = await client.instant_values_structured()
    if status.name == "SUCCESS":
        sensors = data.get('sensor', {})
        for name, info in sensors.items():
            value = info.get('value', 'N/A')
            unit = info.get('unit', '')
            print(f"{name}: {value} {unit}")

# Run demo
asyncio.run(simple_test())
```

## Command Line Usage

You can use the mock client with custom JSON files via the command line:

```bash
# Use mock client with JSON file
pooldose --mock path/to/your/data.json


# Use mock client with model and firmware code (Beispiel mit Fantasiewerten)
pooldose --mock path/to/your/data.json --model-id PDZZ1H1HATEST1V1 --fw-code 654321

# Or as Python module
python -m pooldose --mock path/to/your/data.json
python -m pooldose --mock path/to/your/data.json --model-id PDZZ1H1HATEST1V1 --fw-code 654321
```

## JSON Data Format

The JSON file must have the following structure:

```json
{
    "devicedata": {
        "SERIALNUMBER_DEVICE": {
            "MODEL_FW_w_key1": {
                "current": 25.5,
                "magnitude": ["°C"]
            },
            "MODEL_FW_w_key2": {
                "current": 7.2,
                "magnitude": ["pH"]
            }
        }
    }
}
```

## API Methods

### Initialization

```python
client = MockPooldoseClient(
    json_file_path="path/to/data.json",
    timeout=30,  # Ignored (compatibility)
    include_sensitive_data=True  # Include WiFi keys etc.
)
```

### Connection

```python
status = await client.connect()  # Loads mapping configuration
is_connected = client.is_connected  # Check status
```

### Data Retrieval

```python
# Static device information
status, static_values = client.static_values()

# Live sensor values
status, instant_values = await client.instant_values()

# Structured data (grouped by types)
status, structured_data = await client.instant_values_structured()
```

### Utility Methods

```python
# Get raw data
raw_data = client.get_raw_data()
device_data = client.get_device_data()

# Reload JSON file
success = client.reload_data()
```

## Available Sample Files

The following sample JSON files are available in the repository:

- `references/testdaten/tscherno/instantvalues.json` - Sample device data for testing

## Use Cases

### Unit Tests

```python
def test_temperature_reading():
    client = MockPooldoseClient("sample_data.json")
    asyncio.run(client.connect())
    
    status, values = asyncio.run(client.instant_values())
    assert status.name == "SUCCESS"
    assert values['temperature'][0] == 23.0  # Expected value
```

### Data Analysis

```python
# Analyze all sensor values
client = MockPooldoseClient("production_data.json")
await client.connect()

status, data = await client.instant_values_structured()
sensors = data.get('sensor', {})

for sensor_name, sensor_data in sensors.items():
    value = sensor_data.get('value')
    unit = sensor_data.get('unit', '')
    print(f"{sensor_name}: {value} {unit}")
```

### Integration Tests

```python
async def test_full_integration():
    client = MockPooldoseClient("integration_sample_data.json")
    
    # Test connection
    assert await client.connect() == RequestStatus.SUCCESS
    
    # Test static values
    status, static = client.static_values()
    assert status == RequestStatus.SUCCESS
    assert static.sensor_name is not None
    
    # Test live values
    status, instant = await client.instant_values()
    assert status == RequestStatus.SUCCESS
    assert 'temperature' in instant
```

## Benefits

- **Fast**: No network latency
- **Reliable**: No hardware dependencies  
- **Flexible**: Different scenarios testable
- **Realistic**: Real device data structures
- **Compatible**: Same API as real client
