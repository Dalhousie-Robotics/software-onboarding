# STM32 & PlatformIO Guide

This guide teaches you how to write, build, and flash firmware to the STM32F446RE, the microcontroller that controls the robot's motors in real time over CAN bus.

---

## What is the STM32F446RE?

The STM32F446RE is a microcontroller made by STMicroelectronics. We use it on the **ST Nucleo F446RE** development board, which has an on-board ST-Link debugger -- this means you can flash and debug the chip with a single USB cable, no external programmer needed.

Key specs:
- 180 MHz ARM Cortex-M4 processor with hardware FPU
- Built-in **bxCAN** peripheral (CAN 2.0B), this is how it talks to the motors
- 512 KB flash, 128 KB RAM
- ST-Link/V2 on-board (flash over USB, debug with breakpoints)
- USB Virtual COM port (how it talks to your PC over serial)

Our firmware is written in **C++** (not Python). C++ gives us direct hardware access and the speed needed to run a control loop at 500-1000 Hz.

---

## What is PlatformIO?

PlatformIO is a build system and package manager for embedded systems. It lives inside VS Code as an extension. It handles:
- Compiling C++ for the STM32
- Downloading the right toolchain and libraries automatically
- Flashing firmware to the board via ST-Link over USB
- Running the serial monitor so you can read debug output

We use the **stm32duino Arduino framework**, which means you write `setup()` and `loop()` like Arduino, but targeting the STM32.

---

## Hardware You Need

- ST Nucleo F446RE board
- USB-A to Mini-B cable (the Nucleo uses Mini-B USB, not Micro)
- CAN transceiver module (e.g. SN65HVD230 breakout) -- provided by the Electrical team

> You can build and flash firmware without the CAN transceiver. The firmware will report an error on the serial monitor but it will compile and run.

---

## Project Structure

The firmware lives at `quadruped/src/firmware/`. Here is what each file does:

```
src/firmware/
+-- platformio.ini          Project config (board, framework, libraries)
+-- src/
|   +-- main.cpp            Entry point: setup() once, loop() forever
+-- lib/
    +-- ak40_driver/        Motor driver library
        +-- ak40_driver.h   Class declaration
        +-- ak40_driver.cpp Class implementation
```

---

## Step 1: Install the PlatformIO Extension

1. Open VS Code
2. Click the Extensions icon (Ctrl+Shift+X)
3. Search for **PlatformIO IDE**
4. Click Install
5. Restart VS Code when prompted

PlatformIO will download the STM32 toolchain the first time you build. This can take a few minutes.

---

## Step 2: Open the Firmware Folder

In VS Code: File > Open Folder > navigate to `quadruped/src/firmware` and click Select Folder.

You should see the PlatformIO toolbar at the bottom of the screen (checkmark, right arrow, plug icons).

---

## Step 3: Build the Firmware

Click the **checkmark icon** at the bottom toolbar (PlatformIO: Build), or run in the terminal:

```
pio run
```

A successful build ends with:

```
=== [SUCCESS] Took X.XX seconds ===
```

If you see errors, check that `platformio.ini` lists the correct board (`nucleo_f446re`) and that the `stm32duino/STM32duino CAN` library is in `lib_deps`.

---

## Step 4: Flash to the Board

Plug the Nucleo into your PC with the Mini-B USB cable. Windows should install the ST-Link driver automatically. If not, install [ST-Link drivers](https://www.st.com/en/development-tools/stsw-link009.html) from ST's website.

Click the **right arrow icon** (PlatformIO: Upload), or:

```
pio run -t upload
```

You should see:

```
** Programming Finished **
** Verify OK **
```

The LED on the Nucleo board will blink briefly during upload.

---

## Step 5: Open the Serial Monitor

```
pio device monitor
```

Or click the **plug icon** in the PlatformIO toolbar.

After the STM32 boots (about 2-3 seconds for USB enumeration), you should see:

```
[boot] Dal Robotics Quadruped Firmware
[boot] STM32F446RE @ 1 Mbit/s CAN
[ok] CAN bus ready
[ok] Waiting for commands from PC
```

If you see `[error] CAN init failed`, the bxCAN peripheral could not start. Check that the CAN transceiver is wired to PB8 (RX) and PB9 (TX).

Press Ctrl+C to exit the monitor.

---

## Understanding the Real Firmware

Read through `src/main.cpp` and `lib/ak40_driver/ak40_driver.h`.

### setup() and loop()

```cpp
void setup() {
    // Runs ONCE at power-on or reset.
    // Initialize peripherals here.
}

void loop() {
    // Runs FOREVER after setup() finishes.
    // This is where commands are parsed and forwarded to motors.
}
```

### The bxCAN peripheral

The STM32F446RE has a built-in CAN 2.0B controller called **bxCAN**. It requires an external **CAN transceiver IC** (e.g. SN65HVD230 or TJA1050) between the STM32 GPIO pins and the physical CAN bus wires (CANH/CANL). The firmware handles the software side; the Electrical team designs the transceiver circuit.

CAN1 pins on the Nucleo F446RE:
- **RX:** PB8 (Arduino D15)
- **TX:** PB9 (Arduino D14)

### The STM32duino CAN library

We use `stm32duino/STM32duino CAN` which wraps the HAL CAN API into an Arduino-style interface:

```cpp
CAN.begin(1000000);        // start at 1 Mbit/s
CAN.beginPacket(motorId);  // address a motor
CAN.write(data, 8);        // write 8 bytes
CAN.endPacket();           // transmit

int size = CAN.parsePacket();  // check for incoming frame
if (size > 0) {
    uint8_t byte = CAN.read();  // read one byte
}
```

### Header files (.h) vs implementation files (.cpp)

- `.h` files declare **what** exists (function signatures, class definitions)
- `.cpp` files define **how** it works (actual code)
- Other files `#include` the `.h` to use the class

---

## Exercise

1. Build the firmware (`pio run`) -- it should compile cleanly.
2. Flash it and open the serial monitor. Read the boot messages.
3. Read `ak40_driver.h` and write a short comment next to each function explaining what it does in plain English.
4. Find the line in `main.cpp` where it parses a `CMD` message. Trace the data path from that line to the CAN frame being sent.

---

## Common Errors

| Error | Cause | Fix |
|---|---|---|
| `COM port not found` | ST-Link driver not installed | Install ST-Link drivers from ST's website |
| `Error: Please specify upload_port` | PlatformIO can't detect board | Set `upload_port = COMx` in `platformio.ini` |
| `[error] CAN init failed` | No CAN transceiver or wrong pins | Check transceiver wiring to PB8/PB9 |
| `undefined reference to CAN` | Library not installed | Run `pio lib install` or check `lib_deps` in `platformio.ini` |
| Serial monitor shows nothing | USB CDC not enumerated | Wait 3 seconds after plugging in; try a different USB port |

---

**Next step:** [03: Python & MuJoCo Simulation](../03_python_simulation/python_mujoco_guide.md)
