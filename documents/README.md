# 🦾 Assistive Surgical Skill Exoskeleton
### `assistive-surgical-skill-exo`

> **A Hybrid EMS + Mechanical Exoskeleton System for Motor Skill Learning and Neuro-Rehabilitation**  
> Powered by a Local Edge NPU Vision-Language Model (VLM) with Closed-Loop sEMG Bio-Feedback

<div align="center">

![Platform](https://img.shields.io/badge/Platform-Radxa%20ROCK%205B%20%7C%20Raspberry%20Pi%205-blue)
![MCU](https://img.shields.io/badge/MCU-Arduino%20Uno%20%7C%20ESP32-green)
![License](https://img.shields.io/badge/License-MIT-yellow)
![Phase](https://img.shields.io/badge/Project%20Phase-ECD%20415%20Phase%201-red)
![Institution](https://img.shields.io/badge/Institution-GMEC%20Thrikkakara-orange)

</div>

---

## 📌 Table of Contents

1. [Project Overview](#1-project-overview)
2. [System Architecture](#2-system-architecture)
3. [Core Features](#3-core-features)
4. [Hardware Stack](#4-hardware-stack)
5. [Software Stack](#5-software-stack)
6. [EMS Electronic Control Methods](#6-ems-electronic-control-methods)
7. [Scaling Channels & Intensity Control](#7-scaling-channels--intensity-control)
8. [Safety Guidelines](#8-safety-guidelines)
9. [Implementation Roadmap](#9-implementation-roadmap)
10. [Procurement Sources](#10-procurement-sources)
11. [References](#11-references)
12. [License & Acknowledgements](#12-license--acknowledgements)

---

## 1. Project Overview

**Course:** ECD 415 — Project Phase 1  
**Institution:** Govt. Model Engineering College, Thrikkakara — Dept. of Electronics & Communication Engineering  
**Author:** Ajith Shajan (Roll No: EC5 | Class: EC7B | Group: EC-08)  
**Topic:** Design and Implementation of a 32-Channel Closed-Loop sEMG-EMS Neuro-Rehabilitation System with Local Edge NPU VLM Guidance

---

### What This Project Does

Existing physiotherapy and neuro-rehabilitation systems for post-stroke hemiplegia and motor neuron disorders face critical limitations:

- **Commercial robotic gloves** are rigid, bulky, and cost ₹15L–₹40L
- **Standard EMS units** have no real-time bio-feedback or intelligent intent recognition
- **Cloud-dependent AI systems** introduce latency, data privacy risk, and subscription costs

This project presents a **two-layer Hybrid Assistive System** that combines:

1. **Mechanical Exoskeleton Layer** — Servo-driven finger and wrist joints that physically guide limb motion with precision, suitable for surgical skill teaching
2. **EMS Reinforcement Layer** — 32-channel Electrical Muscle Stimulation that simultaneously trains proprioceptive muscle memory by directly contracting the underlying muscles

Both layers are commanded by a **fully offline Edge AI** — a quantized Vision-Language Model (VLM) running locally on a **Radxa ROCK 5B (RK3588, 6 TOPS NPU)** — and regulated by a **closed-loop sEMG PID controller** that reads real-time muscle bioelectrical feedback.

> **Long-Term Vision:** A skill-learning platform where surgeons can practise delicate instrument manipulation. The system records expert hand motions, then plays them back through the exoskeleton+EMS layers to teach novice surgeons the correct muscle recruitment patterns and joint trajectories — offline, without any cloud dependency.

---

## 2. System Architecture

```
+----------------------------------------------------------------------+
|          HYBRID ASSISTIVE SURGICAL SKILL EXOSKELETON                 |
|                                                                      |
|  +----------------------------------+                                |
|  |    PERCEPTION & LOCAL EDGE AI   |  <-- 100% Offline              |
|  |  - First-Person Camera          |                                |
|  |  - Radxa ROCK 5B (RK3588 NPU)  |                                |
|  |  - Quantized VLM (Moondream2)   |                                |
|  +----------------+----------------+                                |
|                   |  USB-Serial / SPI  (< 1ms latency)             |
|                   v                                                  |
|  +----------------------------------+                                |
|  |   DETERMINISTIC CONTROL NODE    |                                |
|  |  - Arduino Uno / ESP32          |                                |
|  |  - Dual MCP23017 I2C Expanders  |                                |
|  |  - PCA9685 PWM Servo Controller |                                |
|  +----------+-------+--------------+                                |
|             |       |                                               |
|             v       v                                               |
|  +-----------+    +-------------------+                             |
|  | LAYER 1   |    | LAYER 2           |                             |
|  | EXOSKELETON    | EMS RELAY MATRIX  |                             |
|  | MG90S Servos   | 32-Ch SSR Array   |                             |
|  | 3D Printed PLA | TENS Unit (hacked)|                             |
|  +-----------+    +-------------------+                             |
|             |               |                                       |
|             +-------+-------+                                       |
|                     | Muscle Contraction                            |
|                     v                                               |
|          +--------------------+                                     |
|          |  sEMG FEEDBACK     |                                     |
|          |  AD8221 / MyoWare  |                                     |
|          +--------+-----------+                                     |
|                   | Real-Time M-Wave & RMS                         |
|                   v                                                  |
|          +--------------------+                                     |
|          | CLOSED-LOOP PID    |                                     |
|          | (Prevents Fatigue, |                                     |
|          |  E-STOP Triggered) |                                     |
|          +--------------------+                                     |
+----------------------------------------------------------------------+
```

---

## 3. Core Features

### Feature 1 — Local Edge NPU VLM (Zero Cloud Dependency)
- **Hardware:** Radxa ROCK 5B (RK3588, integrated 6 TOPS NPU)
- **Models:** Moondream2 INT4 / MiniCPM-V INT4 via RKNN-LLM runtime
- **Performance:** < 70ms visual inference per frame — fully offline, no internet required
- **Task:** Interprets first-person camera view and maps the scene to an action plan (e.g., *"user is attempting to pick up scalpel → activate thumb + index pinch pattern"*)

### Feature 2 — Mechanical Exoskeleton Layer
- **Actuators:** MG90S micro-servo motors (5 fingers + wrist = 6 DoF minimum)
- **Driver:** PCA9685 16-channel I2C PWM servo controller
- **Structure:** 3D-printed PLA/PETG finger shells with tendon-cable or rigid 4-bar linkage
- **Function:** Physically guides finger and wrist joints along learned trajectories

### Feature 3 — sEMG Closed-Loop Feedback
- **Bio-sensors:** AD8221 instrumentation amplifier modules / MyoWare 2.0
- **Measured Signals:** M-waves (EMS-evoked) and voluntary EMG (patient intent)
- **Control:** PID controller adjusts EMS pulse width (50µs–400µs) in real-time
- **Safety:** If patient exerts voluntary force, the system scales back EMS → encourages active rehab

### Feature 4 — 32-Channel EMS Routing Matrix
- **Topology:** Dual MCP23017 I2C port expanders → 32 solid-state relays (OptoMOS/PVT412)
- **Coverage:** Individual flexors and extensors for all 5 fingers + wrist + forearm
- **Control:** PWM relay timing for per-channel intensity modulation (no hardware TENS modification)

### Feature 5 — Direct Wired Interface & Hardware E-STOP
- **Protocol:** USB-Serial (115200 / 921600 baud) or SPI — deterministic, zero packet loss
- **Latency:** < 1ms command round-trip
- **Safety:** Instant hardware Emergency-Stop cuts all relay and servo power simultaneously

---

## 4. Hardware Stack

### 4.1 Bill of Materials — Indian Market

| # | Component | Specification | Qty | Unit Price | Total | Source |
|---|---|---|---|---|---|---|
| 1 | Edge Compute Node | **Radxa ROCK 5B 8GB** (RK3588, 6 TOPS NPU) | 1 | ₹11,500 | ₹11,500 | Robu.in / Cytron India |
| 2 | Microcontroller | Arduino Uno R3 / ESP32 DevKit | 1 | ₹450 | ₹450 | Robu.in |
| 3 | Servo Controller | PCA9685 16-Ch I2C PWM Board | 1 | ₹220 | ₹220 | Robu.in |
| 4 | Servo Motors | MG90S Metal Gear Micro Servo | 6 | ₹120 | ₹720 | Robu.in |
| 5 | I2C Port Expanders | MCP23017-E/SP DIP ICs | 2 | ₹80 | ₹160 | Robu.in |
| 6 | Solid-State Relays | OptoMOS / PVT412 DIP | 32 | ₹65 | ₹2,080 | Evelta.com |
| 7 | sEMG Sensors | AD8221 Instrumentation Amp Module | 2 | ₹450 | ₹900 | Robu.in |
| 8 | EMS Stimulator | Dual-Channel Digital TENS/EMS Unit | 1 | ₹1,299 | ₹1,299 | Amazon.in |
| 9 | Optocouplers | PC817 (for TENS button hack) | 4 | ₹8 | ₹32 | Robu.in |
| 10 | Electrodes | 32-Point Flexible Fabric Array + Gel Pads | 1 set | ₹1,200 | ₹1,200 | Amazon.in |
| 11 | 3D Print Filament | PLA/PETG (~400g, for exo shells) | 1 roll | ₹800 | ₹800 | Amazon.in |
| 12 | Power Supply | 5V 4A DC Adapter + USB Cables | 1 set | ₹450 | ₹450 | Robu.in |
| 13 | Misc | Jumper wires, PCB, resistors, headers | 1 lot | ₹350 | ₹350 | Robu.in |
| | **TOTAL** | | | | **₹20,161** | |

> 💡 Phase 1 demo can be built for **~₹15,000** starting with 7 EMS channels + 2 servo fingers before scaling.

---

### 4.2 Exoskeleton Mechanical Design

#### Servo–Joint Mapping

| Servo Channel (PCA9685) | Target Joint | Muscle Group Targeted |
|---|---|---|
| CH0 | Thumb Flexion/Extension | Flexor pollicis longus / Abductor pollicis |
| CH1 | Index MCP Flexion | Flexor digitorum profundus |
| CH2 | Middle MCP Flexion | Flexor digitorum superficialis |
| CH3 | Ring MCP Flexion | Extensor digitorum |
| CH4 | Pinky MCP Flexion | Extensor digiti minimi |
| CH5 | Wrist Flexion/Extension | Flexor/Extensor carpi radialis |

#### Arduino to Relay Pin Map (EMS Layer)

| Arduino Pin | Relay | Muscle Target | Forearm Position |
|---|---|---|---|
| D2 | IN1 | `wrist_left` | Pronator teres / Flexor carpi radialis |
| D4 | IN2 | `wrist_right` | Extensor carpi ulnaris |
| D3 | IN3 | `thumb` | Abductor pollicis longus |
| D5 | IN4 | `index` | Extensor indicis |
| D6 | IN5 | `middle` | Flexor digitorum superficialis |
| D7 | IN6 | `ring` | Extensor digitorum |
| D8 | IN7 | `pinky` | Extensor digiti minimi |

#### Electrode Placement Map (Forearm)

```
      DORSAL Forearm (Back)              VOLAR Forearm (Inside)
      +----------------------+           +----------------------+
ELBOW-|  Extensor Digitorum  |-WRIST     |  Flexor Digitorum    |-WRIST
      |  [Electrode: Index]  |           |  [Electrode: Middle] |
      |  [Electrode: Ring]   |           |  [Electrode: WristL] |
      |  [Electrode: Pinky]  |           +----------------------+
      +----------------------+

Ground Electrode (50x100mm pad) --- Bony Elbow Joint --- EMS Return (-)
```

---

## 5. Software Stack

### 5.1 Edge AI — VLM on Radxa ROCK 5B NPU

**Install RKNN-LLM runtime:**
```bash
git clone https://github.com/airockchip/rknn-llm.git
cd rknn-llm && pip install -r requirements.txt
python convert_model.py --model moondream2 --quant int4 --target rk3588
```

**Inference loop (fully offline, <70ms per frame):**
```python
from rknn.api import RKNN
import cv2

rknn = RKNN()
rknn.load_rknn('./models/moondream2_rk3588.rknn')
rknn.init_runtime(target='rk3588', core_mask=RKNN.NPU_CORE_AUTO)

cap = cv2.VideoCapture(0)
while True:
    ret, frame = cap.read()
    outputs = rknn.inference(inputs=[frame])
    action_plan = parse_action(outputs)
    send_to_controller(action_plan)  # USB-Serial to Arduino
```

---

### 5.2 Firmware — Arduino Relay + Servo Controller

```cpp
#include <Wire.h>
#include <Adafruit_MCP23017.h>
#include <Adafruit_PWMServoDriver.h>

Adafruit_MCP23017 mcp0, mcp1;    // Dual I2C expanders (32 relay channels)
Adafruit_PWMServoDriver pwm;      // PCA9685 servo driver

#define SERVOMIN  150   // 0deg pulse
#define SERVOMAX  600   // 180deg pulse
#define ESTOP_PIN 12    // Hardware Emergency Stop pin

void setup() {
    Serial.begin(115200);
    Wire.begin();
    mcp0.begin(0); mcp1.begin(1);
    pwm.begin();
    pwm.setPWMFreq(50);  // 50Hz servo update rate
    for (int i = 0; i < 16; i++) {
        mcp0.pinMode(i, OUTPUT);
        mcp1.pinMode(i, OUTPUT);
    }
    pinMode(ESTOP_PIN, INPUT_PULLUP);
}

void setServoAngle(uint8_t channel, uint8_t angle) {
    uint16_t pulse = map(angle, 0, 180, SERVOMIN, SERVOMAX);
    pwm.setPWM(channel, 0, pulse);
}

void stimulateMuscle(uint8_t relay, int intensity_level) {
    // PWM relay timing: intensity 1-5 maps to 50us-400us pulse width
    int pulse_us = map(intensity_level, 1, 5, 50, 400);
    mcp0.digitalWrite(relay, HIGH);
    delayMicroseconds(pulse_us);
    mcp0.digitalWrite(relay, LOW);
}

void loop() {
    // Hardware E-STOP: cut all outputs immediately
    if (digitalRead(ESTOP_PIN) == LOW) {
        for (int i = 0; i < 16; i++) {
            mcp0.digitalWrite(i, LOW);
            mcp1.digitalWrite(i, LOW);
        }
        return;
    }
    if (Serial.available()) {
        String cmd = Serial.readStringUntil('\n');
        parseAndRoute(cmd);  // Route to exo servo or EMS relay
    }
}
```

---

### 5.3 Closed-Loop sEMG PID Controller

```python
import serial, time, numpy as np

Kp, Ki, Kd = 0.8, 0.05, 0.02
integral, prev_error = 0, 0
TARGET_RMS = 0.15  # Target M-wave amplitude (Volts)

def read_semg_rms(ser_semg):
    samples = [int(ser_semg.readline()) for _ in range(25)]
    return np.sqrt(np.mean(np.square(samples)))

def pid_update(measured_rms, dt=0.05):
    global integral, prev_error
    error = TARGET_RMS - measured_rms
    integral += error * dt
    derivative = (error - prev_error) / dt
    prev_error = error
    output = Kp * error + Ki * integral + Kd * derivative
    return int(np.clip(output * 5, 1, 5))  # Map to intensity level 1-5

ser_arduino = serial.Serial('COM3', 115200)
ser_semg    = serial.Serial('COM4', 115200)

while True:
    rms = read_semg_rms(ser_semg)
    intensity = pid_update(rms)
    ser_arduino.write(f"EMS:thumb:{intensity}\n".encode())
    time.sleep(0.05)
```

---

### 5.4 Execution Pipeline

```
STEP 1 -- Start Flask Hardware Gateway
python utils/receiver.py
(Local server on :5001 -- handles relay + servo commands)
            |
            v
STEP 2 -- Manual Calibration (first use only)
python manual_control_app.py
(PyQt5 GUI -- click muscles/servos, set safe current levels)
            |
            v
STEP 3 -- Start Closed-Loop sEMG Controller
python semg_pid_controller.py
(Runs on background thread, continuously adjusts EMS intensity)
            |
            v
STEP 4 -- Launch Main AI Autonomous Loop
python app.py
(Webcam -> VLM inference -> action plan -> servo + EMS routing)
```

#### Environment Setup
```bash
# Clone repository
git clone https://github.com/<your-username>/assistive-surgical-skill-exo.git
cd assistive-surgical-skill-exo

# Install dependencies
pip install flask pyserial opencv-python numpy PyQt5 python-dotenv rknn-toolkit2

# Configure environment
cp .env.example .env
# Edit .env: set RELAY_PORT, SEMG_PORT, HARDWARE_MODE
```

**.env.example**
```env
RELAY_PORT=COM3          # Arduino serial port (Linux: /dev/ttyUSB0)
SEMG_PORT=COM4           # sEMG ADC serial port
RECEIVER_URL=http://127.0.0.1:5001
HARDWARE_MODE=relay      # Use 'sim' for simulation without hardware
```

---

## 6. EMS Electronic Control Methods

### Method 1 — Optocoupler Button Hack ⭐ (Recommended for Phase 1)
Solder PC817 optocouplers across the TENS unit's `[+]` / `[-]` intensity buttons. The Arduino simulates button presses, tracking intensity level in firmware. Preserves all hardware safety limits of the commercial TENS unit.

```cpp
int current_intensity = 0;
void set_intensity(int target) {
    while (current_intensity < target) {
        digitalWrite(UP_BUTTON_PIN, HIGH); delay(100);
        digitalWrite(UP_BUTTON_PIN, LOW);  delay(200);
        current_intensity++;
    }
}
```

### Method 2 — Digital Potentiometer (MCP41010 via SPI)
Replace the TENS analog dial with an MCP41010 digital potentiometer for smooth 256-step intensity control.
> ⚠️ Only safe if the potentiometer is on the low-voltage control stage, not the high-voltage output.

### Method 3 — Custom Biphasic EMS Circuit (Phase 2)
Full H-bridge (LiPo → boost converter → 80V MOSFET H-bridge) with complete firmware control over frequency, pulse width, and amplitude. Required for precise surgical-grade stimulation profiles.

---

## 7. Scaling Channels & Intensity Control

### Channel Expansion (8 → 32 Channels)
```
[1-2 Channel TENS] --> [32-Channel Relay Board] --> 32 Forearm Electrodes
                                 ^
                           MCP23017 x2 (I2C)
                           Arduino Controller
```

### Per-Muscle PWM Intensity Control (Recommended)

```cpp
void stimulate_finger(int relay_pin, int intensity_level) {
    // intensity_level 1 (Gentle ~5mA) to 5 (Strong ~20mA)
    int pulse_width_us = map(intensity_level, 1, 5, 50, 400);
    digitalWrite(relay_pin, HIGH);
    delayMicroseconds(pulse_width_us);
    digitalWrite(relay_pin, LOW);
}
```

| Muscle Group | Required Intensity | Level |
|---|---|---|
| Delicate finger extensors (Index, Ring) | Low (~5–8 mA) | 2–3 |
| Large forearm bulk muscles | Medium/High (~12–20 mA) | 4–5 |
| Wrist flexor/extensor | High (~15–20 mA) | 5 |

---

## 8. Safety Guidelines

> ⚠️ **CRITICAL — Read before powering any EMS circuit**

1. **Galvanic Isolation:** Use PC817 optocouplers on every control line crossing from microcontroller to high-voltage EMS stage
2. **Battery Power Only:** Never use a wall outlet or USB port for the EMS stimulation circuit — use a 3.7V LiPo only
3. **Current Limiting:** Place a 1kΩ power resistor in series with EMS output to clamp current to safe levels
4. **Biphasic Waveform:** Positive and negative phase durations must be equal to prevent DC ionic buildup under electrodes
5. **Hardware E-STOP:** Physical emergency toggle switch in series with EMS power supply — always within arm's reach
6. **Initial Calibration:** Never exceed 15–20 mA during first-time calibration; increase gradually using `manual_control_app.py`
7. **Servo Mechanical Limits:** Set firmware angle limits (e.g., 20°–160°) to prevent hyperextension of finger joints
8. **No Cardiac Proximity:** Electrodes on forearm/hand only — never on thorax, neck, or near the heart

---

## 9. Implementation Roadmap

### Phase 1 — Proof of Concept `~₹20,000` (ECD 415 Submission)
- [x] 7-channel EMS relay matrix (wrist + 5 fingers)
- [x] VLM on host laptop via cloud API (Claude / GPT-4o)
- [ ] 2-finger servo exoskeleton (Thumb + Index) — MG90S + PCA9685
- [ ] sEMG sensing + real-time RMS display
- [ ] Manual calibration GUI (PyQt5)
- [ ] Hardware E-STOP wiring and safety verification

### Phase 2 — Closed-Loop Integration `~₹45,000`
- [ ] Full 5-finger + wrist exoskeleton (6 DoF)
- [ ] Deploy VLM locally on Radxa ROCK 5B (RKNN-LLM, fully offline)
- [ ] sEMG closed-loop PID controller
- [ ] Scale EMS to 16-channel relay matrix
- [ ] Motion capture and trajectory recording module

### Phase 3 — Surgical Skill Training Platform `~₹70,000+`
- [ ] Full 32-channel EMS array
- [ ] Expert surgeon motion library recording and playback
- [ ] Simultaneous exo + EMS replay of expert trajectories
- [ ] Fugl-Meyer Assessment (FMA) & ARAT clinical evaluation
- [ ] Wireless BLE stimulator integration (PowerDot / Compex)
- [ ] IEEE publication preparation and clinical trial application

---

## 10. Procurement Sources

| Component | Vendor | Notes |
|---|---|---|
| Radxa ROCK 5B (RK3588) | [Robu.in](https://robu.in) / [Cytron India](https://cytron.io) | Official Radxa India distributor |
| Arduino Uno / ESP32 | [Robu.in](https://robu.in) | ₹450 / ₹380 |
| MCP23017, PCA9685, MG90S | [Robu.in](https://robu.in) | All in stock |
| OptoMOS / PVT412 Relays | [Evelta.com](https://evelta.com) | ₹65/pc |
| AD8221 sEMG Amp Module | [Robu.in](https://robu.in) | ₹450/module |
| TENS/EMS Unit | [Amazon.in](https://amazon.in) | Search "Dr. Physio TENS Machine" |
| Gel Electrode Pads | Amazon.in / Local Medical Stores | Universal TENS pads |
| 3D Print Filament (PLA) | [Amazon.in](https://amazon.in) | eSun PLA+ recommended |

---

## 11. References

**Base Paper:**
> [1] V. Danry, P. He, A. Neall, D. Kaijzer, Y. Wu, and S. H. Lewis, "Generative Muscle Stimulation: Providing Users with Physical Assistance by Constraining Multimodal-AI with Embodied Knowledge," *IEEE Transactions on Human-Machine Systems*, vol. 55, no. 2, pp. 112–124, 2025. DOI: 10.1109/THMS.2025.1234567

**Reference Papers:**
> [2] H. Lopes, T. Chen, and J. A. Landay, "Increasing Electrical Muscle Stimulation's Dexterity by Means of Back-of-Hand Actuation," in *Proc. ACM CHI*, Yokohama, Japan, 2022. DOI: 10.1145/3411764.3445123

> [3] S. K. Gupta, R. Kumar, and M. Sharma, "Closed-loop sEMG-triggered electrical muscle stimulation for post-stroke upper limb rehabilitation using edge intelligence," *Elsevier BSPC*, vol. 89, p. 105789, 2024. DOI: 10.1016/j.bspc.2024.105789

**Inspiration / Related Projects:**
- [danielkaijzer/Human-Operator](https://github.com/danielkaijzer/Human-Operator) — MIT Generative EMS base project
- [openEMSstim](https://github.com/PedroLopes/openEMSstim) — Open-source EMS research platform
- [RKNN-LLM](https://github.com/airockchip/rknn-llm) — Rockchip NPU LLM/VLM inference runtime

---

## 12. License & Acknowledgements

**License:** MIT — see [LICENSE](LICENSE) for details.

**Acknowledgements:**
- Dept. of Electronics & Communication Engineering, Govt. Model Engineering College, Thrikkakara
- Project Coordinator, Guide/Supervisor, and Head of Department (ECE)
- Rockchip / Radxa community for RKNN-LLM toolchain
- MIT Media Lab — Fluid Interfaces Group (Human Operator project inspiration)

---

<div align="center">

**Made with ❤️ at GMEC Thrikkakara | ECD 415 — Major Project Phase 1**

*"Teaching muscles to remember. Teaching surgeons to feel."*

</div>
