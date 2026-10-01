# Current hardware architecture

Stage 5A is the fixed environmental Node V1, using ESP32-S3 N16R8. The [frozen software pin-map authority](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/HARDWARE_INTERFACE_AND_PINMAP.md) and [owner-reported physical record](STAGE5A_PHYSICAL_VALIDATION.md) define this baseline.

![Current prototype](../figures/node_v1.png)

*Installed prototype with environmental sensors, TFT, shared-SPI radio and portable power. The screen is a captured telemetry state, not a measurement specification.*

## Subsystems

| Hardware | Physical role |
|---|---|
| ESP32-S3 N16R8 | Controller with 16 MB flash / 8 MB PSRAM configuration; local sensing, alarms and interfaces |
| MQ-2 | Gas/smoke analog observation through existing divider/interface protection; raw/filtered ADC only |
| DHT22 | Temperature and humidity input |
| Digital flame sensor | Active-low flame indication |
| 1.8-inch ST7735S TFT | Local telemetry/network pages and alarm override |
| nRF24L01+ PA+LNA with SMA antenna | PIKU heartbeat/status peer monitoring; mounted on regulated adapter/base board |
| 2S 18650 pack, charger/protection, switch and LM2596 | Portable source and regulated 5 V rail |
| Protoboard, holder, connectors and wiring | Prototype mounting and interconnection |

## GPIO map

| Connection | GPIO |
|---|---:|
| MQ-2 analog through protection | 4 |
| DHT22 data | 5 |
| Flame digital | 6 |
| TFT RST | 8 |
| TFT D/C | 9 |
| TFT CS | 10 |
| Shared SPI MOSI | 11 |
| Shared SPI SCK | 12 |
| Shared SPI MISO, radio return | 13 |
| nRF CE | 14 |
| nRF CSN | 15 |
| Free/reserved | 16 |

![Current circuit and wiring](../figures/circuit_diagram.png)

*Current module connections and supply domains; the radio's 5 V connection is its regulated adapter input.*

## Shared SPI

TFT and nRF share the bus, with TFT CS on GPIO10 and nRF CSN on GPIO15. Radio CE on GPIO14 controls radio operation and is separate from chip select. MOSI and SCK are shared; MISO provides the nRF return path. Display startup initializes the bus before the radio joins it. Sequential library transactions in the cooperative application select each device independently.

The [power/interface record](POWER_AND_INTERFACE_ARCHITECTURE.md) gives supply distribution and reproduction checks. The [BOM](NODE_V1_HARDWARE_BOM_AND_COST.md) records parts without inventing missing costs.

## Retired historical hardware

**SG90, HC-SR04, radar head and directional scan-head logic are retired.** Their earlier GPIO assignments do not apply: GPIO14/15 now serve radio CE/CSN, and GPIO16 is free/reserved. Genuine earlier physical observations are under [history](history/README.md); they are not current Stage 5A evidence.
