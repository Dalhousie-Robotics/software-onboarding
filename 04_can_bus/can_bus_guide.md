# CAN Bus & Motor Protocol Guide

This guide explains how the robot's motors communicate, you don't need to implement this yourself right away, but every team member must understand it to work on the system.

---

## What is CAN Bus?

CAN (Controller Area Network) is a communication protocol designed for industrial and automotive systems where many devices need to talk to each other reliably and quickly.

Think of it as a **shared cable** that all devices on the robot can listen to and speak on:

```
ESP32 ──── CAN transceiver ─────────────────────────────────┐
                                    CAN bus (2 wires)        │
Motor 1 ── CAN transceiver ────────────────────────────────┤
Motor 2 ── CAN transceiver ────────────────────────────────┤
Motor 3 ── CAN transceiver ────────────────────────────────┤
...                                                         │
Motor 12 ─ CAN transceiver ─────────────────────────────────┘
```

Every device has a **unique ID** (1–12 for our motors). A device only acts on messages addressed to its ID, others are ignored.

### Why CAN and not USB or WiFi?
- **Deterministic timing**: messages arrive in a guaranteed order with guaranteed latency
- **Noise resistant**: uses differential signaling (2 wires, opposite signals); immune to electrical noise from motors
- **Multi-master**: any device can send without a central coordinator
- **1 Mbit/s**: fast enough for 1000 Hz control loops with 12 motors

---

## The AK40-10 Motor

The CubeMars AK40-10 is a brushless servo actuator with:
- 10:1 gear ratio
- Peak torque: ~40 Nm
- Built-in encoder (position feedback)
- Built-in CAN transceiver (no external transceiver needed on the motor side)

It supports three control modes:
| Mode | What you send | Use case |
|---|---|---|
| Position | Target angle | Slow, precise movements |
| Velocity | Target speed | Continuous rotation |
| **MIT mode** | (pos, vel, kp, kd, torque_ff) | Our primary mode, torque control |

MIT mode is the most powerful. You send a full state: where you want the joint to be, how fast, and the stiffness of the "spring" (kp/kd). This lets you control both position AND compliance simultaneously.

---

## The MIT Mini Cheetah CAN Protocol

The AK40-10 uses the same protocol as the MIT Mini Cheetah, an open protocol designed for high-performance legged robots.

### Command frame (ESP32 → Motor)

Every command is exactly **8 bytes**:

```
Byte [0:1]   position target     16-bit uint, maps to -12.5 to +12.5 radians
Byte [2]     velocity (high)     upper 8 bits of 12-bit uint
Byte [3]     vel(low) + kp(hi)   lower 4 bits vel | upper 4 bits kp
Byte [4]     kp (low)            lower 8 bits of 12-bit kp
Byte [5]     kd (high)           upper 8 bits of 12-bit kd
Byte [6]     kd(low) + t(hi)     lower 4 bits kd | upper 4 bits torque_ff
Byte [7]     torque_ff (low)     lower 8 bits of 12-bit torque_ff
```

This packing is tight because CAN frames are limited to 8 bytes.

### Response frame (Motor → ESP32)

The motor replies to every command:

```
Byte [0]     Motor ID
Byte [1:2]   Measured position   (same 16-bit mapping)
Byte [3:4]   Measured velocity   (12-bit mapping)
Byte [5:6]   Measured torque     (12-bit mapping)
Byte [7]     Temperature + error flags
```

### Special commands

Before sending normal commands, you must put the motor in MIT mode:

| Action | Bytes to send |
|---|---|
| Enter MIT mode | `FF FF FF FF FF FF FF FC` |
| Exit MIT mode | `FF FF FF FF FF FF FF FD` |
| Set zero position | `FF FF FF FF FF FF FF FE` |

---

## R-Link: Motor Configuration Tool

The R-Link is a USB-to-CAN adapter from CubeMars. You use it with their Windows software to:
- Set motor CAN IDs (each motor must have a unique ID 1–12)
- Tune PID gains stored on the motor
- Test individual motors manually
- Update motor firmware

**You use R-Link for configuration only.** During normal robot operation, the ESP32 handles all CAN communication directly.

### Setting Motor CAN IDs (important one-time setup)

Each motor ships with ID 1 by default. You must set unique IDs before connecting them all to the same bus:

1. Connect only ONE motor to the R-Link
2. Open the CubeMars R-Link software
3. Go to Settings → Motor ID
4. Set the ID for that motor (1–12)
5. Disconnect, repeat for the next motor

**Never have two motors with the same ID on the same bus, they will conflict and corrupt communication.**

---

## How the Control Loop Works

At a high level, every millisecond the ESP32 does this:

```
1. Receive target joint angles from PC (over Serial/WiFi)
2. For each of 12 motors:
   a. Pack target into 8-byte MIT command
   b. Send CAN frame to that motor ID
   c. Read back 8-byte response
   d. Unpack measured position, velocity, torque
3. Send all 12 joint states back to PC
4. Repeat
```

This loop runs at 500–1000 Hz. At 1000 Hz, you have 1 millisecond per loop, every line of code in that loop must be fast.

---

## Exercise

1. Look at `quadruped/src/firmware/lib/ak40_driver/ak40_driver.cpp` and trace through `send_command()`. Match each byte of the frame to the protocol table above.
2. Calculate: if the control loop runs at 500 Hz with 12 motors, how many CAN frames are sent per second?
3. What happens if you call `send_command()` but the motor is not in MIT mode?

---

You now understand the full communication chain. When you're ready to work on firmware or ask questions in meetings, this is the foundation everything else builds on.
