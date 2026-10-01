# Communication architecture - Stage 5A standalone

Node V1 is a fixed environmental endpoint. Wi-Fi exposes its local dashboard and telemetry; nRF24 monitors compatible PIKU peer evidence. Firmware/API details belong to the [frozen software release](SOFTWARE_RELEASE_REFERENCE.md).

![Current communication architecture](../figures/communication_diagram.png)

*Sensors -> ESP32-S3 safety/telemetry -> TFT and browser. PIKU heartbeat/status -> Node V1 peer monitoring, with RF hardware auto-ACK at the link layer.*

Diagram screen values and labels are illustrative. Use the live TFT for addresses and the software API authority for exact state/field names; the centralized alarm API state is NORMAL or ALARM.

## Sensors to Node V1

DHT22 temperature/humidity, MQ-2 analog data and digital flame status enter `SensorManager`. `SafetyManager` owns alarm evaluation; telemetry, TFT and dashboard present that shared state. MQ-2 values are raw/filtered ADC, not ppm. Validity and age information accompany observations.

## TFT

The 1.8-inch ST7735S cycles normal telemetry and network pages. Alarm presentation takes priority over both. It shows the actual connected STA SSID and current STA IP, separate SoftAP SSID/IP, and local/peer radio state. During STA loss it marks the STA address unavailable; recovery displays the current IP rather than an old router address.

## Wi-Fi and browser access

| Path | Identity/address | Use |
|---|---|---|
| SoftAP | `IUB-IRCPS`, `192.168.10.1` | Direct local dashboard and provisioning access |
| STA | Actual connected SSID, current DHCP IP shown on TFT | Dashboard access from the same router network |
| Optional STA mDNS | `node-v1.local` when active and supported by the client network | Alternate local browser address |

Open the root dashboard using the TFT-displayed STA address, or join the node's SoftAP with owner-provided provisioning information and open `http://192.168.10.1/`. Both interfaces serve the same embedded dashboard: Overview, Environment, Communication and System. Assets are local and need no internet service. SoftAP remains available during STA connection/loss/recovery.

A submitted Wi-Fi configuration is a temporary **candidate**. It becomes **last known good (LKG)** only after continuous usable association/IP confirmation and a successful checked storage write. Candidate failure restores the previous LKG when available; without LKG, SoftAP remains the provisioning path. Disconnect retains LKG and suppresses retries; Forget attempts to erase saved configuration. These are source-defined policies, not measured reconnection guarantees. See the [frozen communication document](https://github.com/hhh-habib/CPS-Node-V1-Software-Development/blob/70ef949ce18ab81cbc207fed13bcc320d8b99964/docs/COMMUNICATION_ARCHITECTURE.md) for exact conditions and errors.

The prototype serves plaintext local HTTP without application authentication/TLS. It is not a secured public deployment. Provisioning information is supplied separately by the owner.

## nRF peer monitoring

The nRF24L01+ PA+LNA uses the regulated adapter and shared SPI described in [hardware architecture](HARDWARE_ARCHITECTURE.md). PIKU heartbeat/status packets provide recognized peer evidence.

| State | Meaning |
|---|---|
| Local READY | Node V1 radio hardware initialized/configured and listener started |
| Local ERROR | Local hardware/configuration unavailable; recovery is paced |
| Peer UNSEEN | No recognized PIKU peer evidence yet |
| Peer ONLINE | Initialized local radio and fresh recognized PIKU heartbeat/status evidence |
| Peer STALE | Previously seen evidence expired or local radio became unavailable |

READY does not prove ONLINE. The source's peer freshness window is 5 s, a policy constant rather than RF latency. Initialization/health failures enter bounded, nonfatal recovery so sensing, safety, TFT, Wi-Fi and HTTP continue getting loop turns. This software behavior does not repair broken wires or guarantee survival of electrical damage.

The standalone loop listens and does not periodically send application heartbeats or application-level robot drive commands. **RF hardware auto-ACK is link-layer behavior**, distinct from an application heartbeat, acknowledgement packet or robot control. Owner-reported mutual peer visibility must not be reinterpreted as an implemented Node V1 application transmit path.

The owner reports Node V1 seeing PIKU ONLINE, PIKU seeing Node V1 online, and automatic reacquisition after PIKU reboot. Two defective nRF jumper wires on PIKU caused the initial link failure; replacement restored communication. See [physical validation](STAGE5A_PHYSICAL_VALIDATION.md).

## Retired historical and future interfaces

The former radar/head-control API, including `/api/command`, is **retired historical behavior**. No current Radar view or directional command path remains.

**Future Stage 5B:** coordinator integration and application robot driving, followed by the combined CPS dashboard. The future `/cps` route is absent from standalone Stage 5A. Robot 2 integration is future work; no Robot 2 nRF link is claimed. See the current/future platform context in the [README](../README.md).
