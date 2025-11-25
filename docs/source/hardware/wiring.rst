======
Wiring
======

Wiring diagrams and connector pinouts for the AV2 platform.

.. contents:: Contents
   :local:
   :depth: 2

System Wiring Overview
======================

.. code-block:: text

   ┌────────────────────────────────────────────────────────────────────────────┐
   │                         AV2 WIRING OVERVIEW                                │
   ├────────────────────────────────────────────────────────────────────────────┤
   │                                                                            │
   │    ┌─────────────┐                                                         │
   │    │   XSENS     │◄─── USB Cable ──────────────┐                          │
   │    │   MTi-630   │                              │                          │
   │    └─────────────┘                              │                          │
   │                                                 │                          │
   │    ┌─────────────┐                              │    ┌─────────────────┐  │
   │    │  VELODYNE   │◄─── Ethernet Cat6 ──────────┼───►│                 │  │
   │    │   VLP-16    │◄─── 12V Power ──────┐       │    │      MAIN       │  │
   │    └─────────────┘                     │       │    │    COMPUTER     │  │
   │                                        │       │    │                 │  │
   │    ┌─────────────┐                     │       │    │  ┌───────────┐  │  │
   │    │  REALSENSE  │◄─── USB 3.0 ────────┼───────┼───►│  │ USB Hub   │  │  │
   │    │    D435     │                     │       │    │  └───────────┘  │  │
   │    └─────────────┘                     │       │    │                 │  │
   │                                        │       │    │  ┌───────────┐  │  │
   │    ┌─────────────┐                     │       └───►│  │  Ethernet │  │  │
   │    │ USB CAMERAS │◄─── USB 2.0 ────────┼───────────►│  │   Port    │  │  │
   │    └─────────────┘                     │            │  └───────────┘  │  │
   │                                        │            │                 │  │
   │    ┌─────────────┐                     │            └────────┬────────┘  │
   │    │ CAN MASTER  │◄─── USB Serial ─────┼─────────────────────┘           │
   │    │ (Teensy4.1) │                     │                                  │
   │    └──────┬──────┘                     │                                  │
   │           │ CAN Bus (250 kbps)         │                                  │
   │    ═══════╪═══════════════════════════════════════════                   │
   │           │              │              │              │                  │
   │    ┌──────┴──────┐ ┌─────┴─────┐ ┌─────┴─────┐ ┌─────┴─────┐            │
   │    │  STEERING   │ │ THROTTLE  │ │   BRAKE   │ │   SPARE   │            │
   │    │   NODE      │ │   NODE    │ │   NODE    │ │   NODE    │            │
   │    │ (Teensy4.1) │ │(Teensy4.1)│ │(Teensy4.1)│ │ (future)  │            │
   │    └──────┬──────┘ └─────┬─────┘ └─────┬─────┘ └───────────┘            │
   │           │              │              │                                 │
   │           ▼              ▼              ▼                                 │
   │       Stepper        MCP4728       Linear                                │
   │       Motor +         DAC         Actuator                               │
   │       Feedback                                                            │
   │                                                                            │
   │                                              ┌────────────────────────┐  │
   │                                              │   POWER DISTRIBUTION   │  │
   │                                         ┌───►│         BOARD          │  │
   │                                         │    │                        │  │
   │                                         │    │  12V ──► Sensors       │  │
   │                                         │    │   5V ──► USB/Teensy    │  │
   │                                         │    │  19V ──► Computer      │  │
   │                                         │    │                        │  │
   │                                         │    │         ▲              │  │
   │                                         │    │    ┌────┴────┐         │  │
   │                                         │    │    │ AUX     │         │  │
   │                                         │    │    │ BATTERY │         │  │
   │                                         │    │    │  12V    │         │  │
   │                                         │    │    └─────────┘         │  │
   │                                         │    └────────────────────────┘  │
   │                                         │                                 │
   └─────────────────────────────────────────┴─────────────────────────────────┘

Xsens MTi-630 Wiring
====================

**Connection**: USB Mini-B or 10-pin connector

USB Connection (Recommended)
----------------------------

.. code-block:: text

   Xsens USB Port ◄──── USB Mini-B Cable ────► Computer USB Port

10-Pin Connector
----------------

+------+--------+-------------+
| Pin  | Color  | Signal      |
+======+========+=============+
| 1    | Red    | VCC (5V)    |
+------+--------+-------------+
| 2    | Black  | GND         |
+------+--------+-------------+
| 3    | Green  | TX (out)    |
+------+--------+-------------+
| 4    | White  | RX (in)     |
+------+--------+-------------+
| 5-10 | -      | Reserved    |
+------+--------+-------------+

Velodyne VLP-16 Wiring
======================

Interface Box Connections
-------------------------

**Power Input**:

+--------+-------------+
| Wire   | Connection  |
+========+=============+
| Red    | +12V        |
+--------+-------------+
| Black  | GND         |
+--------+-------------+

**Ethernet**:

Standard Cat6 cable to computer Ethernet port.

**GPS Input** (Optional):

+------+-------------+
| Pin  | Signal      |
+======+=============+
| 1    | PPS Input   |
+------+-------------+
| 2    | GND         |
+------+-------------+
| 3    | GPS NMEA RX |
+------+-------------+

Intel RealSense Wiring
======================

**Connection**: USB 3.0 Type-C

.. code-block:: text

   RealSense USB-C ◄──── USB-C to USB-A 3.0 Cable ────► Computer USB 3.0 Port

.. note::

   Use a high-quality USB 3.0 cable. Poor cables cause bandwidth issues and dropped frames.

Distributed CAN Bus Wiring
==========================

The AV2 uses a distributed CAN bus architecture with multiple Teensy 4.1 nodes, each paired with a Waveshare CAN transceiver.

CAN Bus Topology
----------------

.. code-block:: text

   120Ω                                                                  120Ω
    ║                                                                      ║
   ─┼──────────────────────────────────────────────────────────────────────┼─ CAN_H
    │         │              │              │              │               │
    │    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐         │
    │    │Waveshare│    │Waveshare│    │Waveshare│    │Waveshare│         │
    │    │  CAN    │    │  CAN    │    │  CAN    │    │  CAN    │         │
    │    │(Master) │    │(Steer)  │    │(Throttle│    │(Brake)  │         │
    │    └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘         │
    │         │              │              │              │               │
   ─┼─────────┴──────────────┴──────────────┴──────────────┴───────────────┼─ CAN_L
    ║                                                                      ║
   GND                                                                    GND

**Bus Configuration**:

+-------------------+------------------+
| Parameter         | Value            |
+===================+==================+
| Bit Rate          | 250 kbps         |
+-------------------+------------------+
| Termination       | 120Ω at each end |
+-------------------+------------------+
| Cable Type        | Twisted pair     |
+-------------------+------------------+
| Max Length        | 40 meters        |
+-------------------+------------------+

Teensy 4.1 to Waveshare CAN Wiring
----------------------------------

Each Teensy 4.1 connects to a Waveshare CAN transceiver module:

.. code-block:: text

   Teensy 4.1                  Waveshare CAN              CAN Bus
   ──────────                  ─────────────              ───────
   Pin 22 (CTX1) ──────────► CTX
   Pin 23 (CRX1) ◄────────── CRX
                              CAN_H ──────────────────► CAN_H
                              CAN_L ──────────────────► CAN_L
   5V ──────────────────────► VCC
   GND ─────────────────────► GND

CAN Master Node Wiring
----------------------

The CAN Master node connects to the main computer via USB serial:

.. code-block:: text

   MAIN COMPUTER                 CAN MASTER NODE
   ─────────────                 ───────────────
   USB Port ◄──── USB Cable ───► Teensy 4.1 Micro-USB

                                 Teensy 4.1 Pin 22 ──► Waveshare CTX
                                 Teensy 4.1 Pin 23 ◄── Waveshare CRX
                                                       │
                                                       └──► CAN Bus

Steering Node Wiring
--------------------

.. code-block:: text

   STEERING NODE (Teensy 4.1)
   ──────────────────────────
   Pin 4  (DIR)  ────────────► Stepper Driver DIR
   Pin 5  (STEP) ────────────► Stepper Driver STEP

   Pin 20 ◄───────────────────  Right Limit Switch (INPUT_PULLDOWN)
   Pin 21 ◄───────────────────  Left Limit Switch (INPUT_PULLDOWN)

   Pin A0 ◄───────────────────  Analog Position Feedback (Potentiometer)

   Pin 22 (CTX1) ────────────► Waveshare CAN CTX
   Pin 23 (CRX1) ◄──────────── Waveshare CAN CRX

   5V  ──────────────────────► Waveshare VCC
   GND ──────────────────────► Waveshare GND, Limit Switch Common

Throttle Node Wiring
--------------------

.. code-block:: text

   THROTTLE NODE (Teensy 4.1)
   ──────────────────────────
   Pin 18 (SDA) ─────────────► MCP4728 SDA
   Pin 19 (SCL) ─────────────► MCP4728 SCL
   3.3V ─────────────────────► MCP4728 VCC
   GND ──────────────────────► MCP4728 GND

   Pin 22 (CTX1) ────────────► Waveshare CAN CTX
   Pin 23 (CRX1) ◄──────────── Waveshare CAN CRX

   5V  ──────────────────────► Waveshare VCC
   GND ──────────────────────► Waveshare GND

   MCP4728 DAC Outputs:
   ────────────────────
   Channel A (VOUT_A) ───────► Vehicle Throttle Input
   Channel B (VOUT_B) ───────► Drive Mode Line 1
   Channel C (VOUT_C) ───────► Drive Mode Line 2
   Channel D (VOUT_D) ───────► Drive Mode Line 3

Brake Node Wiring
-----------------

.. code-block:: text

   BRAKE NODE (Teensy 4.1)
   ───────────────────────
   Pin 12 (RPWM) ────────────► H-Bridge RPWM (Extend)
   Pin 11 (LPWM) ────────────► H-Bridge LPWM (Retract)

   Pin A0 ◄───────────────────  Analog Position Feedback (Potentiometer)

   Pin 22 (CTX1) ────────────► Waveshare CAN CTX
   Pin 23 (CRX1) ◄──────────── Waveshare CAN CRX

   5V  ──────────────────────► Waveshare VCC
   GND ──────────────────────► Waveshare GND

   H-Bridge Power:
   ───────────────
   +12V ─────────────────────► H-Bridge VCC
   GND ──────────────────────► H-Bridge GND

Power Distribution Wiring
=========================

From Auxiliary Battery
----------------------

.. code-block:: text

   ┌──────────────┐
   │  AUX BATTERY │
   │     12V      │
   └───────┬──────┘
           │
           ▼
   ┌──────────────────────────────────────────────────────────────┐
   │                    POWER DISTRIBUTION BOARD                  │
   │                                                              │
   │   ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐     │
   │   │ 15A │  │ 2A  │  │ 2A  │  │ 2A  │  │ 2A  │  │ 2A  │     │
   │   └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘     │
   │      │        │        │        │        │        │         │
   └──────┼────────┼────────┼────────┼────────┼────────┼─────────┘
          │        │        │        │        │        │
          ▼        ▼        ▼        ▼        ▼        ▼
       Computer  LIDAR    Teensy   Teensy   Teensy   Teensy
                          Master   Steer   Throttle  Brake
                         (+CAN)   (+CAN)   (+CAN)   (+CAN)

Connector Reference
===================

Standard Connectors Used
------------------------

+----------------------+------------------+------------------+
| Component            | Connector Type   | Notes            |
+======================+==================+==================+
| Xsens                | USB Mini-B       | Or 10-pin        |
+----------------------+------------------+------------------+
| Velodyne             | Weatherpack      | Power + Ethernet |
+----------------------+------------------+------------------+
| RealSense            | USB-C            | USB 3.0 required |
+----------------------+------------------+------------------+
| Teensy (×4)          | Micro-USB        | Power via USB    |
+----------------------+------------------+------------------+
| CAN Bus              | Screw terminals  | Or Deutsch DT04  |
+----------------------+------------------+------------------+
| Power                | Anderson PP      | Or ring terminals|
+----------------------+------------------+------------------+

Wire Color Code
---------------

+--------+------------------+
| Color  | Signal           |
+========+==================+
| Red    | +12V / +5V       |
+--------+------------------+
| Black  | Ground           |
+--------+------------------+
| Yellow | CAN_H            |
+--------+------------------+
| Green  | CAN_L            |
+--------+------------------+
| Blue   | Signal / Data    |
+--------+------------------+
| White  | Signal / Data    |
+--------+------------------+
| Orange | Ignition / Enable|
+--------+------------------+

Best Practices
==============

CAN Bus Wiring
--------------

1. **Twisted Pair**: Use twisted pair cable for CAN_H and CAN_L
2. **Termination**: Install 120Ω resistors at both ends of the bus
3. **Daisy Chain**: Connect nodes in a linear bus topology, not star
4. **Keep Short**: Minimize stub lengths from main bus to nodes
5. **Shield**: Use shielded cable in electrically noisy environments

General Wiring
--------------

1. **Strain Relief**: Secure all cables to prevent movement
2. **Shielding**: Ground shields at one end only
3. **Separation**: Keep power and signal cables apart
4. **Labeling**: Label all connectors and cables
5. **Fusing**: Fuse all power circuits appropriately
6. **Testing**: Verify connections before powering on
