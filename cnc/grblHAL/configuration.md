---
layout: default
title: Overview
parent: grblHAL Guide
nav_order: 1
has_children: false
---

# Overview
grblHAL is a no-compromise, high performance, low cost alternative to parallel-port-based motion control for CNC milling, lathe, and laser operations.  It is mainly aimed at ARM processors (or other 32-bit MCUs) with ample amounts of RAM and flash (compared to AVR 328p) and requires a hardware driver to be functional.  It runs as a native firmware on control boards in a similar way Marlin for 3d printers.  The list of know supported controller boards is listed on the [grblHAL Github page](https://github.com/grblHAL/Controllers).

grblHAL accepts standards-compliant g-code and has been tested with the output of several CAM tools with no problems. Arcs, circles and helical motion are fully supported, as well as, all other primary g-code commands. Macro functions, variables, and some canned cycles are not supported.

grblHAL requires an outside source for management and feeding gcode.  This is commonly referred to as a "sender" and communicates with grblHAL over a serial port (or serial over USB). See the communication page for more details.

---

