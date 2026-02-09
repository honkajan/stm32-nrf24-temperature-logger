# Firmware Architecture

This document describes the firmware structure and design decisions for the
embedded nodes in the system. The goal is not only to explain *what* the firmware
does, but *why* it is structured the way it is.

The system consists of two STM32-based nodes with clearly separated roles:
a remote sensing node and a gateway node.

---

## Design goals

The firmware was designed with the following goals in mind:

- **Deterministic behavior**
- **Clear separation of responsibilities**
- **Minimal implicit behavior**
- **Ease of debugging and validation**
- **Support for long-running operation**

These goals influenced both the overall architecture and many smaller
implementation choices.

---

## Node roles

### Remote node

The remote node is responsible for *measurement only*.

Its main tasks are:

- Sampling two NTC temperature sensors via ADC
- Converting raw ADC values into temperatures
- Maintaining a cached copy of the most recent sample
- Responding to RF requests from the gateway

The remote node does **not** initiate communication and does not transmit
periodically. This avoids unnecessary RF traffic and makes system behavior
fully observable from the gateway.

#### Sampling and caching

Temperature sampling is performed periodically (≈1 Hz).  
Each sample updates an internal cache containing:

- temperatures in millidegrees Celsius
- raw ADC readings
- status flags
- timestamp or age information

When a temperature request is received over RF, the cached sample is returned
immediately without blocking on a new ADC conversion.

This decoupling ensures predictable RF response times.

---

### Gateway node

The gateway node acts as an explicit boundary between:

- wireless RF communication, and
- host-side UART communication

Its responsibilities include:

- Issuing RF commands to the remote node
- Receiving and validating RF responses
- Translating binary RF payloads into textual UART responses
- Enforcing request/response-only UART behavior

The gateway does not perform temperature sampling itself.

---

## Pull-based communication model

Both RF and UART communication follow a strictly pull-based model:

- No background transmissions
- No unsolicited output
- All data transfers are initiated by explicit requests

This model was chosen to:

- simplify host-side tooling
- avoid synchronization issues on UART
- make failures and timeouts explicit
- support reliable scripting and automation

---

## Fixed-size RF packets

RF communication uses fixed-size (32-byte) packets.

Benefits of this approach:

- predictable memory usage
- simple serialization and deserialization
- reduced risk of buffer overflows
- easier debugging and inspection

The RF payload layout is documented separately in `docs/protocol.md`.

---

## Error handling and robustness

The firmware favors explicit error reporting over silent retries.

Examples:

- RF timeouts result in a failed transaction
- Invalid or missing responses are propagated to the host
- No automatic retry loops at the firmware level

Retry and recovery strategies are implemented at the host side, where timing
and policy decisions are easier to adjust.

---

## Debugging and validation

Several design choices were made specifically to support debugging:

- Human-readable UART protocol
- No asynchronous UART output
- Clear command/response boundaries
- Exposure of raw ADC values alongside converted temperatures

These choices allow validation of the entire signal chain:

NTC sensor → ADC → firmware → RF → gateway → UART → Python tooling

---

## Real-world operation

The firmware has been validated in long-running operation:

- continuous sampling over many hours
- observed RF packet loss below 1%
- temperature behavior matching physical heating system dynamics

This validation influenced confidence in the chosen architecture and protocols.

---

## Summary

The firmware is intentionally simple, explicit, and conservative in its behavior.
Rather than optimizing for throughput or minimal latency, it prioritizes
predictability, observability, and correctness in a real-world environment.
