=========
Computing
=========

Computing hardware for the AV2 platform.

.. contents:: Contents
   :local:
   :depth: 2

Main Computer
=============

Overview
--------

The main computer handles all perception, planning, and high-level control tasks.

Recommended Specifications
--------------------------

+------------------+----------------------------+--------------------------------+
| Component        | Minimum                    | Recommended                    |
+==================+============================+================================+
| CPU              | Intel i5-8400 / Ryzen 5    | Intel i7-10700 / Ryzen 7       |
+------------------+----------------------------+--------------------------------+
| RAM              | 8 GB DDR4                  | 16-32 GB DDR4                  |
+------------------+----------------------------+--------------------------------+
| GPU              | NVIDIA GTX 1050 Ti (4GB)   | NVIDIA GTX 1060+ (6GB+)        |
+------------------+----------------------------+--------------------------------+
| Storage          | 256 GB SSD                 | 512 GB NVMe SSD                |
+------------------+----------------------------+--------------------------------+
| USB              | 3× USB 3.0                 | 4× USB 3.0, 2× USB 2.0         |
+------------------+----------------------------+--------------------------------+
| Ethernet         | 1× Gigabit                 | 2× Gigabit                     |
+------------------+----------------------------+--------------------------------+

Software Requirements
---------------------

.. code-block:: bash

   # Operating System
   Ubuntu 20.04 LTS or 22.04 LTS

   # NVIDIA Drivers
   nvidia-driver-525 (or newer)

   # CUDA Toolkit
   cuda-toolkit-11.8

   # Python
   Python 3.8+

Distributed CAN Control System
==============================

Overview
--------

AV2 uses a distributed embedded control architecture with multiple Teensy 4.1 microcontrollers communicating over a 250 kbps CAN bus. This modular design enables:

- Independent development and testing of each subsystem
- Fault isolation between actuators
- Real-time deterministic control loops
- Easy hardware debugging and replacement

.. code-block:: text

   ┌─────────────────────────────────────────────────────────────────────────┐
   │                    DISTRIBUTED CAN ARCHITECTURE                         │
   ├─────────────────────────────────────────────────────────────────────────┤
   │                                                                         │
   │    ┌──────────────────┐                                                │
   │    │   MAIN COMPUTER  │                                                │
   │    │   (Ubuntu/ROS)   │                                                │
   │    └────────┬─────────┘                                                │
   │             │ USB Serial (115200 baud)                                 │
   │             ▼                                                          │
   │    ┌──────────────────┐                                                │
   │    │  TEENSY 4.1      │                                                │
   │    │  CAN MASTER      │                                                │
   │    │  + Waveshare CAN │                                                │
   │    └────────┬─────────┘                                                │
   │             │ CAN Bus (250 kbps)                                       │
   │    ═════════╪═════════════════════════════════════════                 │
   │             │                                                          │
   │    ┌────────┴────────┬─────────────────┬─────────────────┐            │
   │    │                 │                 │                 │            │
   │    ▼                 ▼                 ▼                 │            │
   │ ┌──────────┐   ┌──────────┐   ┌──────────┐              │            │
   │ │ STEERING │   │ THROTTLE │   │  BRAKE   │              │            │
   │ │  NODE    │   │   NODE   │   │   NODE   │              │            │
   │ │Teensy4.1 │   │Teensy4.1 │   │Teensy4.1 │              │            │
   │ │+Waveshare│   │+Waveshare│   │+Waveshare│              │            │
   │ └────┬─────┘   └────┬─────┘   └────┬─────┘              │            │
   │      │              │              │                     │            │
   │      ▼              ▼              ▼                     │            │
   │  Stepper        MCP4728       Linear                    │            │
   │  Motor          DAC           Actuator                  │            │
   │                                                          │            │
   └─────────────────────────────────────────────────────────────────────────┘

Hardware Components
-------------------

Teensy 4.1 Microcontroller
^^^^^^^^^^^^^^^^^^^^^^^^^^

+-------------------------+--------------------------------+
| Parameter               | Value                          |
+=========================+================================+
| Processor               | ARM Cortex-M7 @ 600 MHz        |
+-------------------------+--------------------------------+
| RAM                     | 1024 KB                        |
+-------------------------+--------------------------------+
| Flash                   | 8 MB (+ microSD slot)          |
+-------------------------+--------------------------------+
| CAN Controllers         | 3× FlexCAN (CAN 2.0B)          |
+-------------------------+--------------------------------+
| USB                     | USB 2.0 (480 Mbps)             |
+-------------------------+--------------------------------+
| GPIO                    | 55 digital pins                |
+-------------------------+--------------------------------+
| ADC                     | 2× 12-bit ADC                  |
+-------------------------+--------------------------------+
| Dimensions              | 61 × 18 mm                     |
+-------------------------+--------------------------------+

Waveshare CAN Transceiver
^^^^^^^^^^^^^^^^^^^^^^^^^

Each Teensy 4.1 is paired with a Waveshare CAN transceiver module:

+-------------------------+--------------------------------+
| Parameter               | Value                          |
+=========================+================================+
| Transceiver Chip        | SN65HVD230 or MCP2551          |
+-------------------------+--------------------------------+
| Bus Voltage             | 5V tolerant                    |
+-------------------------+--------------------------------+
| Data Rate               | Up to 1 Mbps                   |
+-------------------------+--------------------------------+
| ESD Protection          | ±15 kV HBM                     |
+-------------------------+--------------------------------+

CAN Bus Configuration
---------------------

**Bit Rate**: 250 kbps

**Termination**: 120Ω resistors at each end of the bus

**Wiring**:

.. code-block:: text

   Teensy 4.1 Pin 22 (CTX1) ──► Waveshare CAN TX
   Teensy 4.1 Pin 23 (CRX1) ◄── Waveshare CAN RX
   Waveshare CAN_H ────────────► CAN Bus High
   Waveshare CAN_L ────────────► CAN Bus Low

CAN Message Protocol
====================

Message IDs
-----------

+------------------+------------+-------------------+
| Subsystem        | Command ID | Status ID         |
+==================+============+===================+
| Throttle         | 0x100      | 0x101             |
+------------------+------------+-------------------+
| Steering         | 0x200      | 0x201             |
+------------------+------------+-------------------+
| Brake            | 0x300      | 0x301             |
+------------------+------------+-------------------+

Throttle Messages (0x100)
-------------------------

**Command Frame** (ID: 0x100, 3 bytes):

+--------+----------------+---------------------------+
| Byte   | Value          | Description               |
+========+================+===========================+
| 0      | 0 or 1         | E-STOP flag               |
+--------+----------------+---------------------------+
| 1      | 0-255          | Throttle (0-100%)         |
+--------+----------------+---------------------------+
| 2      | 'N','D','S','R'| Drive mode                |
+--------+----------------+---------------------------+

**Drive Modes**:

- ``N`` - Neutral
- ``D`` - Drive
- ``S`` - Sport
- ``R`` - Reverse

Steering Messages (0x200)
-------------------------

**Command Frame** (ID: 0x200, 2 bytes):

+--------+----------------+---------------------------+
| Byte   | Value          | Description               |
+========+================+===========================+
| 0      | 0 or 1         | Center flag (recenter)    |
+--------+----------------+---------------------------+
| 1      | -127 to +127   | Steering (-1.0 to +1.0)   |
+--------+----------------+---------------------------+

**Steering Convention**:

- Negative values = Left
- Positive values = Right
- 0 = Center

Brake Messages (0x300)
----------------------

**Command Frame** (ID: 0x300, 2 bytes):

+--------+----------------+---------------------------+
| Byte   | Value          | Description               |
+========+================+===========================+
| 0      | 0 or 1         | E-STOP flag               |
+--------+----------------+---------------------------+
| 1      | 0-255          | Brake pressure (0-100%)   |
+--------+----------------+---------------------------+

Master Node (CAN Master)
========================

The Master node receives commands from the main computer via USB serial and broadcasts them to the actuator nodes over CAN.

Serial Protocol
---------------

**Baud Rate**: 115200

**Command Format**: ASCII text with newline terminator

**Commands**:

+----------+------------------+---------------------------+
| Command  | Format           | Description               |
+==========+==================+===========================+
| Throttle | ``T <value>``    | value: 0.0 to 1.0         |
+----------+------------------+---------------------------+
| Brake    | ``B <value>``    | value: 0.0 to 1.0         |
+----------+------------------+---------------------------+
| Steering | ``S <value>``    | value: -1.0 to 1.0        |
+----------+------------------+---------------------------+
| Mode     | ``M <mode>``     | mode: N, D, S, R          |
+----------+------------------+---------------------------+
| E-Stop   | ``E <0/1>``      | 0: release, 1: engage     |
+----------+------------------+---------------------------+

**Examples**:

.. code-block:: text

   T 0.5       # Set throttle to 50%
   S -0.25     # Steer 25% left
   B 1.0       # Full brake
   M D         # Drive mode
   E 1         # Emergency stop

Firmware
--------

The Master node firmware is implemented using the FlexCAN_T4 library:

.. code-block:: cpp

   #include <FlexCAN_T4.h>

   #define CAN_BITRATE     250000
   #define THROTTLE_ID     0x100
   #define STEER_ID        0x200
   #define BRAKE_ID        0x300

   FlexCAN_T4<CAN1, RX_SIZE_256, TX_SIZE_16> Can1;

   void setup() {
       Serial.begin(115200);
       Can1.begin();
       Can1.setBaudRate(CAN_BITRATE);
   }

Safety Features
===============

Watchdog Timers
---------------

Each actuator node implements a watchdog timer:

- **Throttle**: 200ms timeout → Falls back to Neutral + Idle
- **Steering**: No timeout (holds last position)
- **Brake**: 100ms timeout → Applies full brake

Emergency Stop
--------------

When E-STOP is activated:

1. Brake node applies full braking force
2. Throttle node sets output to 0V and enters Neutral
3. Steering node centers (optional)

All CAN command frames include an E-STOP byte for fail-safe operation.

Development Environment
=======================

PlatformIO Setup
----------------

.. code-block:: ini

   ; platformio.ini
   [env:teensy41]
   platform = teensy
   board = teensy41
   framework = arduino
   lib_deps =
       FlexCAN_T4
       Adafruit MCP4728

Arduino IDE Setup
-----------------

1. Install Teensyduino add-on
2. Select Board: "Teensy 4.1"
3. Install FlexCAN_T4 library

Flashing Firmware
-----------------

.. code-block:: bash

   # Using PlatformIO
   pio run -t upload

   # Using Teensy Loader
   teensy_loader_cli --mcu=TEENSY41 -w firmware.hex

Power Requirements
==================

+------------------+---------------+---------------+
| Component        | Voltage       | Current       |
+==================+===============+===============+
| Main Computer    | 12V / 19V     | 5-15 A        |
+------------------+---------------+---------------+
| Teensy 4.1 (×4)  | 5V (USB)      | 100 mA each   |
+------------------+---------------+---------------+
| Waveshare CAN    | 5V            | 50 mA each    |
+------------------+---------------+---------------+
| MCP4728 DAC      | 3.3V / 5V     | 10 mA         |
+------------------+---------------+---------------+

See :doc:`power-system` for complete power distribution details.
