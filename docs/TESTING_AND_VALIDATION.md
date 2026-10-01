# Testing and validation

Stage 5A evidence has three distinct levels. Physical documentation synchronization does not rebuild, modify or retest the frozen firmware.

| Evidence level | Established record | Boundary |
|---|---|---|
| Frozen software verification | Released software reports PlatformIO build PASS, 22 automated source contracts with no skips, embedded JavaScript parse and release/source checks | Source contracts and parsing do not exercise physical ESP32 drivers or instrument RF/NVS faults; these checks were not rerun for this physical documentation release |
| Owner-reported physical functional validation | Supervised boot, sensors, TFT, dashboard, networking, candidate/LKG, local READY, mutual PIKU visibility, reboot reacquisition and coexistence PASS | Qualitative project-owner observations; Codex did not perform physical tests |
| Instrumented metrics | No new Stage 5A measurement dataset supplied | No numerical RF latency/loss/range, reconnect distribution, reliability, power or battery-runtime result |

## Software authority

Use the [software release reference](SOFTWARE_RELEASE_REFERENCE.md) for the exact executable, merge and tag. The [frozen software testing overview](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/TESTING_AND_VALIDATION.md) records checks and their scope; [software physical validation](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/STAGE5A_PHYSICAL_VALIDATION.md) supplies the owner-report boundary. Do not maintain a duplicate test or API specification here.

## Current physical evidence

The [2026-10-02 physical record](STAGE5A_PHYSICAL_VALIDATION.md) lists reported PASS observations and the defective PIKU jumper-wire finding. The [README screenshots](../README.md) show current UI states; captured IPs, readings, counters and percentages are session snapshots, not general specifications or a performance dataset.

Candidate/LKG was exercised qualitatively. The report does not independently establish every authentication, timeout, persistence-fault or legacy-upgrade case. Supervised coexistence PASS does not establish a numerical reliability percentage or long-duration qualification.

## Physical repository release checks

This documentation release checks Markdown/image links, stale current claims, GPIO/power/software consistency, whitespace, intended figure bytes/deletions, and commit scope. It also compares read-only reference-project file hashes and Git states with pre-work snapshots, then verifies pushed main/tag identity. No firmware build is required in this repository.

## Historical observations and unmeasured quantities

The [pre-nRF physical report](history/PRE_NRF_NODE_V1_FUNCTIONAL_VALIDATION.md) preserves observations from the retired SG90/HC-SR04 revision. Its indoor distances, duration and directional-command observations are historical; they are not Stage 5A nRF evidence.

RF latency/loss/range, exact DHCP/reconnect timing distributions, long-duration reliability, rail ripple/transients, current draw, battery runtime and calibrated sensor accuracy remain uncharacterized. See [limitations and evidence boundaries](LIMITATIONS_AND_EVIDENCE_BOUNDARIES.md).
