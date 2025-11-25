=========
Actuators
=========

Vehicle actuation systems for the AV2 platform.

.. contents:: Contents
   :local:
   :depth: 2

Overview
========

The AV2 platform uses drive-by-wire systems controlled by dedicated Teensy 4.1 nodes communicating over a 250 kbps CAN bus:

- **Steering**: Stepper motor with analog position feedback and limit switches
- **Throttle**: MCP4728 quad-channel 12-bit DAC for throttle and drive mode control
- **Brake**: Linear actuator with analog position feedback

Each actuator subsystem operates autonomously while responding to commands from the CAN Master node, enabling modular development and fault isolation.

Steering System
===============

Description
-----------

The steering system uses a stepper motor to control the steering column, with closed-loop position control using an analog potentiometer for feedback. Hardware limit switches prevent over-travel.

Hardware Components
-------------------

- **Motor**: Stepper motor with driver
- **Feedback**: Analog potentiometer (10-bit ADC)
- **Safety**: Left and right limit switches
- **Controller**: Teensy 4.1 + Waveshare CAN transceiver

Specifications
--------------

+-------------------------+--------------------------------+
| Parameter               | Value                          |
+=========================+================================+
| Control Type            | Closed-loop position           |
+-------------------------+--------------------------------+
| Position Feedback       | Analog potentiometer           |
+-------------------------+--------------------------------+
| ADC Resolution          | 10-bit (0-1023)                |
+-------------------------+--------------------------------+
| Deadband                | ±5 ADC counts                  |
+-------------------------+--------------------------------+
| CAN Command ID          | 0x200                          |
+-------------------------+--------------------------------+
| CAN Status ID           | 0x201                          |
+-------------------------+--------------------------------+

Pin Configuration
-----------------

.. code-block:: text

   Teensy 4.1 Steering Node:
   -------------------------
   Pin 4  → DIR (stepper direction)
   Pin 5  → STEP (stepper pulse)
   Pin 20 → Right limit switch (INPUT_PULLDOWN)
   Pin 21 → Left limit switch (INPUT_PULLDOWN)
   Pin A0 → Analog position feedback
   Pin 22 → CAN TX (to Waveshare)
   Pin 23 → CAN RX (from Waveshare)

Calibration
-----------

The steering system requires calibration of the analog feedback range:

.. code-block:: cpp

   // Calibrate these values for your setup
   const int leftAnalogReading  = 350;  // ADC at full LEFT
   const int rightAnalogReading = 820;  // ADC at full RIGHT
   const int BAND               = 5;    // Deadband (ADC units)

**Calibration Procedure**:

1. Manually move steering to full left, record ADC value
2. Manually move steering to full right, record ADC value
3. Update ``leftAnalogReading`` and ``rightAnalogReading`` in firmware
4. Test center position (should be approximately midpoint)

CAN Protocol
------------

**Command Frame** (ID: 0x200, 2 bytes):

+--------+----------------+---------------------------+
| Byte   | Value          | Description               |
+========+================+===========================+
| 0      | 0 or 1         | Center flag (1 = recenter)|
+--------+----------------+---------------------------+
| 1      | -127 to +127   | Setpoint (-1.0 to +1.0)   |
+--------+----------------+---------------------------+

**Status Frame** (ID: 0x201, 8 bytes):

+--------+---------------------------+
| Byte   | Description               |
+========+===========================+
| 0-1    | Raw ADC reading (16-bit)  |
+--------+---------------------------+
| 2-4    | Reserved                  |
+--------+---------------------------+
| 5      | Limit flags               |
+--------+---------------------------+
| 6      | Step timing (debug)       |
+--------+---------------------------+
| 7      | Current setpoint (int8)   |
+--------+---------------------------+

Throttle System
===============

Description
-----------

The throttle system uses an MCP4728 quad-channel 12-bit DAC to control the electronic throttle and drive mode selection. Channel A controls throttle position, while channels B, C, and D control drive mode selection lines.

Hardware Components
-------------------

- **DAC**: MCP4728 quad-channel 12-bit DAC
- **Interface**: I2C (address 0x60 or 0x64)
- **Output Range**: 0-4.096V (internal 2.048V reference × 2 gain)
- **Controller**: Teensy 4.1 + Waveshare CAN transceiver

Specifications
--------------

+-------------------------+--------------------------------+
| Parameter               | Value                          |
+=========================+================================+
| DAC Resolution          | 12-bit (4096 steps)            |
+-------------------------+--------------------------------+
| Output Voltage Range    | 0 - 4.096V                     |
+-------------------------+--------------------------------+
| Throttle Idle Voltage   | 0.30V                          |
+-------------------------+--------------------------------+
| Throttle Max Voltage    | 4.00V                          |
+-------------------------+--------------------------------+
| I2C Address             | 0x60 or 0x64                   |
+-------------------------+--------------------------------+
| CAN Command ID          | 0x100                          |
+-------------------------+--------------------------------+
| CAN Status ID           | 0x101                          |
+-------------------------+--------------------------------+

DAC Channel Assignment
----------------------

+----------+------------------+---------------------------+
| Channel  | Function         | Notes                     |
+==========+==================+===========================+
| A        | Throttle         | 0.30V (idle) to 4.00V     |
+----------+------------------+---------------------------+
| B        | Mode Line 1      | Drive mode selection      |
+----------+------------------+---------------------------+
| C        | Mode Line 2      | Drive mode selection      |
+----------+------------------+---------------------------+
| D        | Mode Line 3      | Drive mode selection      |
+----------+------------------+---------------------------+

Drive Mode Voltage Configuration
--------------------------------

+---------+------------+------------+------------+
| Mode    | Channel B  | Channel C  | Channel D  |
+=========+============+============+============+
| Neutral | 3.15V      | 1.91V      | 3.20V      |
+---------+------------+------------+------------+
| Drive   | 0.00V      | 1.91V      | 3.18V      |
+---------+------------+------------+------------+
| Sport   | 0.55V      | 1.88V      | 0.00V      |
+---------+------------+------------+------------+
| Reverse | 3.15V      | 0.00V      | 3.18V      |
+---------+------------+------------+------------+

CAN Protocol
------------

**Command Frame** (ID: 0x100, 3 bytes):

+--------+----------------+---------------------------+
| Byte   | Value          | Description               |
+========+================+===========================+
| 0      | 0 or 1         | E-STOP flag               |
+--------+----------------+---------------------------+
| 1      | 0-255          | Throttle (0.0-1.0)        |
+--------+----------------+---------------------------+
| 2      | 'N','D','S','R'| Drive mode character      |
+--------+----------------+---------------------------+

**Status Frame** (ID: 0x101, 8 bytes):

+--------+---------------------------+
| Byte   | Description               |
+========+===========================+
| 0-1    | DAC counts (Channel A)    |
+--------+---------------------------+
| 2      | Current mode (0-3)        |
+--------+---------------------------+
| 3      | Reserved                  |
+--------+---------------------------+
| 4-5    | Throttle voltage × 100    |
+--------+---------------------------+
| 6-7    | Reserved                  |
+--------+---------------------------+

Safety Features
---------------

- **Watchdog Timer**: 200ms timeout reverts to Neutral + Idle
- **Mode Sequencing**: Safe transition through Neutral when changing modes
- **E-STOP**: Immediately sets 0V throttle and Neutral mode

Brake System
============

Description
-----------

The brake system uses a linear actuator with PWM H-bridge control and analog position feedback. The actuator extends to apply braking force.

Hardware Components
-------------------

- **Actuator**: Linear actuator (2" stroke)
- **Driver**: H-bridge (RPWM/LPWM control)
- **Feedback**: Analog potentiometer
- **Controller**: Teensy 4.1 + Waveshare CAN transceiver

Specifications
--------------

+-------------------------+--------------------------------+
| Parameter               | Value                          |
+=========================+================================+
| Actuator Stroke         | 2.0 inches                     |
+-------------------------+--------------------------------+
| PWM Speed               | 255 (full duty)                |
+-------------------------+--------------------------------+
| Position Deadband       | ±5 ADC counts                  |
+-------------------------+--------------------------------+
| Watchdog Timeout        | 100ms                          |
+-------------------------+--------------------------------+
| CAN Command ID          | 0x300                          |
+-------------------------+--------------------------------+
| CAN Status ID           | 0x301                          |
+-------------------------+--------------------------------+

Pin Configuration
-----------------

.. code-block:: text

   Teensy 4.1 Brake Node:
   ----------------------
   Pin 12 → RPWM (extend)
   Pin 11 → LPWM (retract)
   Pin A0 → Analog position feedback
   Pin 22 → CAN TX
   Pin 23 → CAN RX

Calibration
-----------

.. code-block:: cpp

   // Calibrate these values for your actuator
   const int maxAnalogReading = 805;  // ADC at FULL EXTEND (brake applied)
   const int minAnalogReading = 735;  // ADC at FULL RETRACT (brake released)
   const int BAND             = 5;    // Deadband

CAN Protocol
------------

**Command Frame** (ID: 0x300, 2 bytes):

+--------+----------------+---------------------------+
| Byte   | Value          | Description               |
+========+================+===========================+
| 0      | 0 or 1         | E-STOP flag               |
+--------+----------------+---------------------------+
| 1      | 0-255          | Brake position (0.0-1.0)  |
+--------+----------------+---------------------------+

**Status Frame** (ID: 0x301, 8 bytes):

+--------+---------------------------+
| Byte   | Description               |
+========+===========================+
| 0-1    | Raw ADC reading (16-bit)  |
+--------+---------------------------+
| 2-3    | Position in 0.01" units   |
+--------+---------------------------+
| 4      | Command (7=ext, 8=ret, 9=stop)|
+--------+---------------------------+
| 5      | Limit flags               |
+--------+---------------------------+
| 6      | PWM speed                 |
+--------+---------------------------+
| 7      | E-STOP state              |
+--------+---------------------------+

Safety Features
---------------

- **Watchdog Timer**: 100ms timeout applies full brake
- **E-STOP Response**: Immediately extends to full brake position
- **Fail-Safe**: Defaults to braking on communication loss

Emergency Stop System
=====================

The emergency stop system coordinates all actuator nodes:

Activation Methods
------------------

1. **Physical E-Stop Button**: Hardwired, highest priority
2. **Software Command**: ``E 1`` via serial to Master node
3. **Watchdog Timeout**: Automatic on communication loss

E-STOP Behavior
---------------

When E-STOP is activated:

+-------------+----------------------------------+
| Node        | Response                         |
+=============+==================================+
| Throttle    | 0V output, Neutral mode          |
+-------------+----------------------------------+
| Brake       | Full extension (100% brake)      |
+-------------+----------------------------------+
| Steering    | Centers (optional)               |
+-------------+----------------------------------+

Reset Procedure
---------------

1. Clear the physical E-Stop button
2. Send ``E 0`` command via serial
3. Verify all nodes report ready status
4. Resume normal operation

Safety Considerations
=====================

.. danger::

   **Always ensure**:

   - An operator is ready to take manual control
   - The physical E-Stop is accessible and tested
   - The test area is clear of obstacles and people
   - All bystanders are at a safe distance
   - CAN bus termination resistors are properly installed

**Fail-Safe Behavior Summary**:

+----------------------+---------------------------+
| Failure Mode         | Response                  |
+======================+===========================+
| CAN bus failure      | Watchdog applies brakes   |
+----------------------+---------------------------+
| Master node crash    | Actuator watchdogs engage |
+----------------------+---------------------------+
| Power loss           | Actuators hold position   |
+----------------------+---------------------------+
| E-STOP activation    | Full brake, zero throttle |
+----------------------+---------------------------+
