---
layout: default
title: Communication
parent: grblHAL Guide
nav_order: 2
has_children: false
---

# Communication

## Connecting to the Controller
grblHAL communicates through the serial interface on the controller. Connect your controller to your computer with a USB cable. Use any standard serial terminal program to connect to grblHAL or use one of the grblHAL-compatible sender GUIs.

Set the baud rate to 115200 as 8-N-1 (8-bits, no parity, and 1-stop bit). Once connected you should get the Grbl-prompt, which looks like this:

`grblHAL ['$' for help]`

Type $ and press enter to have Grbl print a help message. You should not see any local echo of the $ and enter. Grbl should respond with:

`[HLP:$$ $# $G $I $N $x=val $Nx=line $J=line $SLP $C $X $H ~ ! ? ctrl-x]`

The `$`-commands are grblHAL system commands used to tweak the settings, view or change the states and running modes, and start a homing cycle. The last four non-`$` commands are realtime control commands that can be sent at anytime, no matter what grblHAL is doing. These either immediately change the running behavior or immediately print a report of the important realtime data like current position.

In general, grblHAL assumes all characters and streaming data sent to it is g-code and will parse and try to execute it as soon as it can. However, grblHAL also has two separate system command types that are outside of the normal g-code streaming. One system command type is streamed like g-code, but starts with a $ character to tell grblHAL it's not g-code. The other is composed of a special set of characters that will immediately command grblHAL to do a task in real-time. It's not part of the g-code stream. grblHAL's system commands do things like control machine state, report saved parameters or what it is doing, save or print machine settings, run a homing cycle, or make the machine move faster or slower than programmed.

Next: Commands

---

