# Type Hints & Home Assistant Integration

This package is **PEP-561 compliant** and fully typed for use in Home Assistant integrations.

## Type Safety Features

**PEP-561 Compliance**: Package includes `py.typed` file marking it as fully typed  
**Comprehensive Type Annotations**: All public API methods have complete type hints  
**Type Constants**: Centralized `VALUE_TYPE_*` constants eliminate string literals for entity types  
**mypy Support**: Built-in mypy configuration for static type checking  
**Home Assistant Ready**: Compatible with Home Assistant's strict typing requirements  

## Type-Safe Usage

```python
from pooldose import PooldoseClient
from pooldose.request_status import RequestStatus

# Type checkers will infer all types automatically
client: PooldoseClient = PooldoseClient("192.168.1.100")
status: RequestStatus = await client.connect()

# Dictionary-style access with proper typing
status, instant_values = await client.instant_values()
if status == RequestStatus.SUCCESS and instant_values:
    temperature = instant_values["temperature"]  # Typed as tuple[float, str]
    ph_value = instant_values.get("ph", "N/A")  # Safe access with default

# Structured data with full type safety
status, structured_data = await client.instant_values_structured()
sensors = structured_data.get("sensor", {})  # Type: dict[str, dict[str, Any]]
```

## Integration Benefits

- **IDE Support**: Full autocomplete and type checking in VS Code, PyCharm, etc.
- **Runtime Safety**: Catch type errors before deployment
- **Documentation**: Self-documenting code through type annotations
- **Maintenance**: Easier refactoring with type-guided development

For Home Assistant integrations, add this package to your integration's dependencies and enjoy full type safety throughout your integration code.
