# STM32 Deterministic GPS Telemetry Node

**Status:** Completed and hardware-tested  
**Platform:** STM32F446RE (Nucleo)  
**Architecture:** Interrupt-driven UART reception + ring buffer telemetry pipeline

---

# What It Does

This project implements a real-time GPS telemetry system using the **NEO-6M GPS module** and the **STM32F446RE**.

The firmware continuously receives GPS data over UART using an interrupt-driven architecture, reconstructs NMEA sentences from a streaming byte stream, parses GPS fields, and outputs structured telemetry data over a secondary UART interface.

The system was designed to explore how embedded firmware handles:

- continuous serial data streams
- asynchronous communication
- ring buffer architectures
- protocol parsing
- deterministic ISR behavior
- real-time telemetry pipelines

---

# System Architecture

The firmware is structured as a streaming telemetry pipeline:

```text
NEO-6M GPS Module
        ↓
USART1 RX Interrupt
        ↓
Ring Buffer
        ↓
Line Reconstruction
        ↓
NMEA Sentence Parser
        ↓
Coordinate Extraction
        ↓
Structured Telemetry Output
        ↓
USART2 Debug Terminal
```

This architecture separates:

- byte reception
- buffering
- parsing
- telemetry formatting

into independent stages.

---

# Hardware Used

- **MCU:** STM32F446RE (Nucleo)
- **GPS Module:** NEO-6M
- **Communication:** UART
- **Debug Output:** USART2 Serial Terminal
- **Debugger:** ST-Link

---

# UART Architecture

The system uses two UART peripherals:

| Peripheral | Purpose |
|---|---|
| USART1 | GPS data reception |
| USART2 | Debug / telemetry output |

This creates a clean separation between:

- incoming telemetry stream
- outgoing formatted data

---

# Interrupt-Driven Reception

GPS data is received using:

```c
HAL_UART_Receive_IT()
```

The UART ISR captures incoming bytes one at a time and immediately stores them into a ring buffer.

This avoids:

- blocking reads
- polling loops
- lost serial data during processing

---

# Ring Buffer Design

Incoming GPS bytes are stored in a circular ring buffer.

Core concepts implemented:

- head/tail indexing
- wraparound behavior
- producer-consumer architecture
- asynchronous stream handling

This allows UART interrupts to continuously receive data while the main firmware processes completed messages independently.

---

# NMEA Sentence Reconstruction

GPS modules transmit ASCII NMEA sentences continuously.

The firmware reconstructs complete lines by:

1. receiving bytes individually
2. detecting newline termination
3. storing completed sentences into a line buffer

Example NMEA sentence:

```text
$GPGGA,123519,4807.038,N,01131.000,E,...
```

This stage converts a raw byte stream into structured protocol messages.

---

# GPS Parsing Logic

The parser extracts GPS information from NMEA sentences including:

- latitude
- longitude
- fix information
- UTC time

String tokenization and field extraction are used to isolate individual GPS parameters.

---

# Coordinate Conversion

Raw GPS coordinates are converted into human-readable decimal format.

This required understanding:

- degree-minute representation
- floating-point conversion
- coordinate formatting

---

# Real-Time Telemetry Output

Parsed telemetry data is transmitted over USART2 in a structured format for monitoring and debugging.

Example output:

```text
Latitude : xx.xxxxxx
Longitude: yy.yyyyyy
Fix      : Valid
```

---

# Core Concepts Demonstrated

This project demonstrates:

- interrupt-driven UART reception
- asynchronous serial communication
- ring buffer architecture
- producer-consumer data flow
- stream-oriented parsing
- NMEA protocol handling
- real-time telemetry systems
- multi-UART embedded design
- deterministic ISR behavior
- non-blocking firmware architecture

---

# Design Philosophy

The project was intentionally designed around deterministic embedded communication principles:

- ISR performs minimal work
- parsing occurs outside interrupts
- buffering decouples reception from processing
- communication remains non-blocking

This mirrors real embedded telemetry systems used in:

- GPS trackers
- robotics
- autonomous systems
- vehicle telemetry
- IoT monitoring systems

---

# Repository Structure

```text
STM32-Deterministic-GPS-Telemetry-Node/
│
├── Core/
│   ├── Inc/
│   │   └── Neo_6m.h
│   │
│   └── Src/
│       ├── main.c
│       └── Neo_6m.c
│
├── Drivers/
│
├── STM32-Deterministic-GPS-Telemetry-Node.ioc
├── STM32F446RETX_FLASH.ld
├── STM32F446RETX_RAM.ld
│
└── README.md
```

---

# Tools & Environment

- STM32CubeIDE
- STM32 HAL
- Embedded C
- Linux development environment
- Git & GitHub

---

# Applications

Architectures like this are commonly used in:

- GPS tracking systems
- autonomous robots
- telemetry nodes
- fleet monitoring systems
- navigation systems
- outdoor IoT devices

---

# Status

Fully functional GPS telemetry system running on STM32F446RE using interrupt-driven UART communication and structured NMEA parsing.
