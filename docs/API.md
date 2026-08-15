# Embedded HTTP API

Base URL is the currently used node address: normally `http://192.168.4.1` on Soft AP, `http://<STA-IP>`, or `http://node-v1.local` when mDNS is active. All assets and APIs are local; no internet/CDN is required.

## Dashboard assets

| Method | Route | Response |
|---|---|---|
| GET | `/` | Embedded SPA HTML |
| GET | `/dashboard.css` | Embedded CSS |
| GET | `/dashboard.js` | Embedded JavaScript |

## Telemetry

### `GET /api/telemetry`

Returns the current structured telemetry frame as JSON with `Cache-Control: no-store`. Serving it increments `communication.frames_served`.

Top-level fields:

| Field | Type | Meaning |
|---|---|---|
| `node_id` | string | Stable node address/identity (`NODE-V1`) |
| `node_name` | string | Human-readable system role |
| `seq` | integer | Monotonic telemetry-frame sequence |
| `uptime_ms` | integer | Node uptime from `millis()` |
| `frame_age_ms` | integer | Age of the current 4 Hz telemetry frame |
| `environment` | object | Environmental sensor payload |
| `radar` | object | Directional observation state and recent samples |
| `alarm` | object | Authoritative server safety state |
| `thresholds` | object | Current compiled safety thresholds |
| `wifi` | object | AP/STA link state |
| `system` | object | MCU/build/memory information |
| `communication` | object | Application-layer metrics |

`environment` fields:

- `temperature_c`, `humidity_pct`: number or `null` before any valid sample.
- `gas_raw`, `gas_filtered`: 12-bit ADC raw and moving-average values; not ppm.
- `mq2_warming_up`: gas alarm gating state.
- `flame`: active-low digital detection state.
- `validity.temperature`, `.humidity`, `.gas`, `.flame`: whether the latest acquisition was valid.
- `age_ms.temperature`, `.humidity`, `.gas`, `.flame`: age of the last valid/update timestamp, or `null` if unavailable.

`radar` fields:

- `mode`: `CONTINUOUS` or `MANUAL`.
- `running`: whether continuous movement is active.
- `servo_ready`: whether the 50 Hz servo PWM channel initialized and passed frequency validation.
- `angle`: current logical 0-180 degree head angle (0 left, 90 front, 180 right).
- `sample_angle`: logical angle captured when the latest ultrasonic measurement was triggered; `distance_cm` belongs to this angle, not necessarily the newer moving `angle`.
- `direction`: `LEFT_TO_RIGHT`, `RIGHT_TO_LEFT`, or `STATIONARY`.
- `distance_cm`: latest valid 2-150 cm range or `null`.
- `distance_valid`: whether `distance_cm` represents a valid in-range echo.
- `ultrasonic_status`: `VALID`, `NO_ECHO`, `OUT_OF_RANGE`, or `FAULT`.
- `sample_seq`: monotonic HC-SR04 measurement completion count.
- `sample_age_ms`: last measurement age or `null`.
- `max_range_cm`: visualization/filter radius (150 cm).
- `dot_persistence_ms`: recent-measurement retention (3500 ms).
- `detections`: arrays in `[angle_deg, distance_cm, age_ms]` form. These are recent measurements, not tracked objects.

`alarm` fields:

- `active`: true when any centralized alarm cause is active.
- `state`: `ALARM` or `NORMAL`.
- `cause_mask`: bit mask (`1` high gas, `2` high temperature, `4` flame).
- `causes`: zero or more of `HIGH_GAS`, `HIGH_TEMPERATURE`, `FLAME`.
- `summary`: display-ready combined cause text.

`thresholds` fields:

- `gas_alarm_raw`, `gas_clear_raw`.
- `temperature_alarm_c`, `temperature_clear_c`.

`wifi` fields:

- `ap.ssid`, `ap.ip`, `ap.clients`.
- `sta.configured`, `sta.connected`, `sta.status`, `sta.ssid`, `sta.ip`.
- `sta.rssi`: dBm when connected, otherwise 0.
- `sta.channel`: associated channel when connected, otherwise 0.
- `sta.quality_pct`, `sta.quality`: derived display mapping from RSSI (`EXCELLENT`, `GOOD`, `FAIR`, `WEAK`, or `UNAVAILABLE`).
- `hostname`, `mdns_active`, `mdns_url`.

`system` fields:

- `mcu`, `firmware`, `build`.
- `free_heap`, `free_psram`, `flash_bytes` in bytes.

`communication` fields:

- `telemetry_hz`: nominal node-frame update rate.
- `frames_served`: completed telemetry responses.
- `commands_received`: accepted command count.
- `last_command_id`, `last_command`, `last_ack`, `last_command_age_ms`.
- `last_command_apply_latency_ms`: node-side `APPLIED` time minus command-receive time, or `null` until an applied command exists.
- `api_errors`: invalid route/request/queue-busy responses.

The browser separately reports **Command Accept RTT** (POST through HTTP 202), **Node Apply Latency** (the firmware field above), and **Command Applied RTT** (browser send time through observing the same command ID with an `APPLIED` telemetry acknowledgement). Missing values are displayed as `--` rather than estimated.

## Ping / link test

### `GET /api/ping`

Response:

```json
{"ok":true,"node_ms":837214,"telemetry_seq":1548}
```

The browser runs 10 sequential requests and reports sample count, average/minimum/maximum RTT, and failed application requests. This is an **application-layer link test**, not BER or RF packet-loss measurement.

## Radar commands

### `POST /api/command`

Content type: `application/x-www-form-urlencoded`.

Parameter: `action=<command>`.

| Action | Result |
|---|---|
| `RADAR_CONTINUOUS` | Select continuous mode and start sweeping |
| `RADAR_MANUAL` | Select manual mode and stop sweeping |
| `RADAR_START` | Select/start continuous mode |
| `RADAR_PAUSE` | Pause an active continuous sweep |
| `LOOK_LEFT` | Manual position 0 degrees, settle, then measure |
| `LOOK_FRONT` | Manual position 90 degrees, settle, then measure |
| `LOOK_RIGHT` | Manual position 180 degrees, settle, then measure |
| `RADAR_SCAN` | Manual measurement at the current angle |

Acceptance response (HTTP 202):

```json
{"ok":true,"command_id":17,"action":"LOOK_FRONT","ack":"ACCEPTED","node_ms":837214}
```

`ACCEPTED` means validated and placed in the single-slot command queue; its HTTP timing is Command Accept RTT only. The main loop applies it and changes telemetry `last_ack` to `APPLIED:RADAR_STATE_UPDATED`. The browser waits for that same command ID before calculating Command Applied RTT. A request made while the slot is still occupied receives HTTP 409 `command_queue_busy`.

## Wi-Fi control

### `GET /api/wifi/scan`

Starts/polls an asynchronous scan. While active:

```json
{"running":true,"networks":[]}
```

Completed response contains `networks[]` entries with `ssid`, `rssi`, `channel`, and `secure`. Passwords are never present.

### `POST /api/wifi/connect`

Form parameters: required `ssid`, optional `password`. Valid credentials are stored in NVS and association begins; response is HTTP 202 with status `CONNECTING`. Submitting the already saved SSID with an empty password reuses the saved password without exposing it.

### `POST /api/wifi/disconnect`

Disconnects STA and suspends automatic STA reconnect for this boot. Saved credentials remain. Soft AP stays active.

### `POST /api/wifi/forget`

Disconnects STA, clears SSID/password from NVS, and leaves Soft AP active.

## Errors

API errors use JSON:

```json
{"ok":false,"error":"invalid_action"}
```

Relevant codes are `missing_action`, `invalid_action`, `command_queue_busy`, `missing_ssid`, `invalid_credentials_format`, and `not_found`. They increment `communication.api_errors`.
