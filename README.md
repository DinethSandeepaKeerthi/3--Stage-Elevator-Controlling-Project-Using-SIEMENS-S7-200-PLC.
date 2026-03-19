# 🏗️ 3-Stage Elevator Controlling Project
### Using SIEMENS S7-200 PLC + TB6600 Stepper Driver + NEMA 17 Motor

> **Industrial Automation | KEERTHI P.E.D.S.**

---

## 📌 Project Overview

This project implements a fully automated **3-floor elevator control system** using a **SIEMENS S7-200 PLC** programmed in **Ladder Logic (LAD)**. The elevator responds to floor call buttons and uses **inductive proximity sensors** to detect floor positions, driving a **NEMA 17 stepper motor** via a **TB6600 stepper motor driver**.

The system is designed to demonstrate real-world industrial PLC programming concepts including memory bit state machines, motor direction control, sensor-based interlocking, and multi-floor sequencing.

---

## 🎯 Objectives

- Design and implement a 3-floor elevator control logic using SIEMENS S7-200 PLC
- Use Ladder Diagram (LAD) programming with memory markers (M-bits) as a state machine
- Interface push buttons (floor call) and proximity sensors (floor detection) with the PLC
- Control a NEMA 17 stepper motor (via TB6600 driver) for UP/DOWN movement
- Demonstrate motor direction control and automatic stop at correct floors

---

## 🔧 Hardware Components

| Component | Model / Spec | Quantity | Purpose |
|---|---|---|---|
| PLC | SIEMENS S7-200 | 1 | Main controller |
| Stepper Motor | NEMA 17 | 1 | Elevator drive motor |
| Stepper Driver | TB6600 | 1 | Motor current & step control |
| Push Buttons | Momentary NO | 3 | Floor call buttons (PB01, PB02, PB03) |
| Proximity Sensors | Inductive Metal Detect | 3 | Floor position detection (S1, S2, S3) |
| Power Supply | 12V SMPS | 1 | System power |
| Wiring, DIN Rail, Panel | — | — | Enclosure & wiring |

---

## 🖥️ Software & Tools

- **SIEMENS STEP 7 Micro/WIN** — PLC programming environment
- **Programming Language** — Ladder Diagram (LAD)
- **PLC Communication** — PPI (Point-to-Point Interface) cable

---

## 📐 System Architecture

```
[PB01 Floor 1] ──┐
[PB02 Floor 2] ──┤──► SIEMENS S7-200 PLC ──► TB6600 Driver ──► NEMA 17 Motor
[PB03 Floor 3] ──┘         │                                        │
                            │                                    [Elevator]
[S1 Floor 1 Sensor] ───────┤
[S2 Floor 2 Sensor] ───────┤◄── Position Feedback
[S3 Floor 3 Sensor] ───────┘
```

### I/O Mapping

| Symbol | PLC Address | Type | Description |
|---|---|---|---|
| PB01 | I0.0 | Digital Input | Floor 1 call button |
| PB02 | I0.1 | Digital Input | Floor 2 call button |
| PB03 | I0.2 | Digital Input | Floor 3 call button |
| S1 | I0.3 | Digital Input | Floor 1 proximity sensor |
| S2 | I0.4 | Digital Input | Floor 2 proximity sensor |
| S3 | I0.5 | Digital Input | Floor 3 proximity sensor |
| RUN | Q0.0 | Digital Output | Motor RUN enable (to TB6600) |
| DIR | Q0.1 | Digital Output | Motor direction (UP/DOWN) |

---

## 🔄 State Machine Logic

The program uses **M-memory bits (M0.0 – M1.7)** as a sequential state machine. Each state represents a specific step in the elevator's decision and movement process.

### Memory Bit Definitions

| Memory Bit | State Description |
|---|---|
| M0.0 | System INIT / Motor STOP state |
| M0.1 | Wait for PB01 (Floor 1 call) |
| M0.2 | Check S1 sensor (Floor 1 position) |
| M0.3 | Motor DOWN (moving toward Floor 1) |
| M0.4 | Check S1 ON (arrived at Floor 1) |
| M0.5 | Wait for PB02 (Floor 2 call) |
| M0.6 | Check S2 sensor (Floor 2 position) |
| M0.7 | Check S3 sensor (before moving down) |
| M1.0 | Motor DOWN (moving toward Floor 2 from above) |
| M1.1 | Check S2 ON (arrived at Floor 2) |
| M1.2 | Motor UP (moving toward Floor 2 from below) |
| M1.3 | Check S2 ON (arrived at Floor 2 from below) |
| M1.4 | Wait for PB03 (Floor 3 call) |
| M1.5 | Check S3 sensor (Floor 3 position) |
| M1.6 | Motor UP (moving toward Floor 3) |
| M1.7 | Check S3 ON (arrived at Floor 3) |

---

## ⚙️ Functional Description (Network by Network)

### Network 1 — System Initialization
On the **first scan** (`SM0.1`), memory bit `M0.0` is **Set**, initializing the state machine. This ensures the system starts from a known state every time power is applied.

### Network 2 — Motor STOP State (M0.0)
When `M0.0` is active:
- **RUN (Q0.0)** output is **Reset** → Motor stops
- `M0.0` is **Reset** (self-clearing)
- `M0.1` is **Set** → Move to next state (wait for floor call)

### Network 3 — Floor 1 Button Check (M0.1)
When `M0.1` is active and **PB01 (I0.0)** is pressed:
- `M0.1` is Reset
- `M0.2` is Set → Proceed to floor 1 sensor check

If PB01 is **NOT** pressed (normally closed path):
- `M0.5` is Set → Check for Floor 2 call button instead

### Network 4 — Floor 1 Sensor Check (M0.2)
When `M0.2` is active:
- If **S1 (I0.3) is OFF** (elevator NOT at Floor 1): `M0.3` is Set → Run motor down
- If **S1 is ON** (already at Floor 1): `M0.0` is Set → Go back to STOP state (restart)

### Network 5 — Motor DOWN to Floor 1 (M0.3)
When `M0.3` is active:
- **RUN (Q0.0)** is Set → Motor enabled
- **DIR (Q0.1)** is Set → Direction = DOWN
- `M0.3` is Reset
- `M0.4` is Set → Wait for Floor 1 arrival

### Network 6 — Check Arrival at Floor 1 (M0.4)
When `M0.4` is active and **S1 (I0.3) turns ON** (arrived at Floor 1):
- `M0.0` is Set → Return to STOP state (motor stops, cycle restarts)

### Network 7 — Floor 2 Button Check (M0.5)
When `M0.5` is active and **PB02 (I0.1)** is pressed:
- `M0.6` is Set → Check Floor 2 sensor

If PB02 is **NOT** pressed:
- `M1.4` is Set → Check for Floor 3 call instead

### Network 8 — Floor 2 Sensor Check (M0.6)
When `M0.6` is active:
- If **S2 (I0.4) is ON** (already at Floor 2): `M0.0` Set → STOP
- If **S2 is OFF**: `M0.7` Set → Check S3 to determine direction

### Network 9 — Direction Decision for Floor 2 (M0.7)
When `M0.7` is active:
- If **S3 (I0.5) is ON** (elevator is above Floor 2): `M1.0` Set → Motor DOWN
- If **S3 is OFF** (elevator is below Floor 2): `M1.2` Set → Motor UP

### Network 10 — Motor DOWN to Floor 2 (M1.0)
- **RUN** Set, **DIR** Set (DOWN)
- `M1.1` Set → Wait for Floor 2 arrival sensor

### Network 11 — Check Arrival at Floor 2 from Above (M1.1)
- If **S2 ON**: `M0.0` Set → STOP

### Network 12 — Motor UP to Floor 2 (M1.2)
- **RUN** Set, **DIR Reset** (UP — DIR=0)
- `M1.3` Set → Wait for Floor 2 arrival sensor

### Network 13 — Check Arrival at Floor 2 from Below (M1.3)
- If **S2 ON**: `M0.0` Set → STOP

### Network 14 — Floor 3 Button Check (M1.4)
When `M1.4` is active and **PB03 (I0.2)** is pressed:
- `M1.5` Set → Check Floor 3 sensor

If PB03 is **NOT** pressed:
- `M0.0` Set → Return to STOP / restart

### Network 15 — Floor 3 Sensor Check (M1.5)
- If **S3 (I0.5) is OFF** (not at Floor 3): `M1.6` Set → Motor UP
- If **S3 is ON** (already at Floor 3): `M0.0` Set → STOP

### Network 16 — Motor UP to Floor 3 (M1.6)
- **RUN** Set, **DIR Reset** (UP)
- `M1.7` Set → Wait for Floor 3 arrival

### Network 17 — Check Arrival at Floor 3 (M1.7)
- If **S3 ON**: `M0.0` Set → STOP

---

## 🔁 Process Flow Summary

```
START
  │
  ▼
MOTOR STOP (M0.0)
  │
  ▼
PB01 pressed? ──No──► PB02 pressed? ──No──► PB03 pressed? ──No──► [RESTART]
  │Yes                   │Yes                   │Yes
  ▼                      ▼                      ▼
S1 OFF?              S2 ON?               S3 OFF?
  │Yes (go down)        │No (check S3)        │Yes (go up)
  ▼                     ▼                     ▼
MOTOR DOWN         S3 ON?               MOTOR UP
  │                  │Yes→DOWN │No→UP        │
  ▼                  ▼         ▼             ▼
S1 ON? → STOP    S2 ON?→STOP  S2 ON?→STOP  S3 ON? → STOP
```

---

## 🔌 TB6600 Wiring (to S7-200 Outputs)

| TB6600 Pin | Connected To | Description |
|---|---|---|
| PUL+ / PUL- | Q0.0 (RUN) | Step pulse signal |
| DIR+ / DIR- | Q0.1 (DIR) | Direction control |
| ENA+ / ENA- | 12V / GND | Enable (always ON) |
| A+, A-, B+, B- | NEMA 17 coils | Motor phases |
| VCC | 12V SMPS | Driver power |
| GND | Common GND | Common ground |

> ⚠️ **Note:** The S7-200 output signals (24V DC) may require optocoupler isolation or voltage divider when interfacing with 5V TB6600 logic inputs. Verify signal compatibility before wiring.

---

## 📁 Repository Structure

```
3-Stage-Elevator-PLC/
│
├── README.md                          ← This file
├── Flowchart/
│   └── 3-Stage_Elevator_Flowchart.pdf ← Process flow chart
├── LadderDiagram/
│   └── 3-Stage_Elevator_Ladder.pdf    ← Full PLC ladder program
├── Images/
│   ├── panel_wiring.jpg
│   ├── plc_io.jpg
│   ├── tb6600_wiring.jpg
│   └── full_setup.jpg
└── Video/
    └── demonstration.mp4              ← Live demonstration
```

---

## 📷 Project Images & Demo

> *(Add your wiring photos, PLC panel images, and sensor setup images here.)*
> *(Link your demonstration video below.)*

🎬 **Demonstration Video:** [Watch on YouTube / LinkedIn / Drive](#)

---

## 📚 Key Concepts Demonstrated

- **State Machine Programming** using M-memory bits in Siemens S7-200
- **Sequential Logic Control** — each network depends on the previous state
- **Sensor Interlocking** — proximity sensors prevent motor running past floors
- **Bidirectional Motor Control** — UP/DOWN via DIR output to TB6600
- **Ladder Diagram (LAD)** programming in STEP 7 Micro/WIN
- **Set/Reset (SR) latch logic** for stable state transitions
- **First Scan bit (SM0.1)** for safe system initialization

---

## ⚠️ Safety Considerations

- Proximity sensors act as **hardware interlocks** — elevator stops even if PLC logic fails to reset
- The `M0.0` STOP state always **Resets the RUN output** before any movement begins
- Motor direction is set **before** enabling RUN to avoid direction changes mid-move
- Emergency stop can be implemented by resetting RUN output via a hardwired relay

---

## 👤 Author

**KEERTHI P.E.D.S.**
Industrial PLC Automation
SIEMENS S7-200 | Ladder Logic | Stepper Motor Control

---

## 📄 License

This project is for educational and portfolio purposes.
Feel free to reference or build upon it with proper attribution.

