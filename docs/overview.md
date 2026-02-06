# System Overview

This project implements a small but complete embedded telemetry system built around STM32 microcontrollers and nRF24L01+ radios. The system measures radiator temperatures, transmits them wirelessly, and logs long-term data on a Linux host for analysis.

The focus of the project is not only on measurement accuracy, but on **system-level reliability**, **clear interfaces**, and **reproducibility**.

---

## High-level architecture

The system consists of three main components:

1. **Remote node**
2. **Gateway node**
3. **Host PC (Linux)**

Each component has a clearly defined responsibility and communicates through explicit, request/response interfaces.

---

## Remote node (STM32 + nRF24)

The remote node is responsible for:

- Measuring two radiator temperatures using NTC sensors
- Sampling temperatures periodically (≈1 Hz)
- Caching the most recent measurement
- Responding to RF requests from the gateway

The remote node does not push data autonomously. All communication is initiated by the gateway to keep RF behavior deterministic and observable.

---

## Gateway node (STM32 + nRF24 + UART)

The gateway node acts as a protocol translator:

- Wireless RF communication with the remote node
- Text-based UART communication with the host PC

Its responsibilities include:

- Issuing RF commands (ping, temperature request)
- Receiving fixed-size binary RF packets
- Converting binary payloads into human-readable UART responses
- Ensuring the UART remains silent unless a command is explicitly issued

This design allows reliable scripting on the host side without having to deal with asynchronous or unsolicited output.

---

## Host PC (Python tooling)

The host PC communicates with the gateway via a serial UART interface.

A custom Python CLI tool is used to:

- Discover serial ports
- Issue commands to the gateway
- Fetch temperature samples on demand
- Log long-running measurements to CSV files
- Perform offline analysis and visualization

The tooling is designed to be script-friendly, reproducible, and suitable for long-running data collection.

---

## Design goals

The main goals of this project are:

- Demonstrate a complete embedded system, not just isolated firmware
- Use simple, explicit protocols that are easy to debug
- Support long-running operation and validation
- Make the system understandable and reproducible by others

Further documentation covers firmware structure, communication protocols, and data analysis in more detail.
