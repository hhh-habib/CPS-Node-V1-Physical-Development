# Node V1 physical development changelog

This is a human-friendly progression, not a reconstructed chronology. Dates for earlier phases are not established here.

## Early Node V1

ESP32-S3 environmental sensing, TFT presentation, local alarms, Wi-Fi dashboard access and portable battery power formed the physical prototype foundation.

## Historical SG90/HC-SR04 experiment

An earlier revision added a directional sensor head and browser radar commands. Genuine owner-observed physical results are preserved under [history](docs/history/README.md). They apply to that pre-nRF revision, not current Stage 5A.

## nRF migration and retired hardware

The fixed environmental node gained nRF24L01+ PA+LNA on a regulated adapter for PIKU heartbeat/status monitoring. SG90, HC-SR04, radar and directional-head logic were retired; GPIO14/15 became radio CE/CSN and GPIO16 became free/reserved.

## 2026-10-02 — Stage 5A final physical baseline

- Synchronized current hardware, GPIO, supply distribution, TFT/live-IP, Wi-Fi candidate/LKG and peer-state wording with validated executable `8f6939949cfb0ee3c0feb9319f0e6a2dd87de42b`, released software merge `70ef949ce18ab81cbc207fed13bcc320d8b99964` and tag `nodev1-stage5a-standalone-physical-validated-20261002`.
- Recorded owner-reported supervised functional PASS, including mutual PIKU visibility, reboot reacquisition and coexistence. Initial RF failure was traced to two defective nRF jumper wires on PIKU; replacement restored the link.
- Integrated the owner's current prototype, circuit, power, communication, software and CPS-platform figures, plus Environment, Communication, System and flame-alarm Overview screenshots. Image binaries were preserved.
- Removed obsolete radar-era figures, the duplicate local API document and large stale software/paper audit. Moved genuine older physical validation into clearly marked history.
- Rewrote the README and created a small current physical doc set, with frozen software references, evidence boundaries and no invented current cost total or performance metrics.

Physical release tag: `nodev1-physical-stage5a-synchronized-20261002`.

Next: future Stage 5B Coordinator Integration, then final combined CPS dashboard work. Neither is implemented by this physical documentation synchronization.
