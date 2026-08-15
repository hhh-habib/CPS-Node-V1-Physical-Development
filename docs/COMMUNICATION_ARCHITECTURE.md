# Communication Architecture

## System role

Node V1 is an embedded application-layer endpoint in a bidirectional Wi-Fi communication system. It is simultaneously an information-source encoder/server and a remote-command receiver/physical actuator controller.

```text
INFORMATION SOURCES
  DHT22 | MQ-2 | Flame | Directional HC-SR04
                         |
                         v
ACQUISITION / DIGITALIZATION
  Digital sensor protocol | ADC | GPIO | echo timing
                         |
                         v
ESP32-S3 PROCESSING
  filtering | validity | freshness | safety | radar association
                         |
                         v
DIGITAL TELEMETRY
  node ID | sequence | timestamp | structured JSON | metrics
                         |
                         v
EMBEDDED HTTP COMMUNICATION SERVER
                         |
                         v
WI-FI WIRELESS CHANNEL (AP or STA path)
                         |
                         v
REMOTE BROWSER RECEIVER / DASHBOARD
```

Reverse direction:

```text
REMOTE DASHBOARD COMMAND
  mode | start/pause | left/front/right | measure
                         |
                         v
HTTP POST + APPLICATION ACK
                         |
                         v
WI-FI CHANNEL
                         |
                         v
ESP32-S3 COMMAND RECEIVER / QUEUE
                         |
                         v
RADAR STATE MACHINE + LEDC SERVO GENERATOR
                         |
                         v
PHYSICAL ACTION: SG90 DIRECTIONAL SENSOR HEAD
```

## Layer boundary

### Firmware/application responsibilities

- Acquire, filter, validate, timestamp, and package sensor data.
- Assign node identity and monotonically increasing telemetry sequence.
- Serve the embedded SPA and JSON API.
- Parse, validate, count, queue, and acknowledge commands.
- Report application telemetry age/rate, frames served, Command Accept RTT, node apply latency, Command Applied RTT, command state, and API errors.
- Report Wi-Fi API observations exposed by the ESP32 stack: RSSI, channel, IP, and association state.
- Maintain AP+STA recovery and persistent credentials.

### Wi-Fi/lower-layer responsibilities

- 802.11 modulation, coding, framing, retransmission, channel access, and PHY/MAC error handling.
- RF transport between ESP32-S3 and browser/access point.
- IP, TCP, and link-layer delivery below the application server.

The firmware does not derive BER, SNR, RF packet loss, or PHY throughput. Sequence intervals not rendered by the browser are labelled **skipped telemetry intervals (browser)**, not packet loss. The link test reports HTTP request round-trip time and failed application requests.

## AP+STA topology

```text
                         +-------------------------------+
Phone/Laptop <---Wi-Fi-->| Soft AP: node_v1              |
direct recovery/control  | IP: normally 192.168.4.1      |
                         |                               |
Home AP <-------Wi-Fi--->| STA: saved credentials in NVS |<---LAN---> Browser
                         | IP: DHCP                      |
                         +-------------------------------+
                                      ESP32-S3
```

The ESP32 stays in `WIFI_AP_STA`. A failed STA association does not remove the local AP path. On a successful STA association it also attempts `node-v1.local` mDNS service publication.

## Runtime data ownership

- `SensorManager` owns environmental samples and their validity/freshness timestamps.
- `RadarManager` owns servo direction, current beam angle, latest measurement angle/range, and recent detections. Current `angle` and latest `sample_angle` are intentionally distinct while sweeping.
- `SafetyManager` alone converts sensor truth into alarm causes.
- `NetworkManager` owns Wi-Fi credentials and connection/link state.
- `TelemetryManager` produces the sequenced transmission representation.
- `CommunicationManager` owns API counters, command acceptance, and acknowledgements.
- Browser JavaScript renders node truth; it does not duplicate safety thresholds.

## Timing

| Activity | Nominal timing |
|---|---:|
| Browser telemetry polling | 250 ms, with one request in flight at a time |
| Node telemetry sequence update | 250 ms (4 Hz) |
| Fast environment inputs | 100 ms |
| DHT22 read | 2500 ms |
| Radar servo step | 1 degree / 25 ms |
| Radar sample spacing | >= 70 ms and about 4 degrees |
| Manual servo settle | 250 ms |
| HC-SR04 echo timeout | 25 ms |
| TFT update eligibility | 400 ms |
| Radar-dot retention | 3500 ms |

All schedules are rollover-safe unsigned elapsed-time comparisons. HC-SR04 trigger/echo and servo generation are non-blocking with respect to HTTP servicing.

## Extensibility

The schema starts with `node_id`, and transport, telemetry, acquisition, radar, and safety are separate components. Reserved GPIOs remain unused for future STM32, I2C, camera, audio, or multi-node CPS work. None of those future modules is implemented in this version.
