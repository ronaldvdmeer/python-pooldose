# Security

By default, the client excludes sensitive information like WiFi passwords from device info. To include sensitive data:

```python
client = PooldoseClient(
    host="192.168.1.100", 
    include_sensitive_data=True
)
status = await client.connect()
```

## Security Model

```text
Data Classification:
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Public Data   │    │ Sensitive Data  │    │  Never Exposed  │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ • Device Name   │    │ • WiFi Password │    │ • Admin Creds   │
│ • Model ID      │    │ • AP Password   │    │ • Internal Keys │
│ • Serial Number │    │                 │    │                 │
│ • Sensor Values │    │                 │    │                 │
│ • IP Address    │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
  Always Included      include_sensitive_data=True    Never Included
```
