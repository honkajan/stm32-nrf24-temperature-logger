# Remote Node Hardware

This document provides the schematic and revision history for the STM32 + nRF24 temperature sensor node.

## View schematic

PDF:
[remote_node_schematic_revB.pdf](exports/remote_node_schematic_revB.pdf)

SVG:
[remote_node_schematic_revB.svg](exports/remote_node_schematic_revB.svg)

## Source files

Editable KiCad project files:

docs/hardware/kicad/

Designed with **KiCad 9.0.7**

## Hardware revisions

### RevA
- ADC RC filter: 10 kΩ + 470 µF
- Long startup settling (~20 s required)

### RevB
- ADC RC filter: 10 kΩ + 10 µF
- Reduced startup settling (~20 s → ~3 s)
- Improved measurement responsiveness after power-up

This change significantly improves usability by reducing the delay before valid temperature readings are available after power-up.