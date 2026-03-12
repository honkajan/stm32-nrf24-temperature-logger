# Remote Firmware

This directory refers to the STM32 firmware used in the remote wireless temperature measurement node.

The actual firmware source code is maintained in a separate repository:

- **remote-fw**: https://github.com/honkajan/remote-fw

The remote node:
- performs local temperature measurement
- maintains the latest sampled values
- receives RF requests from the gateway
- returns measurement payloads over the nRF24 link

For system-level architecture, protocol documentation, and diagrams, see the main repository documentation under `docs/`.