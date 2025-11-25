=========
Actuators
=========

All actuators use Teensy 4.1 + Waveshare CAN transceiver nodes on 250 kbps CAN bus.

Steering (0x200)
================

Stepper motor with closed-loop analog position feedback.

+-------------------+---------------------------+
| Control           | Closed-loop position      |
+-------------------+---------------------------+
| Feedback          | Analog potentiometer      |
+-------------------+---------------------------+
| ADC Range         | 350 (left) - 820 (right)  |
+-------------------+---------------------------+
| Deadband          | ±5 ADC counts             |
+-------------------+---------------------------+
| Safety            | Left/right limit switches |
+-------------------+---------------------------+

**Pins**: DIR=4, STEP=5, Right limit=20, Left limit=21, Feedback=A0

**CAN Command** (2 bytes):

+------+---------------------------+
| Byte | Value                     |
+======+===========================+
| 0    | Center flag (0/1)         |
+------+---------------------------+
| 1    | Setpoint (-127 to +127)   |
+------+---------------------------+

Throttle (0x100)
================

MCP4728 quad-channel 12-bit DAC controls throttle voltage and drive mode lines.

+-------------------+---------------------------+
| DAC               | MCP4728 (I2C 0x60/0x64)   |
+-------------------+---------------------------+
| Output Range      | 0 - 4.096V                |
+-------------------+---------------------------+
| Idle Voltage      | 0.30V                     |
+-------------------+---------------------------+
| Max Voltage       | 4.00V                     |
+-------------------+---------------------------+
| Watchdog          | 200ms → Neutral + Idle    |
+-------------------+---------------------------+

**DAC Channels**: A=Throttle, B/C/D=Drive mode lines

**Drive Mode Voltages**:

+---------+--------+--------+--------+
| Mode    | Ch B   | Ch C   | Ch D   |
+=========+========+========+========+
| Neutral | 3.15V  | 1.91V  | 3.20V  |
+---------+--------+--------+--------+
| Drive   | 0.00V  | 1.91V  | 3.18V  |
+---------+--------+--------+--------+
| Sport   | 0.55V  | 1.88V  | 0.00V  |
+---------+--------+--------+--------+
| Reverse | 3.15V  | 0.00V  | 3.18V  |
+---------+--------+--------+--------+

**CAN Command** (3 bytes):

+------+---------------------------+
| Byte | Value                     |
+======+===========================+
| 0    | E-STOP flag (0/1)         |
+------+---------------------------+
| 1    | Throttle (0-255)          |
+------+---------------------------+
| 2    | Mode char ('N','D','S','R')|
+------+---------------------------+

Brake (0x300)
=============

Linear actuator with H-bridge PWM control.

+-------------------+---------------------------+
| Actuator          | 2" stroke linear actuator |
+-------------------+---------------------------+
| Control           | H-bridge (RPWM/LPWM)      |
+-------------------+---------------------------+
| Feedback          | Analog potentiometer      |
+-------------------+---------------------------+
| ADC Range         | 735 (released) - 805 (full)|
+-------------------+---------------------------+
| Watchdog          | 100ms → Full brake        |
+-------------------+---------------------------+

**Pins**: RPWM=12 (extend), LPWM=11 (retract), Feedback=A0

**CAN Command** (2 bytes):

+------+---------------------------+
| Byte | Value                     |
+======+===========================+
| 0    | E-STOP flag (0/1)         |
+------+---------------------------+
| 1    | Brake position (0-255)    |
+------+---------------------------+

E-STOP Behavior
===============

When E-STOP activates:

- Throttle: 0V output, Neutral mode
- Brake: Full extension (100%)
- Steering: Holds position
