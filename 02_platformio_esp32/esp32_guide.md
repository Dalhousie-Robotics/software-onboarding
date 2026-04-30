# ESP32 & PlatformIO Guide

This guide teaches you how to write, build, and flash firmware to the ESP32, the microcontroller that controls the robot's motors in real time.

---

## What is the ESP32?

The ESP32 is a microcontroller, a tiny computer that runs a single program in a loop, starting the moment it powers on. It has:
- Dual-core 240 MHz processor
- Built-in WiFi and Bluetooth
- A built-in **CAN bus peripheral** (called TWAI), this is how it talks to the motors
- ~520 KB of RAM (tiny compared to a PC, be careful with memory)

Our firmware is written in **C++** (not Python). C++ gives us direct hardware access and the speed needed to run a control loop at 500–1000 times per second.

---

## What is PlatformIO?

PlatformIO is a build system and package manager for embedded systems. It lives inside VS Code as an extension. It handles:
- Compiling C++ for the ESP32
- Downloading the right toolchain automatically
- Flashing (uploading) firmware to the board over USB
- Running unit tests

---

## Project Structure

The firmware lives at `quadruped/src/firmware/`. Here's what each file does:

```
src/firmware/
├── platformio.ini          Project configuration (board, framework, libraries)
├── src/
│   └── main.cpp            Entry point: setup() runs once, loop() runs forever
└── lib/
    └── ak40_driver/        Our motor driver library
        ├── ak40_driver.h   Class declaration (what functions exist)
        └── ak40_driver.cpp Class implementation (how they work)
```

---

## Your First Program: Blink an LED

Before touching motor code, verify your toolchain works with a simple LED blink.

### 1. Open the firmware folder in VS Code

```
cd quadruped/src/firmware
code .
```

### 2. Create a test project (PlatformIO Home → New Project)

Or, if working directly in the team repo, create a temporary test file.

### 3. Write the blink program

Replace the contents of `src/main.cpp` **temporarily** with:

```cpp
#include <Arduino.h>

#define LED_PIN 2  // Built-in LED on most ESP32 boards

void setup() {
    Serial.begin(115200);
    pinMode(LED_PIN, OUTPUT);
    Serial.println("Blink test running");
}

void loop() {
    digitalWrite(LED_PIN, HIGH);
    delay(500);
    digitalWrite(LED_PIN, LOW);
    delay(500);
}
```

### 4. Build (compile without uploading)

In VS Code: click the **checkmark icon** at the bottom toolbar (PlatformIO: Build).

Or in the terminal:
```
pio run
```

You should see: `SUCCESS` at the end with no errors.

### 5. Upload to the ESP32

Plug your ESP32 into your PC via USB.

```
pio run -t upload
```

PlatformIO detects the COM port automatically on Windows. The LED on the board should start blinking.

### 6. Open the Serial Monitor

```
pio device monitor
```

You should see `Blink test running` printed. Press Ctrl+C to exit.

**If you see this, your toolchain is working correctly.**

---

## Understanding the Real Firmware

Now read through `src/main.cpp` and `lib/ak40_driver/ak40_driver.h` in the actual repo.

Key things to understand:

### setup() and loop()

```cpp
void setup() {
    // Runs ONCE at power-on or reset
    // Initialize hardware here
}

void loop() {
    // Runs FOREVER, as fast as possible (or with delays)
    // This is the control loop
}
```

### The TWAI (CAN) peripheral

The ESP32's built-in CAN controller is called **TWAI**. It requires an external **CAN transceiver IC** (e.g., SN65HVD230) between the ESP32 GPIO pins and the CAN bus wires. The firmware handles the software side; Electrical team handles the transceiver circuit.

### Header files (.h) vs implementation files (.cpp)

- `.h` files declare **what** exists (function signatures, class definitions)
- `.cpp` files define **how** it works (actual code)
- Other files `#include` the `.h` to use the functionality

---

## Exercise

1. Build the real firmware (`pio run`), it should compile cleanly.
2. Read through `ak40_driver.h` and write a short comment next to each function explaining what it does in plain English.
3. Identify: where would you add code to send a command to motor ID 1?

---

## Common Errors

| Error | Cause | Fix |
|---|---|---|
| `COM port not found` | USB driver issue or wrong cable | Try a different USB cable; install CP2102 or CH340 driver |
| `Error: Please specify upload_port` | PlatformIO can't find the board | Set `upload_port = COMx` in `platformio.ini` |
| `undefined reference to ...` | Missing `.cpp` file or library | Check `lib_deps` in `platformio.ini` |
| `task exceeds available stack` | Stack overflow in ESP32 task | Reduce local variable sizes; use static allocation |

---

**Next step:** [03: Python & MuJoCo Simulation](../03_python_simulation/python_mujoco_guide.md)
