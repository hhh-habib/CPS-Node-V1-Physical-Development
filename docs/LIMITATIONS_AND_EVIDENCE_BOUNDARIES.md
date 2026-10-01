# Limitations and evidence boundaries

- **MQ-2 ADC is not ppm.** Raw/filtered counts and engineering alarm defaults do not establish gas concentration or calibrated hazard limits.
- **Prototype sensors are not certified safety/metrology equipment.** No certified temperature/humidity/flame accuracy or life-safety performance is claimed.
- **RF is functionally observed, not characterized.** No measured nRF range, latency, loss, reconnect distribution, reliability percentage or MTBF dataset is supplied. Dashboard quality percentages are RSSI-derived UI heuristics, not success rates; browser timing/counters are not RF characterization.
- **Power remains uncharacterized.** No battery-runtime, current-draw, ripple or rail-transient result is supplied. Functional coexistence does not establish electrical margin or safety certification.
- **Physical PASS belongs to the owner report.** It describes supervised qualitative operation on 2026-10-02. Codex did not perform hardware tests; source checks do not substitute for physical measurement.
- **Future control remains future.** No Stage 5B coordinator robot-drive validation, Robot 2 integration or Robot 2 nRF role is claimed in standalone Stage 5A.
- **No industrial certification.** The prototype has module wiring and local plaintext HTTP without application authentication/TLS; it is not a qualified industrial deployment or hard real-time system.
- **Historical evidence stays historical.** Earlier SG90/HC-SR04 and radar-head observations are confined to the [historical record](history/README.md), not reused as current nRF performance evidence.

Read the [current physical report](STAGE5A_PHYSICAL_VALIDATION.md) and [testing overview](TESTING_AND_VALIDATION.md) for the supported evidence. Exact software behavior belongs to the [frozen release](SOFTWARE_RELEASE_REFERENCE.md).
