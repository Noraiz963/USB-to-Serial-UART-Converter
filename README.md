# USB to Serial (UART) Converter

A USB Type-C to Serial (UART) converter designed in KiCad, built around the CP2102 USB-to-UART bridge IC — lets you connect a microcontroller's serial pins to a computer's USB port for programming, debugging, or serial communication.

## Why This Exists

Modern PCs only expose USB — there's no native way to talk to a device that "speaks" UART (TX/RX serial). The CP2102 sits in between and acts as a protocol translator: it handles USB enumeration and packetization on one side, and generates real asynchronous serial (UART) signals on the other. Once connected, the PC's OS loads it as a **Virtual COM Port** (e.g. `COM5` on Windows, `/dev/ttyUSB0` on Linux), so any serial terminal or IDE (Arduino Serial Monitor, PuTTY, minicom) can talk to the target microcontroller exactly as if it were plugged into a physical RS-232 port.

## Circuit Overview

- **USB Type-C input**: USB-C receptacle (J1) with CC1/CC2 pull-down resistors for proper USB-C power delivery negotiation, plus ESD protection
- **USB-to-UART bridge**: CP2102 (U1) handles the USB-to-serial conversion, exposing TX/RX lines on the output header
- **Status indicator**: LED with current-limiting resistor, indicating activity/power
- **Decoupling**: Capacitors placed across VBUS/VDD to stabilize the CP2102's supply
- **Reset**: RST line broken out to the output header for manual reset control
- **Output header**: 5-pin header exposing RST, TX, RX, VBUS, and GND — standard pinout for connecting to microcontroller boards (e.g. ESP32, STM32, Arduino) for flashing or serial monitoring

## Signal Flow (Concept)

**PC → Target device (TX path):**
1. PC sends data over USB D+/D− as standard USB packets.
2. CP2102's internal USB engine strips the packet down to raw bytes.
3. CP2102 serializes each byte into UART bits (start bit, 8 data bits, stop bit) at the configured baud rate.
4. Output appears on the CP2102's **TX pin**, routed to J2, then into the target device's **RX pin**.

**Target device → PC (RX path):**
1. Target device sends serial data out its **TX pin** into the CP2102's **RX pin** (via J2).
2. CP2102 samples the incoming UART bitstream and reconstructs it into bytes.
3. Those bytes are packaged into USB packets by the CP2102's internal USB controller.
4. Packets are sent back to the PC over D+/D−, where the VCP driver hands them to whatever serial application is listening.

**Power path:**
- USB-C VBUS (5V) enters through J1 → CC resistors confirm valid power negotiation → CP2102's internal LDO regulates it to 3.3V for internal logic → VBUS is also broken out on J2 to optionally power the target board directly from USB.

**Ground referencing:**
- GND is shared across the USB connector, the CP2102, and the output header. This is critical — UART logic levels (HIGH/LOW) are only meaningful relative to a common ground; without it, the target device can't correctly interpret the TX/RX voltage transitions.

**Reset behavior:**
- RST is pulled high by default (idle/running state) via a pull-up resistor. Pulling it low resets the CP2102. Broken out to J2 so it can optionally be wired to a target MCU's reset line for auto-reset-on-connect behavior (common on Arduino-style boards).

## Schematic
![Schematic](https://github.com/Noraiz963/USB-to-Serial-UART-Converter/blob/7509748a3ca0dd751ea844258050f70bc783c72d/sch.png)

## PCB Layout
![PCB Layout](https://github.com/Noraiz963/USB-to-Serial-UART-Converter/blob/7509748a3ca0dd751ea844258050f70bc783c72d/PCB.png)

## 3D Render
![3D Render](https://github.com/Noraiz963/USB-to-Serial-UART-Converter/blob/7509748a3ca0dd751ea844258050f70bc783c72d/3rd.png)

## Bill of Materials

| Ref | Component | Value |
|-----|-----------|-------|
| J1 | USB Type-C Receptacle | — |
| U1 | USB-to-UART Bridge IC | CP2102 |
| D1, D2 | LED / Protection Diode | — |
| R1–R7 | Resistor | Various (pull-downs, current limiting) |
| C1, C3, C4, C10, C11 | Capacitor | Decoupling |
| J2 | Header (5-pin) | RST / TX / RX / VBUS / GND |

## Output Header Pinout (J2)

| Pin | Signal | Direction (from board) | Notes |
|-----|--------|-------------------------|-------|
| 1 | RST | — | Optional reset control for target MCU |
| 2 | TX | Out | Connect to target device's RX |
| 3 | RX | In | Connect to target device's TX |
| 4 | VBUS | Out | 5V from USB, optional power for target |
| 5 | GND | — | Common ground reference |

> ⚠️ Always cross-connect TX ↔ RX between this board and the target device — never TX to TX.

## Tools Used
- KiCad 10.0 (schematic capture, PCB layout, 3D visualization)

## Status
Completed — fifth PCB design project, first design centered on a USB interface IC rather than power regulation or timer-based circuits.

## Notes
Built as a learning project to understand USB-C connector wiring (CC resistor requirements) and USB-to-serial bridge IC integration — a practical, reusable tool for future embedded projects that need programming/debugging headers.
