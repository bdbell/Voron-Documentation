---
layout: default
title: Communication
parent: grblHAL Guide
nav_order: 3
has_children: false
---

# Communication

## Connection
grblHAL communicates through the serial interface on the controller. You just need to connect your controller to your computer with a USB cable. Use any standard serial terminal program to connect to grblHAL or use one of the sender GUIs.

## Commands
### Status Reporting
When a `?` character is sent to grblHAL (no additional line feed or carriage return character required), it will immediately respond with something like `<Idle|MPos:0.000,0.000,0.000|FS:0.0,0>` to report its state and current position. The `?` is always picked-off and removed from the serial receive buffer whenever grblHAL detects one. So, these can be sent at any time. Also, to make it a little easier for GUIs to pick up on status reports, they are always encased by `<>` chevrons.

grblHAL's status report is fairly simply in organization. It always starts with a word describing the machine state like IDLE (descriptions of these are listed elsewhere). The following data values are usually in the order listed below and separated by `|` pipe characters, but may not be in the exact order or printed at all. For a complete description of status report formatting, read the sections below.

### Real-Time Control Commands
The real-time control commands, `~` cycle start/resume, `!` feed hold, `^X` soft-reset, and all of the override commands, all immediately signal grblHAL to change its running state. Just like `?` status reports, these control characters are picked-off and removed from the serial buffer when they are detected and do not require an additional line-feed or carriage-return character to operate.



---

