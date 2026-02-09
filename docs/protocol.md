# Communication Protocols

This document describes the communication protocols used in the system, covering
both the wireless RF protocol between embedded nodes and the UART protocol between
the gateway and the host PC.

The protocols are intentionally simple, explicit, and deterministic to support
reliability, debuggability, and reproducibility.

---

## Design principles

The following principles guided the protocol design:

- **Request/response only**
  No unsolicited messages are sent on RF or UART.

- **Deterministic behavior**
  All communication is initiated explicitly by the requester.

- **Human-readable where possible**
  UART uses line-based ASCII text for ease of debugging and scripting.

- **Fixed-size binary where appropriate**
  RF packets use fixed-size binary payloads for efficiency and predictability.

- **Clear separation of responsibilities**
  Each layer (remote, gateway, host) performs a single well-defined role.

---

## RF protocol (gateway ↔ remote)

### Overview

The RF protocol is used between the gateway node and the remote temperature-sensing node.
Communication is always initiated by the gateway.

Key characteristics:
- Fixed-size packets (32 bytes)
- Little-endian encoding
- Simple command/response semantics
- No periodic or autonomous transmissions from the remote node

---

### RF commands

| Command | Direction | Description |
|-------|----------|-------------|
| `PING` | Gateway → Remote | Connectivity check |
| `PONG` | Remote → Gateway | Response to `PING` |
| `GTMP` | Gateway → Remote | Request latest temperature sample |
| `TMP!` | Remote → Gateway | Temperature data response |

---

### Temperature response payload (`TMP!`)

The temperature response payload has the following layout
(all values little-endian):

| Field | Type | Description |
|-----|------|-------------|
| `T0_mC` | `int32` | Radiator 1 temperature in millidegrees Celsius |
| `T1_mC` | `int32` | Radiator 2 temperature in millidegrees Celsius |
| `flags` | `uint16` | Status and validity flags |
| `age_ms` | `uint16` | Age of the sample in milliseconds |
| `ADC0` | `uint16` | Raw ADC reading for sensor 1 |
| `ADC1` | `uint16` | Raw ADC reading for sensor 2 |

The remote node maintains a cached temperature sample updated periodically
(≈1 Hz). When a `GTMP` request is received, the most recent cached sample
is returned immediately.

---

## UART protocol (gateway ↔ host)

### Overview

The UART protocol connects the gateway node to a host PC.
It is a line-oriented, ASCII-based protocol designed for interactive use
and for scripting from host-side tools.

Key characteristics:
- One command per line
- Newline-terminated (`\n`)
- One-line response per command
- No unsolicited output from the gateway

This design allows robust use from scripts without needing to handle
asynchronous or background messages.

---

### UART commands

| Command | Description | Response |
|-------|------------|----------|
| `PING` | Gateway connectivity check | `PONG` |
| `ID?` | Query firmware identification | Free-form string |
| `VER?` | Query firmware version | `MAJOR.MINOR.PATCH` |
| `UPTIME?` | Query uptime | Decimal milliseconds |
| `RPING?` | RF ping remote node | `RPONG` or error |
| `TEMP?` | Fetch remote temperature | `TEMP ...` |

---

### Temperature response (`TEMP?`)

The temperature response is a single line of key=value fields:

```
TEMP ADC0=2070 ADC1=2015 T0=24966 T1=25926 age=856 flags=0x0007
```

Fields:
- `ADC0`, `ADC1` – raw ADC values
- `T0`, `T1` – temperatures in millidegrees Celsius
- `age` – sample age in milliseconds
- `flags` – status bitmask

This format is intentionally easy to parse from both shell scripts and
higher-level tools.

---

## Error handling

- Timeouts result in no response
- Failed RF operations are reported by the gateway
- Host-side tools are expected to treat missing or malformed responses
  as failed samples

No retries are performed automatically at the protocol level; retry logic
is handled by the host tooling when appropriate.

---

## Rationale for no unsolicited output

The gateway never prints data unless a command is received.

This ensures:
- predictable behavior for scripting
- clean separation between control and logging
- compatibility with long-running host-side data collection

This constraint was chosen deliberately to avoid subtle failures in
serial-based automation.

---

## Summary

The protocols favor simplicity and explicitness over throughput or complexity.
They are designed to be easy to inspect, debug, and extend while supporting
long-running, real-world measurements.
