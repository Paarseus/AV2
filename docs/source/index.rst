.. AV2 documentation master file

=====================================
AV2 - Autonomous Vehicle Platform
=====================================

.. image:: _static/cpp_logo.png
   :alt: Cal Poly Pomona Logo
   :align: center
   :width: 300px

|

Welcome to the documentation for **AV2**, an autonomous vehicle research platform developed at the **Autonomous Vehicle Laboratory** at **California State Polytechnic University, Pomona**.

The platform implements a distributed drive-by-wire control system using multiple Teensy 4.1 microcontrollers communicating over a CAN bus network.

.. toctree::
   :maxdepth: 1
   :caption: Getting Started

   getting-started/quick-start
   getting-started/installation
   getting-started/dependencies
   getting-started/first-run

.. toctree::
   :maxdepth: 1
   :caption: Hardware

   hardware/sensors
   hardware/computing
   hardware/actuators
   hardware/firmware
   hardware/wiring
   hardware/power-system

.. toctree::
   :maxdepth: 1
   :caption: Mechanical

   mechanical/vehicle-specifications
   mechanical/sensor-mounting
   mechanical/coordinate-systems

.. toctree::
   :maxdepth: 1
   :caption: Software

   software/architecture
   software/perception
   software/planning
   software/control
   software/sensors
   software/visualization

.. toctree::
   :maxdepth: 1
   :caption: Configuration

   configuration/dwa-config
   configuration/sensor-calibration
   configuration/vehicle-parameters

.. toctree::
   :maxdepth: 1
   :caption: Theory

   theory/pure-pursuit
   theory/dwa
   theory/occupancy-grids
   theory/coordinate-transforms

.. toctree::
   :maxdepth: 1
   :caption: Tutorials

   tutorials/basic-navigation
   tutorials/obstacle-avoidance
   tutorials/custom-routes
   tutorials/tuning-controllers

.. toctree::
   :maxdepth: 1
   :caption: Reference

   api/index
   troubleshooting/common-issues
   troubleshooting/sensor-debugging
   development/contributing
   development/code-style

.. toctree::
   :maxdepth: 1
   :caption: Appendix

   appendix/glossary
   appendix/references
   appendix/changelog
