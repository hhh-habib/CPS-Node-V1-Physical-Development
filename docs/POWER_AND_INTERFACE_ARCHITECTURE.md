# Power and interface architecture

This is the current Stage 5A prototype distribution, synchronized with the [frozen software hardware interface](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/HARDWARE_INTERFACE_AND_PINMAP.md).

![Power and electrical architecture](../figures/power_architecture.png)

*Battery source, charger/protection, main switch, buck converter and two supply domains with common ground.*

## Power flow

```text
2S pack (two 18650 cells)
  -> 2S charger/protection module
  -> main switch
  -> LM2596 buck converter
  -> regulated 5 V rail
```

The ESP32 development board receives 5 V through 5VIN and provides the 3V3 domain used below. Battery nominal/full-charge labels in the diagram describe the pack configuration, not measured runtime or available load capacity.

| Supply | Current destination |
|---|---|
| Regulated 5 V rail | ESP32-S3 5VIN |
| Regulated 5 V rail | ST7735S TFT VCC |
| Regulated 5 V rail | MQ-2 VCC |
| Regulated 5 V rail | nRF regulated adapter/base-board input |
| ESP32 3V3 | DHT22 VCC |
| ESP32 3V3 | Digital flame sensor VCC |
| ESP32 3V3 | TFT backlight |
| Common ground | Controller, sensors, TFT, radio adapter and power modules |

## Radio adapter and logic domain

The installed nRF24L01+ PA+LNA is powered through a **regulated adapter/base board**. The 5 V rail goes to that adapter input, **not directly to a bare nRF24 chip**. Confirm the installed adapter's input rating and regulated output before reproducing the assembly. An adapter's supply-input rating does not establish 5 V tolerance on SPI/control signals; ESP32 signals use its 3.3 V domain.

## MQ-2 analog interface

MQ-2 analog output reaches GPIO4 through the existing voltage-divider/interface protection. Keep that protection in the design. The diagram illustrates the interface; verify the actual resistor placement, ratio, worst-case output and ESP32 ADC input limits before energizing a reproduction. Neither a diagram nor a historical divider sketch establishes measured analog headroom. Readings remain raw/filtered ADC counts, not calibrated gas concentration in ppm.

## Reproduction checks and evidence limits

Check cell condition/polarity, series-pack and charger/protection compatibility, module ratings, switch wiring, unloaded buck output, common ground and signal-level protection before connecting loads. Confirm the actual TFT module supports this VCC/backlight arrangement; it is the installed prototype configuration rather than a universal TFT-module rule.

Owner-reported integrated functional operation is recorded in [Stage 5A physical validation](STAGE5A_PHYSICAL_VALIDATION.md). This release supplies no current-draw, rail-ripple/transient, battery-runtime or electrical safety-certification dataset. The diagrams are design documentation, not an instrumented power qualification.

See [hardware architecture](HARDWARE_ARCHITECTURE.md) for GPIO/shared SPI and [limitations](LIMITATIONS_AND_EVIDENCE_BOUNDARIES.md) for evidence boundaries.
