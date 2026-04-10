<div align="center">

# 💬 LoRa Chat
### Secure Peer-to-Peer and Broadcast Messaging over LoRa

_A robust, secure messaging application enabling encrypted communication between multiple nodes using RYLR998 LoRa modules. Features real-time chat in broadcast (group chat) or direct (private chat) via a web interface, multi-hop relay capabilities, and automated network discovery._

[![Last Commit](https://img.shields.io/badge/last%20commit-today-brightgreen)]()
[![Languages](https://img.shields.io/badge/languages-3-blue)]()
[![Python](https://img.shields.io/badge/Python-3.12%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.0.0-green?logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![Socket.IO](https://img.shields.io/badge/Socket.IO-Real--Time-black?logo=socket.io&logoColor=white)](https://socket.io/)
</div>

---

## 📚 Table of Contents

- [✨ Features](#features)
- [🏗️ Project Structure](#project-structure)
- [⚙️ Setup Instructions](#setup-instructions)
  - [Hardware Requirements](#hardware-requirements)
  - [Software Requirements](#software-requirements)
- [🚀 Getting Started](#getting-started)
- [🔐 Security](#security)
- [📡 Protocol Deep Dive](#protocol-deep-dive)
  - [Packet Structure](#packet-structure)
  - [Address Negotiation (Training)](#address-negotiation)
  - [Direct Messaging & Reliability](#direct-messaging)
  - [Relay & Mesh Routing](#relay-routing)
- [❓ Troubleshooting](#troubleshooting)
- [👥 Authors](#authors)

---

<a id="features"></a>
## ✨ Features

- 🔒 **Symmetric AES Encryption**: Secure message payloads using Fernet encryption.
- 🛡️ **Message Integrity**: HMAC-SHA256 ensures messages are authentic and unaltered.
- 💬 **Web-Based UI**: Clean interface built with Flask and SocketIO for real-time messaging.
- 🤖 **Address Negotiation**: Automated training phase for dynamic device address assignment.
- 🔄 **Relay Mechanism**: Multi-hop message forwarding (mesh-like) when direct communication fails.
- 🤝 **CTS/RTS Protocol**: Reliable delivery with Clear-To-Send / Request-To-Send handshakes.
- 🧵 **Threaded Architecture**: Efficient concurrency for handling serial I/O and UI events.

<a id="project-structure"></a>
## 🏗️ Project Structure

```bash
├── app.py               # 🌐 Flask + SocketIO server, web routing & background listener
├── Comm.py              # 🔌 Handles serial port I/O and LoRa AT command setup
├── Message.py           # 📦 Handles message creation, parsing, encryption, HMAC
├── Messenger.py         # 🧠 Core logic manager, orchestration of components
├── Relay.py             # 📡 RelayManager implementation for multi-hop messaging
├── main.py              # 💻 Simple terminal-based test runner
├── encryption_key.py    # 🔑 Pre-shared Fernet key and HMAC key
├── requirements.txt     # 📜 Python dependencies
├── Protocols/
│   ├── DirectMessage.py # 📨 Handles direct messages with retries and CTS logic
│   ├── HostsTracker.py  # 📝 Tracks known LoRa device addresses
│   └── Training.py      # 🎓 Device address negotiation & network initialization
├── static/
│   ├── style.css        # 🎨 Web styling
│   └── sounds/          # 🔊 Notification sounds
└── templates/
    └── index.html       # 🖥️ Frontend chat interface
```

<a id="setup-instructions"></a>
## ⚙️ Setup Instructions

### Hardware Requirements
- **LoRa Module**: RYLR998 (Reyax)
- **Connection**: USB-to-Serial adapter to connect the module to the host device.
- **Nodes**: At least two nodes are required for communication.

### Software Requirements
- **Python**: 3.12+ recommended.
- **Dependencies**: Install via pip:
  ```bash
  pip install -r requirements.txt
  ```

<a id="getting-started"></a>
## 🚀 Getting Started

### 1. Connect the LoRa Device
🔌 Plug your RYLR998 LoRa module into a USB port. Ensure the necessary drivers for your OS are installed.

### 2. Start the Application

**Option A: Web Interface (Recommended)**
Run the Flask server:
```bash
python app.py
```
The app will be available at `http://localhost:5300`.

**Option B: CLI Mode (Testing)**
Run the simple terminal interface:
```bash
python main.py
```

### 3. Configure the Port
- Open the web interface.
- Select the appropriate serial port (e.g., `COM3`, `/dev/ttyUSB0`) from the dropdown menu.

### 4. Messaging
- 📢 **Broadcast**: Use "Group Chat (Broadcast)" to send messages to all devices on the network.
- 📩 **Direct Message**: Click "+ Compose new message" to select a specific address from the list of discovered devices.

<a id="security"></a>
## 🔐 Security

- **Encryption**: Messages are encrypted using **AES (Fernet)** with a pre-shared key defined in `encryption_key.py`.
- **Authentication**: Each message includes an **HMAC-SHA256 signature** to verify sender identity and message integrity.
- **Relay Security**: Relayed messages maintain **end-to-end encryption**; intermediate nodes cannot decrypt the payload, ensuring privacy even in multi-hop scenarios.

<a id="protocol-deep-dive"></a>
## 📡 Protocol Deep Dive

<a id="packet-structure"></a>
### 📦 Packet Structure

Messages are sent as ASCII-encoded `AT+SEND` commands to the RYLR998 module. The internal payload uses the ASCII Unit Separator (`0x1F`) as a delimiter.

**Format:**
```
Flag (2B) | Message (Var) | SeqNum (2B) | Timestamp (Epoch)
```

- **Flag**: A 16-bit field (2 ASCII chars) controlling packet behavior (ACK, Broadcast, Relay, Training).
- **Message**: The encrypted AES payload + HMAC signature.
- **SeqNum**: A 14-bit random sequence number used for tracking and ACKs.
- **Timestamp**: Current epoch time for message ordering.

<a id="address-negotiation"></a>
### 🤖 Address Negotiation (Training)

When a node first joins, it lacks a unique address. The training phase resolves this:

1.  **Search**: The new node broadcasts a `SEARCH` packet (Flag `...00110000`).
2.  **Wait**: It enters a 30-second listening window.
3.  **Reply**: Existing nodes reply with their list of known hosts after a random delay (to avoid collisions).
4.  **Assignment**: The new node collects all used addresses, picks a random available ID (1-10000), and sets it via `AT+ADDRESS`.

<a id="direct-messaging"></a>
### 📨 Direct Messaging & Reliability

Reliability is ensured via a Stop-and-Wait ARQ mechanism:

1.  **Send**: A Direct Message (DM) is sent with bit-11 set (Request ACK).
2.  **Wait**: The sender waits up to 30 seconds for an ACK with a matching Sequence Number.
3.  **Retry**: If no ACK is received, the message is retransmitted (up to 5 attempts).
4.  **Failure**: If all retries fail, the system initiates the **Relay Protocol**.

<a id="relay-routing"></a>
### 🔄 Relay & Mesh Routing

When a direct path is unavailable, the system attempts to find a relay:

1.  **Discovery**: The sender broadcasts a "Who can reach Destination X?" query.
2.  **Offer**: Nodes that have recently heard from Destination X reply with an offer.
3.  **Handshake**: The sender selects the first available relay.
4.  **Forwarding**: The encrypted packet is wrapped in a relay envelope and sent to the relay node, which then forwards it to the final destination.

<a id="troubleshooting"></a>
## ❓ Troubleshooting

- **No COM Port Found**: Ensure your USB-to-Serial driver is installed (e.g., CH340 or CP210x). Check Device Manager (Windows) or `/dev/` (Linux/macOS).
- **SocketIO Errors**: If the UI disconnects, refresh the page. The backend handles reconnection automatically.
- **Dependencies**: If `pip install` fails, try upgrading pip: `python -m pip install --upgrade pip`.
- **"Training Failed"**: Move closer to other nodes or ensure at least one other node is powered on to respond to the address search.

<a id="authors"></a>
## 👥 Authors

Developed @ **NMSU** - Spring 2025 Cellular Networks and Mobile Computing Programming Project.

- **Jack Nolen**
- **Nathan Hoxworth**
- **Horacio Gonzalez**
- **Christian Garcia Rivero**
