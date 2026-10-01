# Frozen software release reference

The [CPS Node V1 Software Development repository](https://github.com/hhh-habib/CPS-Node-V1-Software-Development) is the implementation authority. This physical repository documents the installed design and evidence, with concise links instead of a second API/software specification.

| Release identity | Value |
|---|---|
| Validated executable | `8f6939949cfb0ee3c0feb9319f0e6a2dd87de42b` |
| Released main merge | `70ef949ce18ab81cbc207fed13bcc320d8b99964` |
| Annotated release tag | `nodev1-stage5a-standalone-physical-validated-20261002` |

The [executable commit](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/commit/8f6939949cfb0ee3c0feb9319f0e6a2dd87de42b) identifies tested code. The [main merge](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/commit/70ef949ce18ab81cbc207fed13bcc320d8b99964) and [tag](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/tree/nodev1-stage5a-standalone-physical-validated-20261002) identify the released software/documentation snapshot. Links below are pinned to that merge so later main changes cannot silently redefine this physical release.

| Frozen software document | Authority |
|---|---|
| [README](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/README.md) | Current standalone role, release and entrypoint |
| [Software explanation](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/SOFTWARE_EXPLANATION.md) | Startup, sensing, alarms and recovery behavior |
| [Software architecture](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/SOFTWARE_ARCHITECTURE.md) | Managers, scheduling, state ownership and shared SPI |
| [Hardware interface/pin map](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/HARDWARE_INTERFACE_AND_PINMAP.md) | Current GPIO and supply-domain assignments |
| [Communication architecture](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/COMMUNICATION_ARCHITECTURE.md) | STA/SoftAP, live-IP, candidate/LKG and nRF READY/peer semantics |
| [API](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/API.md) | Current routes, telemetry fields and errors |
| [Testing and validation](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/TESTING_AND_VALIDATION.md) | Released software verification and measurement boundaries |
| [Stage 5A physical validation](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/STAGE5A_PHYSICAL_VALIDATION.md) | Owner-reported physical PASS and defective PIKU jumper-wire finding |

The software's older `FINAL_HARDWARE_VALIDATION.md` explicitly identifies itself as historical pre-nRF evidence and points to the current interface/report above. Do not use its retired pin assignments as the Stage 5A map.

Standalone Stage 5A monitors PIKU peer packets and does not drive robots. Stage 5B coordinator integration and the combined CPS dashboard are future work.
