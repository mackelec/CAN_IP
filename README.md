# canip_peer v4.1

`canip_peer` is a lightweight Python tool for bridging **CAN bus traffic over TCP/IP networks**.  
It makes it easy to forward CAN frames between a Raspberry Pi (with a CAN-USB adapter) and a Linux desktop, or to test with virtual CAN (VCAN) interfaces.

---

## Overview

With `canip_peer`, you can:

- Monitor or capture CAN traffic remotely.  
- Extend CAN buses across Ethernet, Wi-Fi, or VPN links.  
- Test distributed systems with a Raspberry Pi in the field and a Linux workstation at your desk.  

**Supported platforms:**  
- Linux Mint, Raspberry Pi OS (Raspbian)  
- SocketCAN interface  

---

## Features

- ✅ **CAN ↔ TCP bridging** (server/client modes)  
- ✅ **Heartbeat support** for link monitoring  
- ✅ **Optional sequencing** to detect dropped/duplicate frames  
- ✅ **Filtering** with ID lists, ranges, and masks  
- ✅ **Logging** with timestamps  
- ✅ **Works with VCAN** (for safe testing without hardware)  
- ✅ **Pure Python 3**  

---

## Installation

Clone and install requirements:

```
sudo apt update
sudo apt install python3 python3-pip can-utils
git clone https://github.com/<your-repo>/canip_peer.git
cd canip_peer
pip3 install -r requirements.txt
```

---

## Preparing CAN Devices

### CAN-USB Adapter (CandleLight/gs_usb)

```
sudo ip link set can0 up type can bitrate 500000
ip -details link show can0
```

### Virtual CAN (VCAN) — for testing

```
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set vcan0 up
```

---

## Usage

### Raspberry Pi (server)

```
python3 canip_peer.py server --can can0 --port 3333 --log-level INFO
```

### Linux PC (client)

```
sudo ip link add dev vcan0 type vcan
sudo ip link set vcan0 up

python3 canip_peer.py client --can vcan0 --host <pi-ip> --port 3333 --heartbeat 2 --log-level INFO
```

Now all traffic from `can0` on the Pi will appear on the PC’s `vcan0`.

---

## Examples

### 1. VCAN only (single machine)

```
# Terminal 1 — server
python3 canip_peer.py server --can vcan0 --port 3333

# Terminal 2 — client
python3 canip_peer.py client --can vcan1 --host 127.0.0.1 --port 3333

# Terminal 3 — send CAN frame
cansend vcan0 123#11223344

# Terminal 4 — receive
candump vcan1
```

### 2. Raspberry Pi to Linux Desktop

**On Raspberry Pi:**
```
sudo ip link set can0 up type can bitrate 500000
python3 canip_peer.py server --can can0 --port 3333 --log-level INFO
```

**On Linux Desktop:**
```
sudo ip link add dev vcan0 type vcan
sudo ip link set vcan0 up
python3 canip_peer.py client --can vcan0 --host <pi-ip> --port 3333 --heartbeat 2 --log-level INFO
candump vcan0
```

---

## Options

- `--can <iface>` : CAN interface (e.g., `can0`, `vcan0`)  
- `--port <n>` : TCP port (default 3333)  
- `--host <ip>` : Server IP (client mode)  
- `--heartbeat <sec>` : Send heartbeat every N seconds  
- `--log-level <level>` : DEBUG, INFO, WARNING, ERROR  
- `--no-seq` : Disable sequence checking  

---

## Troubleshooting

- **CAN device not found** → check with `ip link show`  
- **No traffic on client** → ensure firewall allows TCP port 3333  
- **Endless loop of frames** → don’t connect both peers to the same CAN bus simultaneously  

---

## License

MIT License — free to use, modify, and distribute.  

