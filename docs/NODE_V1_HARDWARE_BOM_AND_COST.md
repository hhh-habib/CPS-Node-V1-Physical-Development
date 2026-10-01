# Current hardware BOM and historical cost evidence

This is the **Stage 5A installed hardware list**. Known amounts come only from the earlier owner-reported procurement record; they are historical listed costs, not current prices or proof of a fully costed current assembly.

## Current parts

| Component | Current quantity | Historical recorded cost (BDT) | Current role |
|---|---|---:|---|
| ESP32-S3 N16R8 development board | 1 | 840 | Controller, Wi-Fi and peripheral interfaces |
| MQ-2 module | 1 | 160 | Analog gas/smoke observation through protection |
| DHT22 | 1 | 220 | Temperature/humidity |
| Digital flame sensor | 1 | 60 | Flame indication |
| 1.8-inch ST7735S TFT | 1 | 590 | Local telemetry/network/alarm display |
| nRF24L01+ PA+LNA with SMA antenna | 1 | not recorded | PIKU heartbeat/status monitoring |
| Regulated radio adapter/base board | 1 | not recorded | Regulated radio supply from 5 V input |
| 18650 cells | 2, in 2S | 400 for the pair | Portable source |
| 2S charger/protection module | 1 | 150 for earlier listed charging module | Pack charging/protection; current exact module cost not separately established |
| LM2596 buck converter | 1 | 240 | Regulated 5 V rail |
| Two-cell holder | 1 | 50 | Battery mounting/interconnection |
| Main switch | 1 | 5 | Power switching |
| Jumper wires | Set; exact count not recorded | 50 for earlier set | Prototype interconnection |
| Protoboard, connectors and mounting materials | As fitted; exact quantities not recorded | not recorded | Assembly and interconnection |
| MQ-2 divider/interface protection parts | As fitted; exact quantities not recorded | not recorded | GPIO4 analog interface protection |

**No current total is published.** Radio, adapter and additional assembly costs are missing; the earlier component values do not establish a complete current procurement bill. See [hardware](HARDWARE_ARCHITECTURE.md) and [power/interface architecture](POWER_AND_INTERFACE_ARCHITECTURE.md) for the installed arrangement.

## Historical pre-nRF build value

The earlier SG90/HC-SR04 revision had an **owner-reported/estimated approximate build value of BDT 2,985**. Its listed SG90 and HC-SR04 costs were BDT 120 and BDT 100 respectively. Those devices are retired and excluded from the current BOM.

That value belongs only to the historical pre-nRF build; it is not the Stage 5A total or a current market quotation. It excluded labor, development tools, computing equipment, replacement/spare parts and unlisted fabrication/mechanical materials. Original itemization remains in Git history; [historical physical evidence](history/README.md) identifies that revision.

The Node V1 BOM does not include a robot chassis, motor driver or drivetrain and cannot be presented as the cost of a complete PIKU robot. Missing prices are recorded as `not recorded`; no price or economic-superiority claim is inferred.
