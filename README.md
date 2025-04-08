# Master OWL — CANBus + Node-RED Control System

> **Branch Purpose**  
This branch introduces a CANBus-based master-slave section control system for OpenWeedLocator (OWL), utilizing a **master Raspberry Pi running Node-RED** to control multiple **slave Raspberry Pis**, each managing a 1-meter-wide spraying section.

---

## Features

- Node-RED master node to control section ON/OFF
- CANBus communication via `socketcan` (physical or virtual)
- GPIO-to-CAN bridging for physical buttons or STM32 triggers
- Broadcast support for ALL ON / ALL OFF
- Compatible with Open AgiGPS for auto section control

---

## System Overview

```
         STM32 / GPIO Inputs
                 │
                 ▼
        +---------------------+
        |  Master Raspberry Pi|
        |  (Node-RED + CANBus)|
        +---------------------+
                 │
       CAN Bus   ▼
+---------------------+     +---------------------+     +---------------------+
| RPi Slave: Meter 1  | <-->| RPi Slave: Meter 2  | <-->| RPi Slave: Meter 3  |
+---------------------+     +---------------------+     +---------------------+

                ...up to 36 total...
```

---

| **Feature**            | **Description**                              | **CAN IDs per Slave** | **Payloads**                              | **Control Type**      |
|------------------------|----------------------------------------------|-----------------------|-------------------------------------------|-----------------------|
| All Nozzles On/Off     | Turns all solenoids on or off                | 0x201–0x204          | `[0x02]` = ON, `[0x00]` = OFF              | Global toggle         |
| Recording Toggle       | Toggles data recording on/off                | 0x301–0x304          | `[0x01]` = ON, `[0x00]` = OFF              | Global or per slave   |
| Sensitivity Toggle     | Cycles through sensitivity levels (0–10)     | 0x401–0x404          | `[0x00–0x0A]` (0–10 sensitivity levels)     | Global cycle          |
| Spot Spray Enable      | Enables/disables OWL spray logic             | 0x501–0x504          | `[0x01]` = ON, `[0x00]` = OFF              | Per slave toggle      |

## Setup Instructions

### 1. Install Dependencies

Install on **Raspberry Pi** or **WSL2 Ubuntu**:

```bash
sudo apt update
sudo apt install -y nodejs npm can-utils
sudo npm install -g --unsafe-perm node-red
```

---

### 2. Setup CAN Interface

#### For Testing (virtual vcan):
```bash
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

#### For Real CAN Hardware (e.g., MCP2515):
```bash
sudo ip link set can0 up type can bitrate 500000
```

---

### 3. Run Node‑RED

Start Node-RED on the master Raspberry Pi or WSL2:

```bash
node-red
```

Access in your browser:

- Localhost: [http://localhost:1880](http://localhost:1880)
- RPi IP: [http://<raspberrypi-ip>:1880](http://<raspberrypi-ip>:1880)

---

### 4. Import Node‑RED Flows

- In Node-RED Editor → Click Menu → **Import** → **Clipboard**
- Paste flow JSON (from this repo: `flows/master_flow.json`, `meter*_flow.json`)
- Click **Import** → **Deploy**

---

### 5. GPIO Input Integration (STM32 or Buttons)

1. Use `rpi-gpio in` nodes for each section
2. Configure them to trigger a function or inject node
3. Translate GPIO to CAN frame:
   - `meter = section number`
   - `command = 1 (ON)` or `0 (OFF)`
4. Send via `socketcan out` node on `can0`

---

## Message Format

Each CAN frame has:

```js
id: 0x300,
ext: false,
data: [meter, command]  // meter: 1–36 or 255 for broadcast, command: 0=OFF, 1=ON
```

---

## Enhancement Ideas

- [ ] Dashboard for real-time state monitoring
- [ ] Feedback from slaves (status/error messages)
- [ ] OWL trigger integration (pause/play detection)
- [ ] Over-the-air updates to all slaves
- [ ] Sync with AgiGPS section control logic

---

## Branch Strategy

This branch is `canbus-master`.

You can:

```bash
git checkout -b feature/canbus-control
```

Then push PRs for:
- CAN automation
- GPIO upgrades
- Flow improvements
- UI for control and error messages

---

## Credits

Created by **CropCrusaders**  
Backed by the **OpenWeedLocator** open-source initiative  
Licensed under MIT
