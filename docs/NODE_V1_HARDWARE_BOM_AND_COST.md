# Node V1 Hardware Bill of Materials and Prototype Cost

## Purpose

This document records the author-provided hardware cost of the **Node V1 ESP32-S3 wireless telemetry and command prototype** for use in the research-paper evidence package.

The values below are historical prototype procurement values supplied by the researcher. They should be described as **author-reported / author-estimated prototype costs**, not as current market prices.

## Detailed Bill of Materials

| No. | Component | Qty. | Unit / Listed Cost (BDT) | Subtotal (BDT) | Role in Node V1 |
|---:|---|---:|---:|---:|---|
| 1 | ESP32-S3 N16R8 development board | 1 | 840 | 840 | Main controller, Wi-Fi endpoint, embedded HTTP server, telemetry and command processing |
| 2 | MQ-2 gas/smoke sensor module | 1 | 160 | 160 | Raw/filtered analog environmental observation |
| 3 | DHT22 temperature/humidity sensor | 1 | 220 | 220 | Temperature and humidity acquisition |
| 4 | Flame sensor module | 1 | 60 | 60 | Digital flame-event input |
| 5 | 1.8-inch ST7735S TFT LCD, 128×160 | 1 | 590 | 590 | Local human-machine interface |
| 6 | SG90 micro servo | 1 | 120 | 120 | Directional radar/sensor-head actuation |
| 7 | HC-SR04 ultrasonic sensor | 1 | 100 | 100 | Directional distance observation |
| 8 | 2S 8.4 V charging module | 1 | 150 | 150 | Two-cell battery charging/power subsystem |
| 9 | LM2596 buck converter | 1 | 240 | 240 | Regulated 5 V power rail |
| 10 | 3.7 V 18650 battery | 2 | 400 total | 400 | Portable energy source |
| 11 | 2×18650 battery holder | 1 | 50 | 50 | Battery mounting/interconnection |
| 12 | Main power switch | 1 | 5 | 5 | User power control |
| 13 | Male-female jumper wires | Set | 50 | 50 | Prototype interconnection |
|  | **Total prototype hardware cost** |  |  | **2,985 BDT** |  |

## Cost by Engineering Subsystem

| Subsystem | Included Items | Cost (BDT) |
|---|---|---:|
| Controller and wireless communication | ESP32-S3 N16R8 | 840 |
| Environmental and directional sensing | MQ-2, DHT22, flame sensor, HC-SR04 | 540 |
| Local display / HMI | ST7735S TFT | 590 |
| Directional actuation | SG90 servo | 120 |
| Power and charging | 2S charger, LM2596, two 18650 cells, holder, switch | 845 |
| Interconnection | Male-female wires | 50 |
| **Total** |  | **2,985 BDT** |

## Paper-Safe Cost Statement

> The implemented Node V1 prototype used author-reported hardware costing approximately **BDT 2,985**, excluding labor, development tools, computing equipment, replacement/spare parts, and any unlisted fabrication or mechanical materials. The value represents the documented prototype build rather than a current commercial quotation or production cost.

## Scope Boundary

This cost describes the **standalone Node V1 sensing, communication, local-display, radar-head, and power subsystem**. It does **not** include the chassis, DC motors, motor driver, or other mobility hardware of a complete PIKU robotic platform.

When Node V1 concepts are discussed in relation to PIKU, this distinction should be preserved so that the paper does not incorrectly present the Node V1 cost as the total cost of a mobile robot.

## Research Value of the Cost Record

The cost record supports a low-cost engineering argument because it documents the approximate material cost required to reproduce the communication-centric prototype using commonly available embedded modules. It does not by itself establish economic superiority over other research platforms; any comparative cost claim would require normalized external procurement data.
