# SSL/HTTPS Support

The client supports SSL/HTTPS connections for secure communication with your PoolDose device. This is particularly useful when the device is configured for HTTPS or when connecting over untrusted networks.

## Basic SSL Configuration

```python
from pooldose.client import PooldoseClient

# Enable SSL with default settings (port 443, certificate verification enabled)
client = PooldoseClient("192.168.1.100", use_ssl=True)
status = await client.connect()
```

## SSL Configuration Options

```python
# Custom HTTPS port
client = PooldoseClient("192.168.1.100", use_ssl=True, port=8443)

# Disable SSL certificate verification (not recommended for production)
client = PooldoseClient("192.168.1.100", use_ssl=True, ssl_verify=False)

# Complete SSL configuration example
client = PooldoseClient(
    host="pool-device.local",
    timeout=30,
    use_ssl=True,
    port=8443,
    ssl_verify=True,  # Verify SSL certificates
    include_sensitive_data=False,
    include_mac_lookup=False
)
```

## SSL Security Considerations

- **Certificate Verification**: By default, SSL certificate verification is enabled (`ssl_verify=True`). This ensures secure connections but requires valid certificates.
- **Self-signed Certificates**: If your device uses self-signed certificates, set `ssl_verify=False`. Note that this reduces security.
- **Port Configuration**: Use the `port` parameter to specify custom HTTPS ports. Defaults to 443 for HTTPS and 80 for HTTP.
- **Connection Timeouts**: Consider increasing the `timeout` value for SSL connections as they may take longer to establish.

## Migration from HTTP to HTTPS

To migrate existing code from HTTP to HTTPS:

```python
# Before (HTTP)
client = PooldoseClient("192.168.1.100")

# After (HTTPS with SSL verification)
client = PooldoseClient("192.168.1.100", use_ssl=True)

# After (HTTPS with custom port and no verification)
client = PooldoseClient("192.168.1.100", use_ssl=True, port=8443, ssl_verify=False)
```
