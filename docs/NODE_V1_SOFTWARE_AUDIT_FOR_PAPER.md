# Node V1 Software Engineering Audit for Research Paper

## 1. Executive Summary

Node V1 is an implemented ESP32-S3 cyber-physical sensing and command endpoint, not a generic monolithic IoT sketch and not a mobile robot. The production firmware acquires DHT22 temperature/humidity, MQ-2 raw ADC data, and an active-low digital flame input; controls an SG90-mounted HC-SR04 observation head; evaluates centralized alarm causes; exposes structured JSON telemetry and a reverse radar-command channel over a local HTTP server; maintains simultaneous SoftAP and station operation; and presents state on both an embedded browser dashboard and a local ST7735S TFT.

The software is separated into acquisition, radar/actuation, safety, networking, telemetry, HTTP, browser-asset, and display modules. `src/main.cpp` is primarily composition and cooperative orchestration. The HC-SR04 echo path is interrupt-assisted and does not use `pulseIn()` in production. Servo pulse generation uses hardware LEDC. The reverse channel provides separate command acceptance and application status, with browser-observed application-layer timing.

The strongest defensible paper characterization is a modular, bidirectional, local-first embedded CPS communication implementation with explicit validity, freshness, failure-state, and acknowledgement semantics. It is not evidence of a new Wi-Fi PHY, hard-real-time guarantees, calibrated gas concentration, autonomous navigation, object tracking, or certified safety operation.

## 2. Repository Scope and Audit Method

The audit inspected all current production headers and implementations under `include/` and `src/`, plus `platformio.ini`, `README.md`, `docs/API.md`, `docs/COMMUNICATION_ARCHITECTURE.md`, `FINAL_HARDWARE_VALIDATION.md`, `MIGRATION_NOTES.md`, and the historical bring-up program under `docs/bringup/`. Claims were checked against class state, call paths, scheduling conditions, HTTP route registration, JSON construction, and dashboard JavaScript. Source code takes precedence over descriptive documentation.

The production build consists of the `.cpp` files under `src/`. `docs/bringup/FINAL_INTEGRATED_HARDWARE_TEST.cpp` is a preserved hardware-test artifact and is not compiled. It contains blocking `pulseIn()` and delay-based servo validation, but those techniques are not present in the production source path. Mentions of motors, L298N, navigation, or IR obstacle logic occur only in migration/documentation text describing removed or prohibited legacy scope; no such production module exists.

A clean PlatformIO release build was run during this audit. No production firmware was changed and no upload was performed. The only repository modification made by the audit is this report.

## 3. Verified Software Architecture

| Module | Responsibility | Key Evidence | Audit Status |
|---|---|---|---|
| `src/main.cpp` | Constructs modules, orders initialization/update calls, dispatches queued commands, logs status | `setup()`, `loop()`, `applyPendingCommand()` | VERIFIED |
| `include/NodeConfig.h` | Identity, Wi-Fi constants, timing, thresholds, servo/radar calibration, display parameters | `namespace NodeConfig` constants | VERIFIED |
| `include/PinConfig.h` | Central GPIO assignment | `namespace PinConfig` | VERIFIED |
| `include/CommunicationTypes.h` | Command enum/envelope and communication counters/ACK state | `NodeCommandType`, `NodeCommand`, `CommunicationStats` | VERIFIED |
| `SensorManager` | Scheduled DHT22, MQ-2 ADC/filter, flame input, validity and timestamps | `SensorManager::update()`, `updateFastSensors()`, `updateDht()` | VERIFIED |
| `ServoScanner` | 50 Hz LEDC setup, logical-angle mapping, calibrated duty generation, PWM readiness | `ServoScanner::begin()`, `writeLogicalAngle()`, `pulseToDuty()` | VERIFIED |
| `RadarManager` | Continuous/manual head state, ultrasonic trigger/echo state machine, range-angle association, detection ring | `RadarManager::update()`, `updateSweep()`, `updateManual()`, `advanceMeasurement()` | VERIFIED |
| `SafetyManager` | Authoritative gas, temperature, and flame alarm causes with dwell and clear logic | `SafetyManager::updateGas()`, `updateTemperature()`, `updateFlame()` | VERIFIED |
| `NetworkManager` | AP+STA lifecycle, NVS credentials, retry, scan, RSSI/channel/IP, mDNS | `NetworkManager::begin()`, `update()`, `connectSta()`, `scanJson()` | VERIFIED |
| `TelemetryManager` | On-demand structured JSON representation with logical sequence/freshness and subsystem state | `TelemetryManager::update()`, `buildJson()` | VERIFIED |
| `CommunicationManager` | HTTP API, validation, one-slot command queue, acceptance/application ACK state, counters | `CommunicationManager::begin()`, `handleCommand()`, `acknowledgeCommand()` | VERIFIED |
| `DisplayManager` | ST7735S normal view, cached field updates, centralized-alarm override | `DisplayManager::update()`, `drawNormalFrame()`, `drawAlarmScreen()` | VERIFIED |
| `WebDashboard` | Registers embedded HTML/CSS/JS routes | `WebDashboard::registerRoutes()` | VERIFIED |
| `WebDashboardAssets` | Self-contained responsive SPA, polling, rendering, radar canvas, controls, provisioning, browser metrics/audio | `DASHBOARD_HTML`, `DASHBOARD_CSS`, `DASHBOARD_JS` | VERIFIED |
| Motor/drivetrain/navigation/IR obstacle stack | Mobile-robot behavior | No corresponding production header/source; only removal warnings in docs | NOT PRESENT |
| Multi-node coordinator, cloud backend, MQTT, database | Fleet aggregation/persistence | No implementation or dependency | NOT PRESENT |

Responsibilities are generally cohesive: sensor ownership is not mixed with HTTP, safety does not depend on display code, radar owns both physical command state and angle-associated ranging, and networking is isolated from telemetry serialization. Dependencies are mostly directed from presentation/transport modules toward state owners. `main.cpp` contains no sensor algorithms or web-page markup; its significant logic is the command switch that maps validated commands to `RadarManager` methods.

Maintainability is supported by centralized constants and pins, snapshot structs, explicit module interfaces, and a shared alarm owner. Extensibility is plausible at the module/schema level, but future multi-node, camera, audio, STM32, or AI functions are documented aspirations only.

## 4. Cooperative Scheduling and Real-Time Behavior

The application uses the Arduino `setup()`/`loop()` model with cooperative polling; it does not create application FreeRTOS tasks. Each loop calls sensors, radar, safety, network, telemetry, HTTP handling, command application, display, logging, and `yield()` in a fixed order (`src/main.cpp -> loop()`). This makes execution easy to trace but does not provide hard-real-time scheduling, priorities, deadline enforcement, or worst-case execution-time guarantees.

| Activity | Implemented timing | Source evidence |
|---|---:|---|
| Fast MQ-2/flame acquisition | 100 ms | `include/NodeConfig.h`; `SensorManager::update()` |
| DHT22 acquisition | 2500 ms | `include/NodeConfig.h`; `SensorManager::update()` |
| MQ-2 warm-up gate | 60000 ms | `NodeConfig::MQ2_WARMUP_MS`; `SensorManager::updateFastSensors()` |
| Logical telemetry sequence update | 250 ms (nominal 4 Hz) | `TelemetryManager::update()` |
| Browser telemetry polling | 250 ms; one request in flight | `DASHBOARD_JS -> setInterval(poll,250)`, `pollInFlight` |
| TFT update eligibility | 400 ms | `DisplayManager::update()` |
| Serial summary | 3000 ms | `src/main.cpp -> logStatus()` |
| Servo sweep step | 1 degree every 25 ms | `RadarManager::updateSweep()` |
| Continuous HC-SR04 start | At least 70 ms and at least 4 degrees since last sampled angle; effective nominal spacing is about 100 ms at 25 ms/degree | `RadarManager::measurementDue()` |
| Manual mechanical settle | 250 ms after LEFT/FRONT/RIGHT | `RadarManager::moveManual()` |
| Manual live ranging | 250 ms between starts while stationary | `RadarManager::updateManual()` |
| Explicit manual scan defer | 20 ms, also subject to 70 ms minimum interval | `RadarManager::requestManualScan()`, `updateManual()` |
| Echo timeout | 25000 us | `RadarManager::advanceMeasurement()` |
| Recent detection visibility | 3500 ms | `TelemetryManager::buildJson()` |
| STA reconnect attempt | 15000 ms | `NetworkManager::update()` |
| mDNS retry | 5000 ms | `NetworkManager::updateMdns()` |

Production HC-SR04 acquisition does not call `pulseIn()`. The trigger is a deterministic LOW/HIGH/LOW sequence containing 3 us and 10 us `delayMicroseconds()` calls, so each trigger deliberately blocks the cooperative loop for about 13 us plus GPIO overhead. Echo edges are captured by a `CHANGE` interrupt and shared through critical sections; completion and the 25 ms timeout are advanced from the loop. This is accurately described as interrupt-assisted, bounded, and non-blocking during echo waiting, not as completely delay-free.

Servo PWM is generated by ESP32 LEDC and continues independently of the loop after each duty update. Remaining synchronous work includes the DHT library read, HTTP `handleClient()`, dynamic JSON construction, Wi-Fi/library calls, TFT SPI drawing, serial output, and the short ultrasonic trigger. Their jitter has not been instrumented. The design is responsive cooperative firmware, not a formally verified real-time system.

## 5. Sensor Acquisition and Data Validity

`SensorManager` owns all environmental samples and metadata (`include/SensorManager.h -> SensorSnapshot`). GPIO use comes only from `PinConfig`.

- **DHT22:** `SensorManager::updateDht()` calls the DHT library for temperature and humidity every 2.5 s. Each channel has an independent validity flag. A failed read marks the latest attempt invalid but retains the last numeric valid value and its last-valid timestamp. Telemetry therefore can contain a retained number with `validity=false`; consumers must use both value and validity.
- **MQ-2:** `analogReadResolution(12)` is set at initialization. GPIO4 is sampled every 100 ms. `pushGasSample()` implements a 16-entry moving average with an incremental sum; startup uses the available partial window. Both raw and filtered ADC counts are exported. There is no voltage conversion, calibration curve, sensor resistance calculation, environmental compensation, or ppm calculation. A ppm or calibrated concentration claim is unsupported.
- **MQ-2 warm-up:** `mq2WarmingUp` remains true for 60 s from `SensorManager::begin()`. Gas alarm evaluation is disabled during this period. Measurements are still shown and filtered.
- **Flame input:** GPIO6 is sampled every 100 ms and interpreted active-low. It is a binary module output, not a measured flame intensity. The production code marks each GPIO read valid; there is no line-disconnect diagnostic.
- **Freshness:** DHT timestamps represent last valid samples; gas/flame timestamps represent the latest scheduled read. `TelemetryManager::ageJson()` converts these to ages or `null` when never initialized.

Filtering exists for MQ-2. Alarm dwell/clear timing and threshold hysteresis exist in `SafetyManager`; flame sampling itself has no separate acquisition debouncer. The code does not digitally filter DHT values beyond library behavior.

## 6. Radar and Servo Control Architecture

`RadarManager` is the single owner of radar mode, run state, current logical angle, latest measurement angle/range, echo status, sample sequence, and recent detections.

### Servo and movement

`ServoScanner::begin()` configures LEDC channel 0 for 50 Hz at 14-bit resolution and accepts setup only when the returned frequency is positive and within 1 Hz of 50 Hz. It then attaches GPIO16 and writes logical center. Logical 0/90/180 degrees map linearly to 700/1500/2300 us. With a 16383 maximum duty and 20000 us period, rounded duties are 573, 1229, and 1884. The implementation uses no repeated blocking pulse loop.

The readiness limitation is important: `servo_ready` proves that the LEDC setup returned an acceptable frequency and that software enabled the output path. There is no position sensor, current sensor, or physical-motion feedback. A disconnected, unpowered, jammed, or mechanically failed servo can still report `servo_ready=true`. Paper wording should use **servo PWM readiness**, not end-to-end actuator health.

### Continuous mode

When PWM setup succeeds, startup selects continuous mode, begins at 90 degrees, and advances one degree every 25 ms. Direction reverses at 0 and 180 degrees without long waits. Ranging begins only when the echo state is idle, at least 70 ms has elapsed, and the head has advanced at least four degrees. A 180-degree traversal is nominally 4.5 s and a full return cycle 9 s, exclusive of cooperative jitter.

### Manual mode

`RADAR_MANUAL` stops sweep movement. `LOOK_LEFT`, `LOOK_FRONT`, and `LOOK_RIGHT` issue one servo duty update to 0, 90, or 180 degrees, wait 250 ms for settling, then start an ultrasonic measurement. The stationary angle is subsequently ranged every 250 ms without rewriting or moving the servo. `RADAR_SCAN` requests an additional measurement at the current manual angle through the same state machine.

### Ultrasonic state and data association

`RadarManager::beginMeasurement()` snapshots the current angle into `measurementAngle_` before triggering. `finishMeasurement()` publishes that angle as `sampleAngle`, so the latest distance remains associated with the trigger-time angle even if the current sweep angle has changed. A valid echo is converted with a fixed 0.0343 cm/us speed-of-sound factor and accepted only from 2 to 150 cm. No echo produces `NO_ECHO`; an incomplete high pulse at timeout produces `FAULT`; a completed echo outside the accepted range produces `OUT_OF_RANGE`. Invalid results publish `distance_cm=null` and do not add a detection.

Valid measurements enter a 40-entry ring. Telemetry emits only entries no older than 3.5 s as `[angle, distance, age]`. These are fading observations, not tracked object identities, trajectories, occupancy mapping, or obstacle-avoidance decisions. HC-SR04 data is observation-only.

If PWM initialization or a software servo write fails, `RadarManager::setServoFault()` sets `servoReady=false`, stops movement, forces `STATIONARY`, and clears pending manual ranging. Movement commands return failure; telemetry and the browser expose the fault. The software angle is updated only after a successful servo write, preventing the known virtual-sweep mismatch. The TFT does not currently show a dedicated servo-fault message.

## 7. Centralized Safety Architecture

`SafetyManager` is the sole module that converts sensor readings into authoritative alarm causes. It owns a three-bit mask: high gas, high temperature, and flame. `TelemetryManager`, `DisplayManager`, serial logging, and browser JavaScript consume this state; they do not independently compare readings to alarm thresholds. Display/browser code formats summaries and colors, but does not create additional authoritative alarm decisions.

| Cause | Activation | Clear | Validity/warm-up behavior |
|---|---|---|---|
| High gas | Filtered raw ADC >= 2500 continuously for 1500 ms | <= 2300 continuously for 500 ms | Forced inactive and timers reset while invalid or during 60 s warm-up |
| High temperature | Valid retained reading >= 50.0 C for 1000 ms | <= 47.0 C for 500 ms | Invalid latest DHT attempt resets timing but does not clear an already active cause |
| Flame | Active-low detection for 100 ms | No detection for 500 ms | Invalid input causes no state transition; production acquisition normally marks the digital read valid |

The activation/clear timers run against the current held sample each loop, not a count of independent sensor samples. For temperature, a high DHT sample can satisfy the 1 s dwell before the next 2.5 s DHT acquisition. This is a valid state-hold design but should not be described as multiple-sample confirmation.

`SafetyManager::causesText()` and `stateName()` produce centralized summary/state. Telemetry also exposes the cause mask, normalized cause names, and compiled thresholds. HC-SR04 data is absent from `SafetyManager`'s interface and evaluation, conclusively excluding distance/radar from alarm generation.

This is prototype hazard annunciation, not a certified safety instrument. Thresholds are engineering defaults and MQ-2 values are uncalibrated.

## 8. Wi-Fi AP+STA and Network Management

`NetworkManager::begin()` explicitly selects `WIFI_AP_STA`, assigns hostname `node-v1`, and starts password-protected SoftAP `node_v1`. The AP is not intentionally stopped when STA connects, disconnects, scans, or retries. The design therefore provides a direct local path without a router and a concurrent infrastructure path through a home/access-point network.

Station credentials are loaded from and written to Preferences/NVS namespace `node-wifi`. Empty-password submission for the already stored SSID reuses the stored password without returning it to the browser. `connectSta()` validates SSID/password length, persists the values, and starts association. Unexpected STA loss is retried every 15 s while reconnect is enabled. User **Disconnect** retains credentials but disables retry for the remainder of that boot; **Forget** clears Preferences and also disables retry. A later connect request re-enables it. On reboot, retained credentials are loaded and association resumes.

Network scans are asynchronous and return SSID, RSSI, channel, and a secure/open flag. Connected STA telemetry reports DHCP IP, RSSI, channel, and a simple percentage/label derived from RSSI. The derivation is a UI heuristic, not a calibrated link-quality estimator. mDNS publication of HTTP service is attempted only while STA is connected and retried at 5 s intervals.

The implementation supports local AP access and LAN/STA access; it does not require internet or a cloud service. It does not implement TLS, per-user authentication, authorization, captive-portal redirection, enterprise Wi-Fi, IPv6 application behavior, PHY instrumentation, or encrypted application payloads. NVS storage is not explicitly encrypted by this firmware. The static AP password and plaintext HTTP API are prototype security limitations.

## 9. Embedded HTTP/API Architecture

The ESP32-S3 is accurately described as an embedded web server, local HTTP API endpoint, telemetry source, and reverse-command receiver. `CommunicationManager` owns an Arduino `WebServer` on TCP port 80 and calls `handleClient()` cooperatively.

| Method | Route | Implemented behavior | Source evidence |
|---|---|---|---|
| GET | `/` | Embedded SPA HTML; `no-cache` | `WebDashboard::registerRoutes()` |
| GET | `/dashboard.css` | Embedded CSS; public max-age 3600 | `WebDashboard::registerRoutes()` |
| GET | `/dashboard.js` | Embedded JavaScript; public max-age 3600 | `WebDashboard::registerRoutes()` |
| GET | `/api/telemetry` | Current JSON telemetry; increments served counter; `no-store, no-cache, must-revalidate` | `CommunicationManager::handleTelemetry()` |
| GET | `/api/ping` | `ok`, node `millis()`, telemetry sequence; `no-store` | `CommunicationManager::handlePing()` |
| POST | `/api/command` | Form action validation and one-slot enqueue; HTTP 202 acceptance | `CommunicationManager::handleCommand()` |
| GET | `/api/wifi/scan` | Starts/polls asynchronous scan JSON | `CommunicationManager::handleWifiScan()` |
| POST | `/api/wifi/connect` | Form SSID/password validation, persistence, association; HTTP 202 | `CommunicationManager::handleWifiConnect()` |
| POST | `/api/wifi/disconnect` | Stops STA retry for boot, retains credentials | `handleWifiDisconnect(false)` |
| POST | `/api/wifi/forget` | Clears credentials and disconnects | `handleWifiDisconnect(true)` |
| Any | Unknown route | JSON 404 and API-error counter increment | `sendApiError()` |

Commands are trimmed, uppercased, and matched to a fixed enum. Missing/unknown actions return 400; an occupied queue returns 409. Missing SSID or invalid credential length returns 400. Errors are compact JSON and counted. JSON is serialized manually with Arduino `String`; string helpers escape quotes/backslashes and omit control characters.

The API has no authentication beyond access to the network, no CSRF protection, no TLS, no rate limiting, and no request body size policy visible at the application layer. It is appropriate for a controlled prototype LAN, not an exposed production system.

## 10. Telemetry Design

`TelemetryManager::buildJson()` composes a current snapshot on each telemetry request. The payload includes:

- node ID/name, logical sequence, uptime, and frame age;
- temperature, humidity, raw/filtered gas, warm-up, flame, per-sensor validity and age;
- radar mode/run/PWM readiness, current and sample angles, direction, distance/status, sample sequence/age, visualization limits, and recent detections;
- authoritative alarm state, mask, causes, summary, and thresholds;
- AP SSID/IP/client count and STA configured/connected/status/SSID/IP/RSSI/channel/quality;
- hostname and mDNS state;
- MCU model, firmware/build string, runtime free heap/free PSRAM, and physical flash size;
- nominal telemetry rate, frames served, accepted-command state, node apply latency, and API errors.

The sequence increments every 250 ms independently of whether a client requests telemetry. It is a logical generation cadence, but no immutable frame object is captured at the increment instant: `buildJson()` reads current subsystem snapshots on demand. Consequently, `seq` and `frame_age_ms` are useful freshness/cadence markers but should not be described as a stored packet log or exact acquisition timestamp for every field. Individual sensor/sample ages provide the more precise data freshness semantics.

Sequence gaps observed by the browser reveal missed logical update intervals at the application consumer, which may result from polling schedule, browser suspension, HTTP failure, server delay, or network effects. They are not direct RF packet-loss measurements. `frames_served` counts completed telemetry handler invocations, not unique generated frames or MAC frames.

Dynamic `String` assembly is simple and readable for the prototype but allocates heap on every request and may become a fragmentation/latency concern under long-duration or multi-client load. There is no persisted telemetry history, wall-clock timestamp, database, push/WebSocket stream, MQTT, or cloud forwarding.

## 11. Reverse Command and Acknowledgement Architecture

The implemented end-to-end path is:

1. `DASHBOARD_JS -> sendCommand()` posts `action=<command>` to `/api/command` and records browser `performance.now()`.
2. `CommunicationManager::handleCommand()` validates the fixed command set, rejects a busy one-slot queue, assigns a monotonically increasing command ID, records node receive time, marks `ACCEPTED`, and returns HTTP 202 JSON.
3. The browser calculates **Command Accept RTT** after the 202 response body is parsed.
4. In the same cooperative loop, after HTTP handling, `src/main.cpp -> applyPendingCommand()` consumes the queued command and calls `RadarManager`.
5. `CommunicationManager::acknowledgeCommand()` records `APPLIED:RADAR_STATE_UPDATED` or a rejection such as `REJECTED:SERVO_NOT_READY`, plus node application time.
6. A later `/api/telemetry` response exposes the same command ID and ACK. The browser matches the ID and calculates **Command Applied RTT** from original send time to observation of `APPLIED` telemetry. Rejections clear the pending browser state and are displayed.
7. Radar state and any subsequent angle/range changes are rendered from telemetry.

Implemented actions are `RADAR_CONTINUOUS`, `RADAR_MANUAL`, `RADAR_START`, `RADAR_PAUSE`, `LOOK_LEFT`, `LOOK_FRONT`, `LOOK_RIGHT`, and `RADAR_SCAN`.

The one-slot queue is not a durable message queue and has no persistence, retry, deduplication, expiry, or idempotency key. Because the server handles requests cooperatively and command application follows immediately in the same loop, normal commands should have small node-side apply latency; this remains an empirical question. `last_command_apply_latency_ms` is emitted only for applied commands. Rejected commands have an ACK but `null` apply latency. Command IDs and millisecond counters can eventually wrap.

## 12. Browser Dashboard Architecture

The dashboard is self-contained in PROGMEM: HTML, CSS, JavaScript, inline SVG icons, and canvas rendering are all served by the node. There are no CDN, font, icon, analytics, or external audio dependencies. It is a four-view SPA (Overview, Environment, Radar, System/Link) using hash-based local navigation.

Telemetry polling runs every 250 ms with a `pollInFlight` guard. Successful receive timestamps support a five-second browser receive-rate estimate. The UI classifies no successful response as connecting, under 2 s as online, 2-4.5 s as stale, and over 4.5 s as offline. Poll errors are intentionally swallowed; elapsed time drives stale/offline visibility.

The Environment view displays values, validity/freshness, thresholds, and semantic icons. The Radar view renders a fixed 150 cm semicircle, rings, current beam, and age-faded recent detections, and exposes all reverse commands. The System view reports node/network/memory/communication state and implements scan/connect/disconnect/forget workflows. A ten-request sequential `/api/ping` test reports application HTTP RTT min/average/max and failed requests.

Safety authority remains in firmware. The browser reads `alarm.active` and `alarm.causes`, renders the banner/icons, and optionally generates a user-enabled two-beep Web Audio warning with vibration. It does not compare measurements to local thresholds. Audio uses guarded 1800/2400 Hz square-wave oscillators at 0.58 gain with gaps and a roughly 900 ms repeat gate. Browser support, autoplay policy, volume, background throttling, and user mute affect audio but not firmware/TFT alarm state.

When `servo_ready=false`, the dashboard labels a radar hardware fault, displays stationary state, disables radar movement controls, and overlays the canvas. This should be interpreted as PWM initialization/readiness failure, not proof of a diagnosed mechanical fault.

## 13. Local TFT Interface

`DisplayManager` initializes the ST7735 with `INITR_BLACKTAB`, rotation 3, and the configured SPI pins. The normal screen shows temperature, humidity, filtered MQ-2 raw value/warm-up, flame, latest radar sample angle/range, AP/STA state, and radar mode/angle/direction.

Normal operation is update-on-change. A static frame is drawn once; each cached field is cleared and redrawn only when its string changes. Eligibility is limited to every 400 ms. Full-screen redraw occurs at initialization, when entering/changing the alarm screen, and once when returning to normal. This is a defensible anti-flicker strategy with lower SPI traffic than unconditional full redraw.

The alarm screen is driven by `SafetyManager` and replaces the normal view with a red cause display while any authoritative alarm remains active. It does not continue rendering live values during the override. The TFT reconstructs short cause labels from centralized cause bits; this is presentation duplication, not duplicate threshold logic. A dedicated PWM/servo readiness fault is not shown on TFT.

## 14. Robustness and Failure-State Handling

| Condition | Actual behavior | Evidence / limitation |
|---|---|---|
| Invalid DHT22 temperature/humidity | Latest validity flag becomes false; last valid numeric value/timestamp retained; telemetry exposes validity and age | `SensorManager::updateDht()` |
| Invalid temperature while alarm active | Active temperature cause remains latched; timing resets until valid input resumes | `SafetyManager::updateTemperature()` |
| MQ-2 warm-up | Values continue; gas alarm is forced inactive for 60 s | `SensorManager::updateFastSensors()`, `SafetyManager::updateGas()` |
| MQ-2 electrical/read failure | No explicit ADC fault diagnostic; each `analogRead()` is marked valid | Limitation in `updateFastSensors()` |
| Flame electrical/read failure | No line-disconnect diagnostic; digital read normally marked valid | Limitation in `updateFastSensors()` |
| Missing HC-SR04 echo | After 25 ms, status `NO_ECHO`, distance invalid/null, no dot | `RadarManager::advanceMeasurement()` |
| Echo rises but does not fall | Status `FAULT` at timeout | `RadarManager::advanceMeasurement()` |
| Echo outside 2-150 cm | Status `OUT_OF_RANGE`, invalid/null, no dot | `advanceMeasurement()`, `finishMeasurement()` |
| LEDC setup/frequency failure | `servo_ready=false`, movement stopped/stationary, movement commands rejected, browser fault | `ServoScanner::begin()`, `RadarManager::setServoFault()` |
| Physical servo disconnected/jammed | Not detected if PWM setup succeeds | No feedback sensor; claim limitation |
| STA unavailable/lost | SoftAP remains; status exposed; automatic 15 s retry unless user disabled reconnect | `NetworkManager::update()` |
| No stored credentials | AP-only startup; STA `NOT_CONFIGURED` | `loadCredentials()`, `begin()` |
| User disconnect | STA disconnected and retry disabled for boot; credentials retained | `disconnectSta(false)` |
| User forget | Preferences cleared, STA disconnected, retry disabled | `disconnectSta(true)` |
| Browser telemetry failure | Failed poll ignored; UI transitions stale after 2 s and offline after 4.5 s | `DASHBOARD_JS -> poll()`, `updateConnection()` |
| Missing command action | HTTP 400 JSON; API-error count incremented | `CommunicationManager::handleCommand()` |
| Unknown command | HTTP 400 `invalid_action` | `parseCommand()` |
| Busy command slot | HTTP 409 `command_queue_busy` | `handleCommand()` |
| Unknown HTTP route | HTTP 404 JSON; API-error count incremented | `server_.onNotFound()` |
| Browser command HTTP failure | UI displays `COMMAND ERROR`; no automatic retry | `DASHBOARD_JS -> sendCommand()` |
| mDNS failure | Retried every 5 s while STA is connected; IP access remains | `NetworkManager::updateMdns()` |

Additional robustness limits include no application watchdog configuration, no persistent event log, no wall-clock synchronization, no authenticated command channel, and no formal soak/concurrency test evidence in the repository. Arduino/ESP32 framework facilities may provide lower-level watchdog/network behavior, but the paper must not attribute unconfigured guarantees to this application.

## 15. Communication Metrics Implemented

| Metric | Implemented? | Exact Meaning | Source Evidence | Suitable Paper Claim |
|---|---|---|---|---|
| Telemetry sequence | YES | Logical counter incremented nominally every 250 ms | `TelemetryManager::update()` | Node provides monotonic logical frame sequencing |
| Frame age | YES | `millis()` elapsed since last logical sequence increment | `TelemetryManager::frameAgeMs()`, `buildJson()` | Current logical telemetry cadence age |
| Nominal telemetry rate | YES | Constant `1000/250 = 4.0 Hz` | `TelemetryManager::buildJson()` | Configured application update rate, not measured throughput |
| Browser receive rate | YES | Successful telemetry response rate over recent 5 s browser timestamps | `DASHBOARD_JS -> browserRate()` | Observed application response cadence |
| Skipped telemetry intervals | YES | Positive gaps in sequence values observed by browser | `DASHBOARD_JS -> poll()` | Missed logical updates at consumer; not RF packet loss |
| Frames served | YES | Number of `/api/telemetry` handlers invoked | `CommunicationManager::handleTelemetry()` | Telemetry HTTP service count |
| Commands received | YES | Valid commands accepted into queue | `handleCommand()` | Accepted application command count |
| Command ID/ACK | YES | Last accepted ID plus ACCEPTED/APPLIED/REJECTED state | `CommunicationStats`, `acknowledgeCommand()` | Correlated two-stage application acknowledgement |
| Command Accept RTT | YES, browser | Browser send through parsed HTTP 202 acceptance response | `DASHBOARD_JS -> sendCommand()` | Application HTTP acceptance RTT |
| Node apply latency | YES | Node `millis()` at APPLIED minus node receive time; null for rejection | `TelemetryManager::buildJson()` | Cooperative command dispatch/application latency |
| Command Applied RTT | YES, browser | Browser send through observing same ID as APPLIED in polled telemetry | `DASHBOARD_JS -> render()` | End-to-end application acknowledgement observation time, including polling |
| Ping RTT | YES, browser | Ten sequential `/api/ping` HTTP round trips; min/mean/max and failures | `DASHBOARD_JS -> runLinkTest()` | Application-layer request RTT sample |
| API errors | YES | Count of application error responses generated via `sendApiError()` | `CommunicationManager::sendApiError()` | Server-side application error count |
| RSSI | YES | ESP32 Wi-Fi stack RSSI for connected STA | `NetworkManager::rssi()` | Stack-reported received signal strength observation |
| Wi-Fi channel/IP/state | YES | Stack association/channel/address state | `NetworkManager` accessors | Network state context |
| Uptime | YES | ESP32 `millis()` value | `TelemetryManager::buildJson()` | Runtime since boot, subject to counter wrap |
| Free heap/free PSRAM | YES | Runtime ESP API values at serialization | `TelemetryManager::buildJson()` | Runtime memory availability snapshots |
| BER | NO | No bit-error instrumentation | No source implementation | Must not claim |
| SNR/noise floor | NO | No SNR/noise measurement | No source implementation | Must not claim |
| RF/MAC packet loss | NO | No lower-layer packet counters | No source implementation | Must not claim |
| PHY/MAC throughput | NO | No byte/time or radio-rate experiment | No source implementation | Must not claim |
| One-way latency | NO | No synchronized clocks | No source implementation | Must not claim |

All implemented timing metrics are application-layer or node-local. None isolates Wi-Fi airtime, TCP retransmission, server processing, browser scheduling, or radio propagation.

## 16. Software-Engineering Strengths

1. **Modular ownership:** state and behavior are divided into cohesive managers with explicit snapshot interfaces.
2. **Orchestration-focused entry point:** `main.cpp` composes modules and routes commands rather than embedding acquisition, rendering, or networking algorithms.
3. **Centralized safety truth:** one alarm mask and state machine drives telemetry, TFT, browser, and logs.
4. **Responsive directional observation:** hardware PWM and interrupt-assisted echo waiting avoid long blocking servo/ultrasonic loops in production.
5. **Correct spatial association:** current servo angle and trigger-time sample angle are distinct.
6. **Bidirectional application protocol:** fixed validated commands, IDs, one-slot admission control, and acceptance/application acknowledgements form a traceable control path.
7. **Structured observability:** identity, sequence, ages, validity, failure status, command state, network context, and memory data are exposed together.
8. **Local-first operation:** embedded assets and AP+STA allow router-independent access plus home-LAN operation without cloud dependency.
9. **Failure-state visibility:** DHT validity, ultrasonic status, Wi-Fi status, stale/offline state, and PWM readiness are explicit rather than silently simulated.
10. **Practical display design:** cached TFT updates reduce redraw traffic and flicker; browser visualization remains lightweight and self-contained.
11. **Configuration discipline:** pins, timings, thresholds, calibration, and N16R8 build overrides are centralized.

## 17. Limitations and Threats to Validity

- No BER, SNR, noise-floor, RF/MAC packet-loss, PHY throughput, or one-way latency measurement exists.
- Command and ping RTTs conflate browser, HTTP/TCP/IP/Wi-Fi, server, and scheduling effects.
- `Command Applied RTT` includes up to the telemetry polling delay and browser scheduling.
- No quantitative experimental dataset, statistical analysis, or automated benchmark is stored in the repository.
- Validation is a single-node prototype; multi-node scaling, contention, and coordinated addressing are not implemented or measured.
- Cooperative scheduling is not hard real time; DHT reads, TFT SPI, HTTP handling, JSON construction, and library calls can introduce jitter.
- Telemetry is HTTP polling at 4 Hz, not push streaming; it has request overhead and browser background throttling risk.
- The logical telemetry sequence is not an immutable captured frame or persisted log.
- Dynamic Arduino `String` construction may create heap fragmentation under prolonged/heavy use; no soak evidence is present.
- MQ-2 data is raw/filtered ADC only; no ppm calibration, cross-sensitivity correction, temperature/humidity compensation, or certified threshold basis exists.
- Flame sensing is a binary module output without line-integrity or intensity diagnostics.
- Ultrasonic conversion uses a fixed speed of sound and is not temperature compensated; acoustic geometry/environment can affect results.
- `servo_ready` is PWM configuration readiness, not physical position/motion feedback.
- No application authentication, TLS, authorization, CSRF protection, or encrypted credential storage is implemented; the AP password is static.
- The one-slot command queue has no durability, retries, deduplication, or concurrency guarantees.
- No explicit application watchdog, persistent fault log, UTC clock, or remote firmware update path is implemented.
- The alarm system is a prototype annunciator, not a certified or redundant safety controller.
- Browser audio/vibration depend on user permission, device/browser support, volume, and foreground scheduling.
- TFT servo-fault presentation is incomplete compared with browser/telemetry fault visibility.
- Wi-Fi results depend on router, client, RF environment, antenna, channel occupancy, and power conditions.
- PlatformIO's generic board banner identifies the base manifest as an 8 MB/no-PSRAM DevKitC variant even though project overrides select 16 MB, `default_16MB.csv`, `qio_opi`, and `BOARD_HAS_PSRAM`. The physical N16R8 capacity is supported by `FINAL_HARDWARE_VALIDATION.md` and should be corroborated by runtime evidence in the paper package, not inferred from the generic banner alone.

## 18. Claims We CAN Safely Make in the Paper

- The implemented ESP32-S3 endpoint integrates environmental acquisition, directional ultrasonic observation, local safety evaluation, embedded visualization, and bidirectional HTTP communication in a modular firmware architecture.
- The production HC-SR04 echo path is interrupt-assisted and timeout-driven and does not use blocking `pulseIn()`.
- The SG90 is controlled by 50 Hz, 14-bit hardware LEDC using verified 700/1500/2300 us calibration for left/front/right.
- Telemetry distinguishes current servo angle from the angle associated with the latest ultrasonic sample.
- The node operates in concurrent SoftAP/station mode, supporting direct local access and infrastructure-LAN access without a cloud dependency.
- The embedded HTTP API provides structured telemetry and a validated reverse command channel with command IDs and separate acceptance/application acknowledgement states.
- Alarm decisions for filtered raw gas threshold, temperature, and flame are centralized in firmware and shared by browser and TFT presentations.
- Telemetry exposes validity, freshness, sequence, radar failure state, network context, communication counters, and runtime memory information.
- The browser dashboard is self-contained and served from the ESP32-S3 with no external asset dependency.
- The measured dashboard latency facilities are application-layer metrics, not PHY-layer measurements.

## 19. Claims We MUST NOT Make

- Node V1 measures or improves BER, SNR, RF packet loss, radio propagation delay, PHY throughput, coding gain, or modulation performance.
- Browser skipped sequence intervals equal Wi-Fi packet loss.
- Command RTT is pure wireless-channel latency or one-way latency.
- MQ-2 readings are ppm, calibrated gas concentration, smoke density, or certified exposure measurements.
- The HC-SR04 is an alarm sensor, collision-avoidance controller, object tracker, mapper, or navigation subsystem.
- The system is a mobile robot or contains motors, drivetrain, L298N, IR obstacle sensing, autonomous movement, or navigation.
- `servo_ready` proves that the SG90 physically moved or reached the commanded angle.
- The cooperative loop is deterministic hard real time or has proven deadlines/WCET.
- The alarm implementation is fail-safe, redundant, certified, or suitable for life-safety deployment.
- The HTTP interface is secure for hostile networks or internet exposure.
- Multi-node PIKU coordination, cloud telemetry, MQTT, AI, camera, audio, STM32 integration, or fleet scalability has been implemented or experimentally validated.
- Recent radar dots represent persistent tracked objects.

## 20. Paper-Ready Software Contributions

### Embedded software engineering

- A cohesive manager-based firmware decomposition with centralized configuration/pin ownership and an orchestration-focused main loop.
- A bounded cooperative acquisition/control design combining hardware PWM with interrupt-assisted ultrasonic timing and explicit validity/failure states.
- Dual local interfaces: update-on-change TFT presentation and a self-contained embedded SPA.

### Communication engineering

- A local HTTP telemetry/control endpoint with structured state, logical sequence/freshness, API counters, and correlated command acknowledgement.
- Concurrent recovery/control SoftAP and infrastructure STA paths, with persistent provisioning and observable link context.
- Built-in application-level RTT and service instrumentation suitable for controlled experiments, while preserving the PHY/application boundary.

### CPS architecture

- Explicit coupling of physical acquisition, software state ownership, centralized safety evaluation, communication representation, human presentation, and remote actuator command.
- Correct separation between moving actuator state and trigger-time sensor observation state.
- Cyber/physical consistency protection that prevents software sweep progression when PWM initialization is unavailable.

### Human-in-the-loop monitoring/control

- A browser interface that combines fresh/invalid/stale status, directional visualization, manual/continuous control, command ACK visibility, provisioning, and optional alarm audio while leaving safety authority on the node.

### Scalability toward future PIKU systems

- `node_id`, modular boundaries, and a structured schema are implementation affordances for future multi-node work. They are not a demonstrated multi-node system. Any scalability contribution should be phrased as an extensible single-node baseline until coordinator, addressing, contention, aggregation, and multi-node experiments exist.

These are implementation contributions. Novelty relative to prior literature requires a separate related-work analysis.

## 21. Paper-Ready Implementation Facts

| Fact | Verified value | Evidence |
|---|---|---|
| Node identity | `NODE-V1`; "Wireless Telemetry & Command Node" | `NodeConfig.h` |
| Production framework | Arduino on Espressif32; clean audit build used platform 7.0.1 and Arduino-ESP32 package 2.0.17 | `platformio.ini`; build output |
| Target project profile | ESP32-S3 N16R8 project configuration | `platformio.ini`; `FINAL_HARDWARE_VALIDATION.md` |
| Flash/PSRAM configuration | 16 MB flash, `default_16MB.csv`, `qio_opi`, `BOARD_HAS_PSRAM`; hardware doc reports 8 MB PSRAM | `platformio.ini`; hardware validation |
| Fast sensor period | 100 ms | `NodeConfig.h` |
| DHT22 period | 2500 ms | `NodeConfig.h` |
| MQ-2 filter | 16-sample moving average of 12-bit ADC values | `SensorManager` |
| MQ-2 warm-up | 60000 ms | `NodeConfig.h` |
| Telemetry logical period/rate | 250 ms / 4 Hz | `TelemetryManager` |
| Browser polling | 250 ms with one in-flight request | `DASHBOARD_JS` |
| TFT period | 400 ms | `DisplayManager` |
| Servo GPIO/PWM | GPIO16, 50 Hz, 14 bit, LEDC channel 0 | `PinConfig.h`, `NodeConfig.h`, `ServoScanner` |
| Servo calibration | 0/90/180 degrees = 700/1500/2300 us; duties 573/1229/1884 | `NodeConfig.h`, `ServoScanner::pulseToDuty()` |
| Sweep | 1 degree every 25 ms; logical 0-180 degrees | `RadarManager::updateSweep()` |
| Continuous ranging | >=70 ms and >=4 degrees; effective nominal about 100 ms | `measurementDue()` |
| Manual ranging | 250 ms settle after movement; 250 ms stationary sampling | `moveManual()`, `updateManual()` |
| Ultrasonic accepted range | 2-150 cm | `NodeConfig.h`, `advanceMeasurement()` |
| Echo timeout | 25000 us | `NodeConfig.h` |
| Detection storage/display | 40-entry ring; 3500 ms telemetry retention | `RadarManager`, `TelemetryManager` |
| Gas alarm/clear | >=2500 for 1500 ms; <=2300 for 500 ms; filtered raw ADC | `SafetyManager`, `NodeConfig.h` |
| Temperature alarm/clear | >=50.0 C for 1000 ms; <=47.0 C for 500 ms | `SafetyManager`, `NodeConfig.h` |
| Flame alarm/clear | active-low for 100 ms; inactive for 500 ms | `SafetyManager`, `NodeConfig.h` |
| Wi-Fi mode | `WIFI_AP_STA`; SoftAP `node_v1`; STA credentials in Preferences | `NetworkManager` |
| STA retry | 15000 ms when configured/reconnect-enabled/disconnected | `NetworkManager::update()` |
| HTTP server | Port 80, embedded assets, telemetry/ping/command/provisioning routes | `CommunicationManager`, `WebDashboard` |
| Command queue | One pending command slot | `CommunicationManager` |
| Command set | 8 radar actions | `CommunicationManager::parseCommand()` |
| Clean build RAM | 49,504 / 327,680 bytes (15.1% internal RAM accounting) | Audit build output |
| Clean build flash | 871,137 / 6,553,600 bytes (13.3% of application partition capacity) | Audit build output |

Relevant external/application libraries resolved in the audit build were DHT sensor library 1.4.7, Adafruit Unified Sensor 1.1.15, Adafruit GFX 1.12.6, Adafruit ST7735/ST7789 1.11.0, and framework SPI, ESPmDNS, WebServer, Preferences, and WiFi 2.0.0.

## 22. Source-to-Claim Traceability Matrix

| Paper Claim / Technical Statement | Supporting File(s) | Supporting Class/Function | Confidence |
|---|---|---|---|
| Main loop is orchestration-focused | `src/main.cpp` | `setup()`, `loop()`, `applyPendingCommand()` | HIGH |
| GPIO mapping is centralized | `include/PinConfig.h` | `PinConfig` constants | HIGH |
| Timing/calibration/thresholds are centralized | `include/NodeConfig.h` | `NodeConfig` constants | HIGH |
| DHT values have validity and last-valid timestamps | `include/SensorManager.h`, `src/SensorManager.cpp` | `SensorSnapshot`, `updateDht()` | HIGH |
| MQ-2 is 12-bit raw ADC with 16-sample moving average | `src/SensorManager.cpp`, `include/NodeConfig.h` | `begin()`, `pushGasSample()` | HIGH |
| Flame input is active-low digital | `src/SensorManager.cpp` | `updateFastSensors()` | HIGH |
| Safety is centralized for gas/temperature/flame | `src/SafetyManager.cpp`, `src/TelemetryManager.cpp`, `src/DisplayManager.cpp` | `SafetyManager::update*()`, consumers | HIGH |
| Radar is excluded from alarm evaluation | `include/SafetyManager.h`, `src/SafetyManager.cpp`, `src/main.cpp` | `SafetyManager::update(const SensorSnapshot&)` | HIGH |
| Servo uses 50 Hz/14-bit LEDC on GPIO16 | `include/NodeConfig.h`, `include/PinConfig.h`, `src/ServoScanner.cpp` | `ServoScanner::begin()` | HIGH |
| PWM readiness is checked and reported | `src/ServoScanner.cpp`, `src/RadarManager.cpp`, `src/TelemetryManager.cpp` | `begin()`, `setServoFault()`, `buildJson()` | HIGH for software readiness; LOW for physical motion |
| HC-SR04 production echo waiting is interrupt-assisted | `src/RadarManager.cpp` | `begin()`, `echoInterrupt()`, `advanceMeasurement()` | HIGH |
| Production firmware does not use `pulseIn()` | `src/`, `include/`; historical use only in `docs/bringup/` | Repository search and production build scope | HIGH |
| Current angle and sample angle are distinct | `include/RadarManager.h`, `src/RadarManager.cpp` | `beginMeasurement()`, `finishMeasurement()` | HIGH |
| Manual moves automatically range and remain live at 250 ms | `src/RadarManager.cpp`, `include/NodeConfig.h` | `moveManual()`, `updateManual()` | HIGH |
| Recent radar points are observations, not tracks | `src/RadarManager.cpp`, `src/TelemetryManager.cpp`, dashboard note | `addDetection()`, `buildJson()` | HIGH |
| Node operates AP+STA concurrently | `src/NetworkManager.cpp` | `NetworkManager::begin()` | HIGH |
| STA credentials persist in NVS Preferences | `src/NetworkManager.cpp` | `loadCredentials()`, `connectSta()` | HIGH |
| SoftAP provides router-independent access | `src/NetworkManager.cpp`, `src/CommunicationManager.cpp` | `begin()` methods | HIGH |
| ESP32 hosts local assets and API | `src/WebDashboard.cpp`, `src/CommunicationManager.cpp` | route registration | HIGH |
| Telemetry has identity, sequence, freshness, subsystem and communication state | `src/TelemetryManager.cpp` | `buildJson()` | HIGH |
| Reverse commands have ID, acceptance, application ACK | `include/CommunicationTypes.h`, `src/CommunicationManager.cpp`, `src/main.cpp` | `handleCommand()`, `acknowledgeCommand()`, `applyPendingCommand()` | HIGH |
| Command Applied RTT includes polling/observation delay | `src/WebDashboardAssets.cpp` | `sendCommand()`, `render()`, `poll()` | HIGH |
| Dashboard is self-contained/offline | `src/WebDashboardAssets.cpp`, `src/WebDashboard.cpp` | PROGMEM assets/routes | HIGH |
| Browser is presentation, not safety authority | `src/SafetyManager.cpp`, `src/TelemetryManager.cpp`, dashboard JS | `updateAlarm()` consumes server state | HIGH |
| TFT uses cached partial redraw and alarm override | `src/DisplayManager.cpp` | `update()`, `updateField()`, `drawAlarmScreen()` | HIGH |
| No motor/navigation implementation exists | `src/`, `include/`, `MIGRATION_NOTES.md` | Production file inventory | HIGH |
| N16R8 memory configuration is selected | `platformio.ini`, `FINAL_HARDWARE_VALIDATION.md` | build overrides / documented hardware test | HIGH for config; MEDIUM from software audit alone for physical capacity |
| Current production release compiles | PlatformIO audit build | `pio run` | HIGH |

## 23. Recommended Experimental Measurements

The smallest useful paper experiment set can use the existing API plus an external browser/test client without changing firmware:

1. **Command timing and success:** issue at least 100 commands per path (direct SoftAP and STA/LAN), record HTTP acceptance RTT, node apply latency, applied RTT, command ID, and final ACK. Report median, mean, standard deviation, 95th percentile, maximum, rejection/failure count, and command success rate.
2. **Telemetry continuity:** poll for at least 30-60 minutes per path and record sequence, frame age, browser receive rate, skipped intervals, HTTP failures, and API errors. Report achieved response rate, age distribution, and interruptions; label these application-level results.
3. **RSSI-stratified observations:** repeat the above at several naturally observed RSSI bands and channels, recording topology/router/client. Treat RSSI as context/correlation, not SNR or causal proof.
4. **Ping baseline:** run repeated ten-request link tests for AP and STA paths and report RTT distributions and failed application requests.
5. **Uptime/resource stability:** operate the integrated node for 12-24 hours while sampling uptime, free heap, free PSRAM, API errors, telemetry continuity, and STA reconnect events. Look for memory decline or resets.
6. **Radar command correctness:** verify requested left/front/right angle, matching `sample_angle`, valid/invalid echo status, and ACK across repeated commands. Physical angle/distance accuracy requires external reference measurements and should be analyzed separately from communication latency.

Do not convert these tests into BER, RF packet-loss, or one-way latency claims. Those require different lower-layer instrumentation and synchronized methods.

## 24. Final Audit Verdict

The current software architecture is coherent and materially stronger than a monolithic sensor/web sketch. State ownership is mostly clear, safety is centralized, physical observation is associated with direction, the reverse command path is traceable, and both local and browser presentations consume shared firmware truth.

It is suitable as the implementation basis for an IEEE-style prototype paper, provided the paper is framed as an embedded software/communication/CPS implementation and is paired with reproducible quantitative experiments. The strongest software contributions are modular subsystem ownership, local-first AP+STA operation, structured freshness/validity telemetry, interrupt-assisted directional ranging, centralized multi-cause alarm state, and correlated application-layer command acknowledgement.

Before publication, the project should experimentally quantify command acceptance/applied timing, telemetry continuity/frame age, application request failures, RSSI context, command success, uptime, and memory stability. Hardware photographs and validation records should substantiate the N16R8 physical configuration and sensor/actuator behavior.

Manuscript wording must explicitly state that MQ-2 values are raw/filtered ADC counts; communication measurements are application-layer; radar dots are recent observations rather than tracks; `servo_ready` is PWM readiness rather than physical feedback; the loop is cooperative rather than hard real time; HC-SR04 does not generate alarms; the prototype is neither a robot nor a certified safety system; and multi-node scalability remains a future design direction rather than a demonstrated capability.
