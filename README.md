# 4DUK Local Network Protocol Specification (Revision 1.3)

This repository contains the official, implementation-agnostic application-level protocol specification for local interaction between smart home devices within the **4DUK** ecosystem. 

The protocol is designed for seamless, high-performance deployment in trusted local area networks (LAN/WLAN) within a single L2 network segment.

## 🚀 Architectural Principles

* **Decentralization (Peer-to-Peer):** All network nodes are equal; data exchange is fully asynchronous without strict master-slave blocking constraints.
* **Event-Driven Model:** Data transmission is instantly triggered by changes in the physical or logical state of the hardware.
* **Broadcast Transport:** All messages are encapsulated in lightweight UDP Broadcast packets, eliminating session maintenance overhead.
* **Passive Registration:** The 4DUK control gateway automatically discovers and registers devices by intercepting their telemetry or keep-alive packets without dedicated discovery polling sequences.

---

## 📡 Network Transport Parameters

All network nodes (gateways, controllers, and end-device execution modules) must configure their sockets according to the following baseline parameters:

| Parameter | Value | Description |
| :--- | :--- | :--- |
| **Protocol** | UDP | User Datagram Protocol |
| **Network Port** | `9009` | Fixed destination port for both transmitting and receiving |
| **Destination Address** | `255.255.255.255` | Limited broadcast address (local subnet) |
| **Data Encoding** | UTF-8 / ASCII | Raw text strings |
| **Topology Requirement** | AP Isolation Disabled | Client isolation must be deactivated on the Wi-Fi router |

---

## 🔤 Packet Format & Syntax

All packets are plain text strings. Elements within a packet are strictly separated by a colon character (`:`). Spaces are **not permitted** unless they are part of the target payload data.

The protocol defines three core types of packets, strictly identified by their first element (prefix marker):
1. `device` — Control command packet (**Gateway ➔ Network**).
2. `devstate` — Status/Telemetry packet (**Device ➔ Network**).
3. `devping` — Service network presence/keep-alive packet (**Device ➔ Network**).

### 1. Control Packet (`device`)
Sent by the central gateway to change the physical or logical state of a target end-device execution module.

* **Syntax:** `device:devname:action:actionname:value:data`
* **Fields:**
  * `device` *(const)*: Control payload marker.
  * `devname` *(string)*: Unique system name of the target device in the local network.
  * `action` *(const)*: Action transmission indicator.
  * `actionname` *(string)*: Technical name of the command/method to be executed (e.g., `power`, `brightness`).
  * `value` *(const)*: Data transmission indicator.
  * `data` *(string)*: The argument or target value of the command (e.g., `on`, `off`, `50`).
* **Example:** `device:relay_kitchen:action:power:value:on`

### 2. Status and Telemetry Packet (`devstate`)
Sent by an end-device to synchronize its current state with the gateway and other listening nodes. Each capability or parameter of the device must be transmitted in a separate individual UDP packet.

* **Syntax:** `devstate:devname:statename:value`
* **Fields:**
  * `devstate` *(const)*: Status/Telemetry packet marker.
  * `devname` *(string)*: System name of the transmitting device.
  * `statename` *(string)*: Technical name of the reported parameter or capability.
  * `value` *(string)*: Current value of the parameter.
* **Example:** `devstate:relay_kitchen:power:on`

### 3. Presence Service Packet (`devping`)
Sent by an end-device to confirm its physical availability, prevent timeout flags, and update network routing tables when no state changes occur.

* **Syntax:** `devping:devname:MAC:IP`
* **Fields:**
  * `devping` *(const)*: Presence/Keep-alive service packet marker.
  * `devname` *(string)*: System name of the transmitting device.
  * `MAC` *(string)*: Physical address of the device's network interface with all colons removed (**12 characters, uppercase**, e.g., `AABBCCDDEEFF`).
  * `IP` *(string)*: Current local IPv4 address assigned to the device.
* **Example:** `devping:relay_kitchen:001A2B3C4D5E:192.168.1.105`

---

## ⏱️ Timers and Device Lifecycle

