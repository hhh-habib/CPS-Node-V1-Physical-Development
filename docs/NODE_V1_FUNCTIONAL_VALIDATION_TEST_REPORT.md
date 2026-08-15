# Node V1 Functional Validation Test Report

## 1. Purpose

This document records the practical functional validation performed on **Node V1**, an ESP32-S3-based wireless telemetry and command node. The purpose of the testing was to verify stable operation, dashboard responsiveness, local SoftAP communication, infrastructure Wi-Fi operation, remote radar-command execution, and alarm behavior before preparation of the research paper.

> **Important:** These tests are functional prototype-validation tests. They are based on direct observation during operation and were not performed with laboratory-grade network instrumentation. Therefore, this document reports only observations that were actually made and avoids fabricating precise latency, packet-loss, BER, SNR, or throughput values.

---

## 2. System Under Test

**Platform:** Node V1  
**Main controller:** ESP32-S3 N16R8  
**Communication:** Wi-Fi AP+STA with embedded HTTP/JSON interface  
**User interface:** Browser-based dashboard and local TFT display  
**Relevant functions tested:**
- Environmental telemetry
- Dashboard updates
- SoftAP communication
- STA/home-router communication
- Manual radar control
- HC-SR04 + SG90 directional observation
- Alarm-state behavior
- Continuous system operation

---

## 3. Test Environment

The prototype was evaluated in a normal indoor residential environment.

Two network operating conditions were exercised:

1. **SoftAP mode** — the user device communicated directly with Node V1 through the ESP32-S3 access point.
2. **STA mode** — Node V1 connected through a home Wi-Fi router.

Testing included operation from different positions within the residence and remote manual control of the radar head.

---

## 4. Continuous Operation / Stability Test

### Method

Node V1 was powered and operated continuously for **more than one hour** while the dashboard, sensing functions, wireless communication, and control functions remained active.

### Observation

- The system remained operational for the entire test period.
- The browser dashboard continued to update and respond.
- No system crash, freeze, or loss of basic functionality was observed.
- No manually observed communication interruption required a reboot or recovery action.

### Result

**PASS**

The prototype demonstrated stable continuous operation for more than one hour under the tested conditions.

---

## 5. Dashboard Responsiveness Test

### Method

The browser dashboard was used repeatedly during normal system operation to observe telemetry and issue control commands.

### Observation

- Dashboard updates appeared responsive throughout testing.
- Operator commands produced an immediate-feeling response during normal use.
- No human-perceptible delay that interfered with operation was observed.

### Result

**PASS**

### Paper-safe interpretation

The result should be described as:

> “The browser dashboard remained responsive during functional testing, and no human-perceptible command-response delay that affected operation was observed.”

This test does **not** establish zero network latency. Precise RTT values were not instrumentally recorded during this validation session.

---

## 6. SoftAP Communication Range Test

### Method

A user device was connected directly to the ESP32-S3 SoftAP while Node V1 remained operational. Communication and control were checked while increasing separation within the available indoor environment.

### Observation

Reliable SoftAP operation was observed at approximately:

**50–60 ft (about 15–18 m)**

within the tested indoor environment.

During this range test:
- the dashboard remained accessible,
- telemetry remained usable,
- commands remained operational.

### Result

**PASS**

### Limitation

This value is an **observed indoor functional range**, not a guaranteed RF specification. Actual range can vary with walls, interference, antenna orientation, device placement, and local RF conditions.

---

## 7. STA / Home Wi-Fi Coverage Test

### Method

Node V1 was connected to the home Wi-Fi network in STA mode and operated from different locations within an approximately **1800 ft² residence**.

### Observation

- Node V1 remained usable throughout the tested home environment.
- Dashboard access and command operation remained satisfactory.
- Coverage through the infrastructure network was better than the direct SoftAP test in the tested residence.

### Result

**PASS**

### Interpretation

The approximately **1800 ft²** observation describes the tested household environment and must **not** be presented as an intrinsic Node V1 range specification.

STA-mode coverage depends strongly on:
- router capability,
- router placement,
- walls and building materials,
- interference,
- client-device characteristics.

A suitable paper statement is:

> “In STA mode, Node V1 remained accessible throughout the approximately 1800 ft² test residence; however, infrastructure-mode coverage is dependent on the connected router and indoor propagation environment.”

---

## 8. Remote Manual Radar Command Test

### Method

The dashboard manual-radar controls were used remotely to command the SG90-mounted HC-SR04 observation head.

Commands included directional/manual radar movement while the operator was located approximately:

**50–80 ft (about 15–24 m)**

from the node under the tested network conditions.

### Observation

- Radar-head movement followed the issued dashboard commands.
- Manual directional operation remained responsive.
- No command-response delay perceptible enough to interfere with manual operation was observed.
- The bidirectional control path remained functional at the tested distances.

### Result

**PASS**

### Communication path demonstrated

**Operator → Browser Dashboard → HTTP Command → Wi-Fi → ESP32-S3 → Command Processing → Radar/Servo Action → Updated State/Telemetry**

This provides practical functional evidence of Node V1's bidirectional telemetry-and-command architecture.

---

## 9. Alarm Behavior / False-Alarm Observation

### Method

Node V1 remained active during the extended functional test while environmental sensing and alarm evaluation were enabled.

### Observation

- No false alarm was observed during the test period.
- Alarm behavior did not interfere with normal operation.
- The system continued to report environmental status normally.

### Result

**PASS under tested conditions**

### Limitation

This observation does not prove a statistically zero false-alarm rate. The academically correct statement is:

> “No false alarm was observed during the conducted functional-validation period.”

---

## 10. Functional Test Summary

| Test | Observed Result | Status |
|---|---|---|
| Continuous operation | More than 1 hour of stable operation | PASS |
| Dashboard responsiveness | Responsive; no human-perceptible delay affecting operation | PASS |
| SoftAP communication | Reliable at approximately 50–60 ft indoors | PASS |
| STA/home Wi-Fi | Functional throughout approximately 1800 ft² residence | PASS |
| Remote radar command | Successful manual control at approximately 50–80 ft | PASS |
| Bidirectional command path | Dashboard commands produced expected radar/servo actions | PASS |
| Alarm behavior | No false alarm observed during validation period | PASS |
| Overall prototype validation | All conducted functional tests were satisfactory | PASS |

---

## 11. Command Success Interpretation

During the performed manual tests, issued commands that were observed by the operator produced the expected radar-head response.

However, the **exact number of attempted commands was not formally counted** during this validation session.

Therefore, the research paper should **not invent a numerical command-success percentage** such as 100% unless a counted trial set is later performed.

The safe claim is:

> “All manually observed command trials during the conducted functional tests produced the expected radar response; however, a fixed trial count was not recorded.”

---

## 12. Latency Interpretation

The testing demonstrated subjectively responsive operation, but precise numerical latency values were not recorded.

Therefore:

### Supported statement
> “No human-perceptible command-response delay that affected manual operation was observed.”

### Unsupported statements
Do **not** claim:
- zero latency,
- exact command RTT,
- exact node-apply latency,
- exact applied RTT,
- sub-millisecond response,
- deterministic network latency,

unless those values are later measured using the firmware/dashboard instrumentation.

---

## 13. What These Tests Validate

The conducted tests provide practical evidence that Node V1 can:

1. remain operational continuously for an extended prototype-validation session;
2. provide responsive browser-based telemetry and control;
3. support direct local communication through ESP32-S3 SoftAP;
4. operate through an infrastructure/home-router network in STA mode;
5. execute remote radar/servo commands over the wireless command channel;
6. maintain functional bidirectional communication at meaningful indoor distances;
7. perform environmental monitoring without an observed false alarm during the test period;
8. function as a self-contained embedded telemetry-and-command CPS node.

---

## 14. What These Tests Do Not Validate

The current functional tests do **not** constitute laboratory characterization of:

- bit-error rate (BER),
- signal-to-noise ratio (SNR),
- PHY-layer throughput,
- RF packet-loss probability,
- deterministic latency,
- calibrated MQ-2 gas concentration in ppm,
- certified safety performance,
- statistically established false-alarm probability,
- long-duration reliability over days/weeks,
- multi-node scalability.

These should not be claimed in the paper without additional dedicated measurements.

---

## 15. Threats to Validity

The following limitations should be acknowledged when interpreting the results:

- Testing was performed in a single practical indoor environment.
- SoftAP range can vary with obstacles and interference.
- STA coverage depends strongly on the external router.
- Exact command counts were not formally logged.
- Numerical RTT/latency measurements were not preserved during this validation session.
- The one-hour stability test demonstrates prototype stability under the tested conditions but does not establish long-term reliability.
- False-alarm performance was observational rather than statistically characterized.

---

## 16. Paper-Ready Experimental Statements

The following statements are considered safe for use in the research paper:

1. **Stability:**  
   “Node V1 was operated continuously for more than one hour without an observed crash or loss of core functionality.”

2. **SoftAP range:**  
   “Direct SoftAP-based access remained operational at approximately 50–60 ft in the tested indoor environment.”

3. **Infrastructure Wi-Fi:**  
   “In STA mode, the node remained accessible throughout the approximately 1800 ft² test residence, although coverage is dependent on router capability and indoor propagation conditions.”

4. **Remote control:**  
   “Manual radar commands were successfully executed from approximately 50–80 ft under the tested network conditions.”

5. **Responsiveness:**  
   “The browser dashboard remained responsive, with no human-perceptible command-response delay that interfered with operation.”

6. **Alarm behavior:**  
   “No false alarm was observed during the conducted functional-validation period.”

7. **Bidirectional CPS operation:**  
   “The tests verified practical bidirectional operation in which browser-issued HTTP commands were received and applied by the embedded node while system state was returned through telemetry.”

---

## 17. Final Validation Verdict

**Overall Result: PASS — Functional Prototype Validation**

Node V1 successfully completed the conducted practical validation tests for continuous operation, dashboard interaction, SoftAP communication, infrastructure Wi-Fi operation, remote radar control, and alarm behavior.

The results support describing Node V1 as a **working ESP32-S3-based bidirectional wireless telemetry and command prototype** with local and browser-based supervision.

For the research paper, these results should be presented as **functional and application-level validation**, while avoiding unsupported claims about zero latency, RF-layer performance, calibrated gas concentration, or statistically proven reliability.
