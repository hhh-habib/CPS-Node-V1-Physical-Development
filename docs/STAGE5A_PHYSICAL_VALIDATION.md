# Stage 5A physical validation - 2026-10-02

**Result: OWNER-REPORTED SUPERVISED PHYSICAL FUNCTIONAL PASS.**

| Reference | Record |
|---|---|
| Validation date | 2026-10-02, project-owner report, Asia/Dhaka |
| Validated software executable | `8f6939949cfb0ee3c0feb9319f0e6a2dd87de42b` |
| Released software main merge | `70ef949ce18ab81cbc207fed13bcc320d8b99964` |
| Software release tag | `nodev1-stage5a-standalone-physical-validated-20261002` |
| Scope | Standalone Node V1 with supervised PIKU 2.1 Rev A peer testing |

The project owner performed the intensive supervised functional tests and reported these observations. Codex records the supplied report and checks documentation consistency; Codex did not operate the hardware or acquire instrumented physical measurements. The [frozen software physical report](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/STAGE5A_PHYSICAL_VALIDATION.md) is the authoritative released wording.

## Hardware scope

ESP32-S3 N16R8; DHT22, MQ-2 through existing analog protection and digital flame sensor; 1.8-inch ST7735S TFT; nRF24L01+ PA+LNA with SMA antenna on regulated adapter/base board; 2S 18650 pack, charger/protection, main switch and LM2596 5 V rail with common ground. See [hardware](HARDWARE_ARCHITECTURE.md) and [power](POWER_AND_INTERFACE_ARCHITECTURE.md).

## Owner-reported observations

| Area | Reported result |
|---|---|
| Normal boot | PASS - normal ESP32-S3 startup |
| Environmental sensing | PASS - DHT22 temperature/humidity, MQ-2 ADC and flame monitoring |
| TFT and embedded dashboard | PASS - local display and browser interface operate |
| Live STA identity/address | PASS - current STA SSID/IP shown; opening that TFT-displayed IP reaches the dashboard |
| STA and SoftAP | PASS - both access paths operate, with separate AP identity/IP |
| STA loss/recovery | PASS - loss/recovery exercised; restored connection presents current IP |
| Candidate/LKG | PASS - behavior exercised during supervised testing; no per-case statistics supplied |
| Local nRF health | PASS - Node V1 radio READY |
| Peer monitoring | PASS - Node V1 sees PIKU ONLINE; PIKU reports Node V1 peer online |
| PIKU reboot/reacquisition | PASS - peer returns automatically after reboot without reflashing |
| Coexistence | PASS - TFT, sensors, Wi-Fi, dashboard and nRF operate together without an observed functional regression during supervised use |

The candidate/LKG observation does not independently validate every storage-fault, authentication, timeout or legacy-upgrade case. Local READY and peer ONLINE are separate evidence states; mutual reported visibility does not imply application-level Node V1 robot driving.

## Initial RF failure: two defective PIKU jumper wires

The owner traced the initial end-to-end failure to **two defective nRF jumper wires on PIKU 2.1 Rev A**. Replacing them restored peer communication and subsequent reboot/reacquisition worked. The finding was not a Node V1 firmware or protocol defect; software hardening alone did not repair the wires.

## Remaining boundaries

No RF latency, packet-loss percentage, maximum range, reconnect distribution, reliability percentage/MTBF, battery runtime, current draw, rail ripple/transient or certified sensor-accuracy result was supplied. Functional PASS is qualitative and supervised, not industrial qualification.

Stage 5B coordinator robot driving, Robot 2 integration and the combined CPS dashboard remain future work. See [testing](TESTING_AND_VALIDATION.md), [limitations](LIMITATIONS_AND_EVIDENCE_BOUNDARIES.md) and [current dashboard evidence](../README.md).
