======
Wiring
======

Sensor Connections
==================

+-------------------+---------------------------+
| Xsens MTi-630     | USB to computer           |
+-------------------+---------------------------+
| Velodyne VLP-16   | Ethernet + 12V power      |
+-------------------+---------------------------+
| RealSense D435    | USB 3.0 to computer       |
+-------------------+---------------------------+

CAN Bus Wiring
==============

.. code-block:: text

   120Ω                                              120Ω
    │                                                  │
   ─┼──────────┬──────────┬──────────┬─────────────────┼─ CAN_H
    │     ┌────┴────┐┌────┴────┐┌────┴────┐┌─────┴────┐│
    │     │ Master  ││ Steer   ││Throttle ││  Brake   ││
    │     │Waveshare││Waveshare││Waveshare││Waveshare ││
    │     └────┬────┘└────┬────┘└────┬────┘└────┬─────┘│
   ─┼──────────┴──────────┴──────────┴──────────┴──────┼─ CAN_L
    │                                                  │
   GND                                                GND

**Bus**: 250 kbps, 120Ω termination at each end

Teensy to Waveshare
-------------------

.. code-block:: text

   Teensy 4.1          Waveshare CAN
   Pin 22 (CTX1) ────► CTX
   Pin 23 (CRX1) ◄──── CRX
   5V ────────────────► VCC
   GND ───────────────► GND

Node Pin Assignments
====================

**Steering Node**:

- DIR=4, STEP=5
- Right limit=20, Left limit=21
- Feedback=A0
- CAN: TX=22, RX=23

**Throttle Node**:

- I2C: SDA=18, SCL=19 (to MCP4728)
- CAN: TX=22, RX=23

**Brake Node**:

- RPWM=12 (extend), LPWM=11 (retract)
- Feedback=A0
- CAN: TX=22, RX=23

Wire Colors
===========

+--------+------------------+
| Red    | +12V / +5V       |
+--------+------------------+
| Black  | Ground           |
+--------+------------------+
| Yellow | CAN_H            |
+--------+------------------+
| Green  | CAN_L            |
+--------+------------------+
