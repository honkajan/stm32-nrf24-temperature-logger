# Host Tools

This directory contains host-side tools used to interact with the embedded system
via the gateway node.

## UART CLI tool

The primary host-side interface is a Python-based UART control and logging utility,
maintained as a separate repository:

- **uartctl**: https://github.com/honkajan/uartctl

The tool provides a simple, script-friendly command-line interface for:
- discovering serial ports
- issuing request/response commands to the gateway
- fetching and logging temperature data
- producing CSV output suitable for long-running analysis and plotting

Keeping the UART tool as a standalone repository allows it to be reused across
projects while still integrating cleanly with this system.

