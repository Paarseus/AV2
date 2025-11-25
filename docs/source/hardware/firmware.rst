========
Firmware
========

Teensy 4.1 firmware for the distributed CAN control system.

Source Files
============

+----------------------------------+-------------+---------------------------+
| File                             | CAN ID      | Function                  |
+==================================+=============+===========================+
| ``Master/Master_CAN_Final.cpp``  | --          | Serial-to-CAN bridge      |
+----------------------------------+-------------+---------------------------+
| ``Steer/Steer_CAN_Final.cpp``    | 0x200/0x201 | Stepper motor control     |
+----------------------------------+-------------+---------------------------+
| ``Throttle/Throttle_CAN_Final.cpp``| 0x100/0x101| MCP4728 DAC control     |
+----------------------------------+-------------+---------------------------+
| ``Brake/Brake_CAN_Final.cpp``    | 0x300/0x301 | Linear actuator control   |
+----------------------------------+-------------+---------------------------+

Dependencies
============

.. code-block:: ini

   ; platformio.ini
   [env:teensy41]
   platform = teensy
   board = teensy41
   framework = arduino
   lib_deps =
       FlexCAN_T4
       Adafruit MCP4728

Master Node
===========

Receives serial commands from main computer, broadcasts CAN messages to actuators.

**Serial Input** (115200 baud):

.. code-block:: text

   T <0.0-1.0>     # Throttle
   S <-1.0-1.0>    # Steering
   B <0.0-1.0>     # Brake
   M <N/D/S/R>     # Drive mode
   E <0/1>         # E-Stop

**Key Functions**:

- ``sendThrottle(estop, value, mode)`` → ID 0x100
- ``sendSteer(center, value)`` → ID 0x200
- ``sendBrake(estop, value)`` → ID 0x300

Steering Node
=============

Closed-loop stepper motor position control.

**Calibration** (line 18-20):

.. code-block:: cpp

   const int leftAnalogReading  = 350;  // ADC at full LEFT
   const int rightAnalogReading = 820;  // ADC at full RIGHT
   const int BAND               = 5;    // deadband

**Control Loop**: Reads analog feedback, steps motor toward setpoint.

Throttle Node
=============

MCP4728 DAC control for throttle voltage and drive mode lines.

**Calibration** (line 34-36):

.. code-block:: cpp

   static constexpr float THROTTLE_IDLE_V = 0.30f;
   static constexpr float THROTTLE_MAX_V  = 4.00f;

**Watchdog**: 200ms timeout → Neutral + Idle

Brake Node
==========

H-bridge PWM control for linear actuator.

**Calibration** (line 29-30):

.. code-block:: cpp

   const int maxAnalogReading = 805;  // ADC at FULL EXTEND
   const int minAnalogReading = 735;  // ADC at FULL RETRACT

**Watchdog**: 100ms timeout → Full brake (E-STOP behavior)

Flashing
========

.. code-block:: bash

   # PlatformIO
   pio run -t upload

   # Arduino IDE
   # Select Board: Teensy 4.1, then Upload

   # Teensy Loader CLI
   teensy_loader_cli --mcu=TEENSY41 -w firmware.hex
