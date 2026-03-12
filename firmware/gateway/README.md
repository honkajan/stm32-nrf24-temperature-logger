# Gateway Firmware

This directory refers to the STM32 firmware used in the UART-to-nRF24 gateway node.

The actual firmware source code is maintained in a separate repository:

- **gateway-fw**: https://github.com/honkajan/gateway-fw

The gateway node:
- receives UART commands from the host tool
- performs RF request/response transactions with the remote node
- returns formatted UART responses back to the host

For system-level architecture, protocol documentation, and diagrams, see the main repository documentation under `docs/`.