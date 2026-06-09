# 🛡️ CEH — IoT & OT Attack Techniques
### EC-Council | Certified Ethical Hacker (CEH) Lab Documentation

![CEH](https://img.shields.io/badge/EC--Council-CEH-red?style=for-the-badge&logo=security&logoColor=white)
![IoT](https://img.shields.io/badge/IoT-Attack-orange?style=for-the-badge)
![OT](https://img.shields.io/badge/OT-Security-blue?style=for-the-badge)
![License](https://img.shields.io/badge/License-Educational-green?style=for-the-badge)

> ⚠️ **Disclaimer:** This documentation is strictly for **educational purposes** under the EC-Council CEH certification program. All techniques described here must only be performed in **authorized lab environments**. Unauthorized use against real systems is **illegal and unethical**.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Lab Environment](#lab-environment)
- [Module 1 — Gather Information using Online Footprinting Tools](#module-1--gather-information-using-online-footprinting-tools)
- [Module 2 — Capture and Analyze IoT Traffic using Wireshark](#module-2--capture-and-analyze-iot-traffic-using-wireshark)
- [Module 3 — Perform Replay Attack on CAN Protocol](#module-3--perform-replay-attack-on-can-protocol)
- [Key Findings Summary](#key-findings-summary)
- [Tools Used](#tools-used)
- [References](#references)

---

## Overview

This repository documents **IoT and OT (Operational Technology) attack techniques** covered in the **EC-Council CEH** certification lab modules. The labs cover three core areas of IoT/OT security:

1. **Online Footprinting** — Discovering IoT devices exposed on the internet
2. **IoT Traffic Analysis** — Capturing and analyzing MQTT protocol traffic
3. **CAN Protocol Replay Attack** — Exploiting automotive networks

These exercises demonstrate the real-world attack surface of IoT and OT environments and help ethical hackers understand how to identify, analyze, and exploit vulnerabilities in connected device ecosystems.

---

## Lab Environment

| Component | Details |
|---|---|
| **OS** | Windows 10 / Kali Linux / Ubuntu |
| **Network** | Isolated Lab Network |
| **Tools** | Shodan, Wireshark, Bevywise MQTT Route, Bevywise IoT Simulator, ICSim, candump, canreplay |
| **Protocol Focus** | MQTT, TCP/IP, CAN Bus |

---

## Module 1 — Gather Information using Online Footprinting Tools

### 🎯 Objective
Use online open-source intelligence (OSINT) tools to discover IoT devices exposed on the internet, identify services, ports, and potential vulnerabilities without directly interacting with the target systems.

---

### 1.1 Shodan — IoT Search Engine

**Shodan** is a powerful search engine that indexes internet-connected devices, making it a critical tool for IoT footprinting.

#### 🔍 Searching for MQTT-Enabled Devices

MQTT (Message Queuing Telemetry Transport) is one of the most widely used IoT protocols. To find exposed MQTT brokers:

**Step 1:** Navigate to [https://www.shodan.io](https://www.shodan.io)

**Step 2:** Enter the following search query in the search field:

```
port:1883
```

**Step 3:** Analyze the results — Shodan will return a list of IP addresses running MQTT brokers on port 1883.

#### 📌 Key Answer

| Query | Port | Protocol |
|---|---|---|
| `port:1883` | **1883** | MQTT (Unencrypted) |
| `port:8883` | 8883 | MQTT over TLS/SSL |

> ✅ **Answer:** The port number entered in the Shodan search field to find MQTT-enabled devices is **`1883`**

#### 📊 What You Will Find

When searching `port:1883` on Shodan, results typically reveal:
- IP addresses of exposed MQTT brokers
- Geographic location of devices
- ISP and organization information
- Device banners and version information
- Open/unauthenticated MQTT brokers (critical vulnerability)

---

### 1.2 Wireless Network Footprinting

During the CEH lab, the target wireless network **`CEH_FINANCE_NETWORK`** is identified using wireless footprinting tools.

#### Tools Used
- **Wigle.net** — Online wireless network database
- **inSSIDer** — Wi-Fi scanner
- **NetSurveyor** — SSID detection tool

#### Target SSID Details

| Field | Value |
|---|---|
| **SSID** | `CEH_FINANCE_NETWORK` |
| **Encryption** | WPA2 |
| **Purpose** | Target network for CEH wireless footprinting lab |

---

### 1.3 WebSocket Port — Bevywise MQTT IoT Simulator

When using **Bevywise MQTT IoT Simulator** to establish a WebSocket connection with **Bevywise MQTT Route**:

| Connection Type | Default Port |
|---|---|
| TCP (MQTT) | **1883** |
| WebSocket | **8083** |
| Secure MQTT (TLS) | 8883 |

---

## Module 2 — Capture and Analyze IoT Traffic using Wireshark

### 🎯 Objective
Use **Wireshark**, **Bevywise MQTT Route** (broker), and **Bevywise IoT Simulator** (client) to capture, inspect, and analyze IoT MQTT traffic in real time.

---

### 2.1 Lab Setup

```
┌─────────────────────┐         MQTT TCP         ┌──────────────────────┐
│  Bevywise IoT       │  ──────────────────────►  │  Bevywise MQTT       │
│  Simulator (Client) │       Port: 1883           │  Route (Broker)      │
└─────────────────────┘                            └──────────────────────┘
           │                                                  │
           └──────────────── Wireshark ───────────────────────┘
                         (Packet Capture)
```

### 2.2 Default Ports

| Component | Role | Default TCP Port |
|---|---|---|
| **Bevywise MQTT Route** | Broker | **1883** |
| **Bevywise IoT Simulator** | Client | Connects to 1883 |
| **WebSocket Connection** | WebSocket | **8083** |

> ✅ **Answer:** The default TCP port used by Bevywise MQTT Route to connect with Bevywise IoT Simulator is **`1883`**

---

### 2.3 Step-by-Step: Capturing MQTT Traffic

**Step 1:** Launch **Bevywise MQTT Route** (broker starts listening on port 1883)

**Step 2:** Open **Wireshark** and select the active network interface

**Step 3:** Apply the capture filter:
```
tcp.port == 1883
```

**Step 4:** Launch **Bevywise IoT Simulator** and connect to the broker

**Step 5:** Observe MQTT packets in Wireshark

---

### 2.4 MQTT Packet Types to Observe

| MQTT Packet | Description |
|---|---|
| `CONNECT` | Client initiates connection to broker |
| `CONNACK` | Broker acknowledges connection |
| `PUBLISH` | Client/broker sends a message |
| `SUBSCRIBE` | Client subscribes to a topic |
| `SUBACK` | Broker confirms subscription |
| `PINGREQ / PINGRESP` | Keepalive heartbeat |
| `DISCONNECT` | Client disconnects from broker |

---

### 2.5 Wireshark Display Filters for MQTT Analysis

```bash
# Filter all MQTT traffic
mqtt

# Filter by specific TCP port
tcp.port == 1883

# Filter MQTT CONNECT packets only
mqtt.msgtype == 1

# Filter MQTT PUBLISH packets only
mqtt.msgtype == 3

# Filter by topic name
mqtt.topic contains "sensor"
```

---

### 2.6 Sample MQTT Traffic Analysis

When Wireshark captures the MQTT session, you will observe:

```
No.   Time      Source          Destination     Protocol  Info
1     0.000     192.168.1.10    192.168.1.1     TCP       SYN → Port 1883
2     0.001     192.168.1.1     192.168.1.10    TCP       SYN-ACK
3     0.002     192.168.1.10    192.168.1.1     MQTT      CONNECT
4     0.003     192.168.1.1     192.168.1.10    MQTT      CONNACK (Return Code: 0 — Accepted)
5     0.100     192.168.1.10    192.168.1.1     MQTT      PUBLISH [Topic: sensors/temp]
6     0.200     192.168.1.10    192.168.1.1     MQTT      SUBSCRIBE [Topic: commands/#]
```

---

## Module 3 — Perform Replay Attack on CAN Protocol

### 🎯 Objective
Use **ICSim (Instrument Cluster Simulator)** on Ubuntu to simulate a CAN bus network, sniff CAN traffic, and perform a **replay attack** by replaying captured log files to manipulate vehicle behavior.

---

### 3.1 What is CAN Protocol?

**Controller Area Network (CAN)** is a vehicle bus standard allowing microcontrollers and devices to communicate within a vehicle without a host computer. It is used in:
- Engine control units (ECUs)
- ABS braking systems
- Dashboard instrument clusters
- Door locking systems
- Transmission control

> ⚠️ CAN protocol has **no authentication or encryption**, making it highly vulnerable to replay attacks.

---

### 3.2 Lab Setup — ICSim on Ubuntu

#### Prerequisites

```bash
# Install required dependencies
sudo apt-get update
sudo apt-get install -y can-utils libsdl2-dev libsdl2-image-dev git

# Clone ICSim repository
git clone https://github.com/zombieCraig/ICSim.git
cd ICSim

# Compile ICSim
make
```

#### Setup Virtual CAN Interface

```bash
# Load the vcan kernel module
sudo modprobe vcan

# Create virtual CAN interface
sudo ip link add dev vcan0 type vcan

# Bring the interface up
sudo ip link set up vcan0

# Verify interface is active
ip link show vcan0
```

> ✅ **Answer:** The interface used while sniffing CAN traffic in Ubuntu is **`vcan0`**

---

### 3.3 Step-by-Step: CAN Traffic Sniffing & Replay Attack

#### Step 1 — Launch ICSim

```bash
# Start the instrument cluster simulator on vcan0
./icsim vcan0

# In a separate terminal, start the controls
./controls vcan0
```

#### Step 2 — Start CAN Sniffer (candump)

```bash
# Sniff and log all CAN traffic on vcan0
candump -l vcan0
```

This creates a timestamped log file: `candump-YYYY-MM-DD_HHMMSS.log`

#### Step 3 — Generate CAN Traffic

Using the ICSim controls, perform the following actions to generate traffic:

| Action | CAN Frames Generated |
|---|---|
| 🚗 **Accelerate** | Engine RPM frames |
| ⬅️ **Turn Left** | Steering angle frames |
| ➡️ **Turn Right** | Steering angle frames |
| 🔓 **Unlock Doors** | Door control frames |
| 🔒 **Lock Doors** | Door control frames |

#### Step 4 — Perform Replay Attack

```bash
# Replay the captured CAN log file on vcan0
canreplay -i vcan0 candump-2024-01-01_120000.log
```

Watch the ICSim dashboard — the **doors will unlock/lock**, **speedometer will move**, and **indicators will activate** — demonstrating a successful replay attack.

---

### 3.4 CAN Replay Attack — Explained

```
PHASE 1: SNIFFING
─────────────────
Attacker listens on vcan0
candump captures all CAN frames with timestamps

PHASE 2: ANALYSIS  
──────────────────
Attacker identifies CAN IDs responsible for:
- Door lock/unlock → CAN ID: 0x19B
- Acceleration     → CAN ID: 0x244
- Turn signals     → CAN ID: 0x188

PHASE 3: REPLAY
───────────────
Attacker replays captured frames using canreplay
Vehicle responds as if original commands were sent
No authentication = successful attack
```

---

### 3.5 CAN Frame Structure

```
┌──────────┬───────────┬─────────────────────────────┐
│  CAN ID  │  DLC      │         Data (Payload)       │
│ (11-bit) │ (4-bit)   │         (0-8 bytes)          │
├──────────┼───────────┼─────────────────────────────┤
│  0x19B   │    8      │  00 00 00 00 00 00 00 00     │
└──────────┴───────────┴─────────────────────────────┘
```

---

### 3.6 Key CAN Tools Reference

| Tool | Command | Purpose |
|---|---|---|
| `candump` | `candump -l vcan0` | Sniff and log CAN traffic |
| `canreplay` | `canreplay -i vcan0 file.log` | Replay captured CAN frames |
| `cansend` | `cansend vcan0 19B#0000` | Send single CAN frame |
| `cangen` | `cangen vcan0` | Generate random CAN traffic |
| `cananalyzer` | `cananalyzer vcan0` | Analyze CAN bus traffic |

---

## Key Findings Summary

| Module | Topic | Key Answer |
|---|---|---|
| **Module 1** | MQTT port for Shodan search | **1883** |
| **Module 1** | Target wireless SSID | **CEH_FINANCE_NETWORK** |
| **Module 1** | Bevywise WebSocket port | **8083** |
| **Module 2** | Default TCP port — Bevywise MQTT Route ↔ IoT Simulator | **1883** |
| **Module 3** | Virtual CAN interface for sniffing | **vcan0** |

---

## Tools Used

| Tool | Category | Purpose |
|---|---|---|
| [Shodan](https://www.shodan.io) | OSINT | IoT device discovery |
| [Wireshark](https://www.wireshark.org) | Traffic Analysis | Packet capture & analysis |
| [Bevywise MQTT Route](https://www.bevywise.com) | IoT Broker | MQTT broker for testing |
| [Bevywise IoT Simulator](https://www.bevywise.com) | IoT Client | Simulates IoT device |
| [ICSim](https://github.com/zombieCraig/ICSim) | OT Simulation | CAN bus simulator |
| [can-utils](https://github.com/linux-can/can-utils) | OT Tools | CAN sniffing & replay |
| [Wigle.net](https://www.wigle.net) | OSINT | Wireless network mapping |

---

## References

- [EC-Council CEH Official Page](https://www.eccouncil.org/programs/certified-ethical-hacker-ceh/)
- [MQTT Protocol Specification](https://mqtt.org/mqtt-specification/)
- [Shodan Documentation](https://help.shodan.io)
- [ICSim GitHub Repository](https://github.com/zombieCraig/ICSim)
- [can-utils Documentation](https://github.com/linux-can/can-utils)
- [Bevywise MQTT Route](https://www.bevywise.com/mqtt-broker/)
- [NIST IoT Security Guidelines](https://www.nist.gov/programs-projects/nist-cybersecurity-iot-program)

---

## ⚖️ Legal & Ethical Notice

> This repository is created **solely for educational purposes** as part of the **EC-Council CEH** certification curriculum. All lab exercises must be performed **only in authorized, isolated lab environments** provided by EC-Council or equivalent controlled setups.
>
> **Never** apply these techniques on production systems, public networks, or any system without explicit written authorization. Unauthorized access to computer systems is a **criminal offense** under laws such as the Computer Fraud and Abuse Act (CFAA) and equivalent legislation worldwide.

---

<div align="center">

**Made for CEH Students | EC-Council Certified Ethical Hacker**

![EC-Council](https://img.shields.io/badge/EC--Council-Certified-red?style=flat-square)
![IoT Security](https://img.shields.io/badge/IoT-Security-orange?style=flat-square)
![OT Security](https://img.shields.io/badge/OT-Security-blue?style=flat-square)
![Educational](https://img.shields.io/badge/Purpose-Educational-green?style=flat-square)

</div>
