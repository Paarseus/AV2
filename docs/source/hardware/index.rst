========
Hardware
========

AV2 hardware documentation.

.. toctree::
   :maxdepth: 1

   sensors
   computing
   actuators
   wiring
   power-system

Components
==========

**Sensors**

- Xsens MTi-630 (GPS/IMU)
- Velodyne VLP-16 (LIDAR)
- Intel RealSense D435 (RGB-D camera)

**Computing**

- Main computer (Ubuntu, perception/planning)
- 4× Teensy 4.1 + Waveshare CAN (distributed actuator control)

**Actuators**

- Steering: Stepper motor with analog feedback
- Throttle: MCP4728 DAC
- Brake: Linear actuator with H-bridge

**Power**

- 12V auxiliary battery for computing/sensors
- Power distribution board with fuses
