<div align="center">

# 💬 LoRa Chat
### Secure Peer-to-Peer and Broadcast Messaging over LoRa

_A robust, secure messaging application enabling encrypted communication between multiple nodes using RYLR998 LoRa modules. Features real-time chat via a web interface, multi-hop relay capabilities, and automated network discovery._

[![Python](https://img.shields.io/badge/Python-3.12%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.0.0-green?logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![Socket.IO](https://img.shields.io/badge/Socket.IO-Real--Time-black?logo=socket.io&logoColor=white)](https://socket.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

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
- [📡 Protocol Design](#protocol-design)
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

<a id="protocol-design"></a>
## 📡 Protocol Design

| Component | Responsibility |
|-----------|----------------|
| **`app.py`** | Manages the Flask server and SocketIO events, bridging the web UI with the backend logic. |
| **`Messenger.py`** | The central controller that coordinates communication, message caching, and protocol handling. |
| **`Comm.py`** | Handles low-level serial communication with the RYLR998 module using AT commands. |
| **`Message.py`** | Responsible for packet structure, serialization, AES encryption/decryption, and HMAC verification. |
| **`Relay.py`** | Implements the relay logic to forward messages when direct transmission is not possible. |
| **`DirectMessage.py`** | Manages the reliable delivery of direct messages using acknowledgments. |
| **`Training.py`** | Handles the initial network discovery and address assignment phase. |

<a id="authors"></a>
## 👥 Authors

Developed @ **NMSU** - Spring 2025 Cellular Networks and Mobile Computing Programming Project.

- **Jack Nolen**
- **Nathan Hoxworth**
- **Horacio Gonzalez**
- **Christian Garcia Rivero**
