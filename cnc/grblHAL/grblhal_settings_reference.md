---
layout: default
title: grblHAL Configuration Settings Reference Guide
parent: grblHAL Guide
nav_order: 8
has_children: false
---

# grblHAL Configuration Settings Reference Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Core Motion Settings ($0-$199)](#core-motion-settings-0-199)
3. [Extended grblHAL Settings ($200-$399)](#extended-grblhal-settings-200-399)
4. [Plugin Settings ($400+)](#plugin-settings-400)
5. [Networking Plugin Settings](#networking-plugin-settings)
6. [Spindle Plugin Settings](#spindle-plugin-settings)
7. [Trinamic Plugin Settings](#trinamic-plugin-settings)
8. [SD Card Plugin Settings](#sd-card-plugin-settings)
9. [Additional Plugin Settings](#additional-plugin-settings)

---

## Introduction

This reference guide provides comprehensive documentation of all grblHAL configuration settings, including core functionality and plugin extensions. Settings are accessed via the `$$` command and modified using `$<number>=<value>` format.

### Key Features of grblHAL Settings
- **Backward Compatibility**: Maintains compatibility with grbl 1.1 settings
- **Dynamic Extensions**: Plugin-specific settings allocated automatically
- **Real-time Updates**: Most settings take effect immediately
- **Validation**: Built-in range checking and validation
- **Persistence**: Settings stored in non-volatile memory

### Important Notes
- **Settings Version**: Current version is 22 (as of build 20250604)
- **Reset Behavior**: Settings reset may occur on firmware updates
- **Backup/Restore**: Use `$EXPORT` and `$IMPORT` commands for backup
- **Range Validation**: Invalid values are rejected with error messages

---

## Core Motion Settings ($0-$199)

### Step Pulse and Timing

#### $0 - Step Pulse Time (microseconds)
- **Range**: 2-1000
- **Default**: 10
- **Description**: Duration of step pulse signals
- **Usage**: High step rates (>80kHz) require values <10μs
- **Example**: `$0=5` for 200kHz step rates

#### $1 - Step Idle Delay (milliseconds)
- **Range**: 0-65535, 255=infinite
- **Default**: 25
- **Description**: Delay before disabling steppers after motion stops
- **Usage**: Set to 255 to keep steppers always enabled
- **Power Saving**: Lower values save power but may cause position loss

#### $29 - Step Pulse Delay (grblHAL Extension)
- **Range**: 0-1000 (microseconds)
- **Default**: 0
- **Description**: Delay between direction and step signals
- **Usage**: Required for some stepper drivers that need setup time

### Direction and Inversion Control

#### $2 - Step Port Invert Mask
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts step pulse polarity per axis
- **Calculation**: X=1, Y=2, Z=4, A=8, B=16, C=32
- **Example**: `$2=5` inverts X and Z axes (1+4=5)

#### $3 - Direction Port Invert Mask
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts direction signal polarity per axis
- **Usage**: Change axis direction without rewiring
- **Example**: `$3=2` inverts Y-axis direction only

#### $4 - Step Enable Invert
- **Range**: 0 (normal low) or 1 (inverted high)
- **Default**: 0
- **Description**: Inverts stepper enable pin logic
- **Usage**: Some drivers require high signal to enable

#### $8 - Ganged Direction Invert Mask (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts direction for ganged/dual motor axes
- **Usage**: Compensates for reversed motor mounting

### Input Signal Configuration

#### $5 - Limit Pins Invert Mask
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts limit switch input logic
- **Note**: grblHAL defaults to normally closed (NC) switches
- **Quick Setup**: Use with pull-up disable settings for NO switches

#### $6 - Probe Pin Invert
- **Range**: 0 (NC) or 1 (NO)
- **Default**: 0
- **Description**: Inverts probe input logic
- **Usage**: Set to 1 for normally open probe switches

#### $14 - Control Signals Invert Mask
- **Range**: 0-1023 (bitmask)
- **Default**: 0
- **Description**: Inverts control input signals
- **Signals**: Reset=1, Feed Hold=2, Cycle Start=4, Safety Door=8, Block Delete=16, Optional Stop=32, EStop=64, Motor Warning=128, Motor Fault=256, Probe Disconnect=512
- **Quick Fix**: `$14=73` for NO switches when testing without connections

#### $17 - Control Pull-up Disable Mask (grblHAL Extension)
- **Range**: 0-1023 (bitmask)
- **Default**: 0
- **Description**: Disables internal pull-up resistors for control inputs
- **Warning**: Requires external pull-down resistors when disabled

#### $18 - Limit Pull-up Disable Mask (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Disables internal pull-up resistors for limit switches
- **Warning**: Requires external pull-down resistors when disabled

#### $19 - Probe Pull-up Disable (grblHAL Extension)
- **Range**: 0 (enabled) or 1 (disabled)
- **Default**: 0
- **Description**: Disables internal pull-up resistor for probe input

### Spindle and PWM Control

#### $7 - Spindle PWM Options (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Various spindle PWM behavior options
- **Flags**: Implementation-dependent, consult driver documentation

#### $9 - Spindle PWM Extended Options (grblHAL Extension)
- **Range**: 0-255 (bitmask)  
- **Default**: 0
- **Description**: Additional spindle PWM configuration options

#### $30 - Spindle PWM Maximum
- **Range**: 0-1000
- **Default**: 1000
- **Description**: Maximum spindle speed for PWM output
- **Usage**: Corresponds to S1000 command

#### $31 - Spindle PWM Minimum  
- **Range**: 0-1000
- **Default**: 0
- **Description**: Minimum spindle speed for PWM output
- **Usage**: Prevents stalling at low speeds

#### $32 - Laser Mode Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables laser-specific behaviors
- **Features**: Dynamic power scaling, M4 support

#### $33 - Spindle PWM Frequency (grblHAL Extension)
- **Range**: Driver-dependent (typically 100-40000 Hz)
- **Default**: Driver-dependent
- **Description**: PWM frequency for spindle control
- **Usage**: Higher frequency reduces motor noise

#### $34 - Spindle PWM Off Value (grblHAL Extension)
- **Range**: 0-100 (percentage)
- **Default**: 0
- **Description**: PWM duty cycle when spindle is off

#### $35 - Spindle PWM Min Value (grblHAL Extension)
- **Range**: 0-100 (percentage)
- **Default**: 0
- **Description**: Minimum PWM duty cycle percentage

#### $36 - Spindle PWM Max Value (grblHAL Extension)
- **Range**: 0-100 (percentage) 
- **Default**: 100
- **Description**: Maximum PWM duty cycle percentage

#### $15 - Coolant Invert Mask
- **Range**: 0-7 (bitmask)
- **Default**: 0
- **Description**: Inverts coolant control outputs
- **Signals**: Flood=1, Mist=2, Additional outputs driver-dependent

#### $16 - Spindle Invert Mask
- **Range**: 0-7 (bitmask)
- **Default**: 0
- **Description**: Inverts spindle control outputs
- **Signals**: Enable=1, Direction=2, Additional signals driver-dependent

### Motion Planning and Control

#### $10 - Status Report Mask
- **Range**: 0-2047 (bitmask)
- **Default**: 1
- **Description**: Configures real-time status report contents
- **Flags**:
  - 0: Work position (WPos)
  - 1: Machine position (MPos) 
  - 2: Planner buffer status
  - 3: RX buffer status
  - 4: Limit pin status
  - 5: Current feed and speed
  - 6: Pin states on mode change
  - 7: Override values
  - 8: Probe coordinates
  - 9: Buffer sync on WPos change
  - 10: Parser state on WPos change
  - 11: Substates in Run mode (grblHAL)

#### $11 - Junction Deviation
- **Range**: 0.001-2.0 (mm)
- **Default**: 0.01
- **Description**: Cornering junction deviation for path planning
- **Effect**: Higher values = faster cornering, lower precision
- **Tuning**: Start with 0.01mm, increase gradually if needed

#### $12 - Arc Tolerance
- **Range**: 0.002-1.0 (mm)
- **Default**: 0.002  
- **Description**: Arc segmentation tolerance
- **Effect**: Smaller values = smoother arcs, more processing
- **Recommendation**: Keep at default unless arc quality issues

#### $13 - Report Inches
- **Range**: 0 (mm) or 1 (inches)
- **Default**: 0
- **Description**: Units for status reports
- **Note**: Independent of G-code units (G20/G21)

### Limits and Homing

#### $20 - Soft Limits Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables software-based travel limits
- **Requirement**: Homing must be completed first
- **Safety**: Prevents crashes when properly configured

#### $21 - Hard Limits Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables limit switch monitoring during operation
- **Effect**: Triggers alarm on limit switch activation

#### $22 - Homing Cycle Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 1
- **Description**: Enables homing cycle functionality
- **Requirement**: Must have limit switches properly configured

#### $23 - Homing Direction Invert
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Sets homing direction for each axis
- **Usage**: 1=positive direction, 0=negative direction per axis

#### $24 - Homing Feed Rate
- **Range**: 1-65535 (mm/min)
- **Default**: 25
- **Description**: Slow precision approach rate during homing
- **Usage**: Final approach speed to limit switches

#### $25 - Homing Seek Rate
- **Range**: 1-65535 (mm/min)
- **Default**: 500
- **Description**: Fast search rate during homing
- **Usage**: Initial search speed to find limit switches

#### $26 - Homing Debounce Delay
- **Range**: 0-65535 (milliseconds)
- **Default**: 250
- **Description**: Delay for limit switch debouncing
- **Usage**: Increase if false triggers occur

#### $27 - Homing Pull-off Distance
- **Range**: 0-1000 (mm)
- **Default**: 1.0
- **Description**: Distance to back off from limit switches after homing
- **Safety**: Ensures switches are not left activated

### Per-Axis Settings ($100-$199)

Each axis has four dedicated settings with 10-number increments:

#### Travel Resolution (Steps per mm)
- **$100**: X-axis steps per mm
- **$101**: Y-axis steps per mm  
- **$102**: Z-axis steps per mm
- **$103**: A-axis steps per mm (if configured)
- **$104**: B-axis steps per mm (if configured)
- **$105**: C-axis steps per mm (if configured)

**Range**: 0.1-10000.0
**Calculation**: (Motor steps/rev × Microstepping) ÷ Travel per revolution
**Examples**:
- Belt drive: (200 × 16) ÷ (2mm × 20 teeth) = 80 steps/mm
- Leadscrew: (200 × 16) ÷ 8mm = 400 steps/mm

#### Maximum Rate (mm/min)
- **$110**: X-axis maximum rate
- **$111**: Y-axis maximum rate
- **$112**: Z-axis maximum rate
- **$113**: A-axis maximum rate
- **$114**: B-axis maximum rate
- **$115**: C-axis maximum rate

**Range**: 1-65535 mm/min
**Usage**: Limits maximum feed rate for rapid moves
**Safety**: Set conservatively initially, increase as verified

#### Acceleration (mm/sec²)
- **$120**: X-axis acceleration
- **$121**: Y-axis acceleration
- **$122**: Z-axis acceleration
- **$123**: A-axis acceleration
- **$124**: B-axis acceleration
- **$125**: C-axis acceleration

**Range**: 1-65535 mm/sec²
**Usage**: Controls acceleration and deceleration rates
**Tuning**: Balance speed vs. mechanical stress

#### Maximum Travel (mm)
- **$130**: X-axis maximum travel
- **$131**: Y-axis maximum travel
- **$132**: Z-axis maximum travel
- **$133**: A-axis maximum travel
- **$134**: B-axis maximum travel
- **$135**: C-axis maximum travel

**Range**: 0-1000000 mm
**Usage**: Defines soft limit boundaries
**Requirement**: Requires homing and soft limits enabled

---

## Extended grblHAL Settings ($200-$399)

### Advanced Motion Control

#### $28 - G73 Retract Distance (grblHAL Extension)
- **Range**: 0-1000 (mm)
- **Default**: 1.0
- **Description**: Retract distance for G73 peck drilling cycle

#### $37 - Steppers Deenergize Mask (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 255 (all axes)
- **Description**: Defines which steppers to deenergize when motion completes
- **Usage**: X=1, Y=2, Z=4, etc. Sum for multiple axes
- **Power Saving**: Reduces motor heating and power consumption

#### $38 - Spindle Encoder PPR (grblHAL Extension)
- **Range**: 1-10000
- **Default**: Driver-dependent
- **Description**: Spindle encoder pulses per revolution
- **Usage**: Required for spindle-synchronized motion (threading)

#### $39 - Enable Legacy RT Commands (grblHAL Extension)
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 1
- **Description**: Enables printable real-time command characters
- **Characters**: `?` (status), `!` (feed hold), `~` (cycle start)
- **Safety**: Disable if characters appear in G-code comments

#### $40 - Limit Jog Commands (grblHAL Extension)
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables soft limits for jogging commands
- **Safety**: Prevents jogging beyond travel limits

### Homing Configuration Extensions

#### $43 - Homing Locate Cycles (grblHAL Extension)
- **Range**: 0-255
- **Default**: 1
- **Description**: Number of homing locate cycles per axis
- **Usage**: Multiple cycles can improve precision

#### $44-$49 - Homing Order (grblHAL Extension)
Priority-based homing order (lowest number executed first):
- **$44**: 1st priority axis mask
- **$45**: 2nd priority axis mask  
- **$46**: 3rd priority axis mask
- **$47**: 4th priority axis mask
- **$48**: 5th priority axis mask
- **$49**: 6th priority axis mask

**Range**: 0-255 (bitmask)
**Usage**: Allows custom homing sequences
**Example**: `$44=4` (Z first), `$45=3` (X,Y together)

### Jogging Defaults

#### $50 - Jogging Step Speed (grblHAL Extension)
- **Range**: 1-65535 (mm/min)
- **Default**: 100
- **Description**: Default speed for step jogging
- **Usage**: Used by jogging interfaces

#### $51 - Jogging Slow Speed (grblHAL Extension)
- **Range**: 1-65535 (mm/min)
- **Default**: 25
- **Description**: Default speed for slow continuous jogging

#### $52 - Jogging Fast Speed (grblHAL Extension)
- **Range**: 1-65535 (mm/min)
- **Default**: 1000
- **Description**: Default speed for fast continuous jogging

#### $53 - Jogging Step Distance (grblHAL Extension)
- **Range**: 0.001-1000 (mm)
- **Default**: 0.1
- **Description**: Default distance for step jogging

#### $54 - Jogging Slow Distance (grblHAL Extension)
- **Range**: 0.001-1000 (mm)
- **Default**: 1.0
- **Description**: Default distance for slow jogging

#### $55 - Jogging Fast Distance (grblHAL Extension)
- **Range**: 0.001-1000 (mm)
- **Default**: 10.0
- **Description**: Default distance for fast jogging

### System Configuration

#### $338 - Planner Buffer Blocks (grblHAL Extension)
- **Range**: 16-1000
- **Default**: 100
- **Description**: Number of linear motions in planner buffer
- **Impact**: More blocks = better lookahead, more RAM usage
- **Tuning**: Increase if complex paths have speed variations

#### $376 - Disable G92 Persistence (grblHAL Extension)
- **Range**: 0 (enabled) or 1 (disabled)
- **Default**: 0 (when COMPATIBILITY_LEVEL ≤ 1)
- **Description**: Controls whether G92 offsets are saved to NVS
- **Compatibility**: Disabled by default for grbl 1.1 compatibility

---

## Plugin Settings ($400+)

Plugin settings are dynamically allocated based on enabled plugins. Setting numbers may vary depending on plugin loading order.

---

## Networking Plugin Settings

The networking plugin provides Ethernet, WiFi, and various protocol support.

### Network Interface Configuration

#### $70 - Network Services Enable
- **Range**: 0-31 (bitmask)
- **Default**: 0
- **Description**: Enables network services
- **Services**:
  - 1: Telnet
  - 2: WebSocket  
  - 4: HTTP
  - 8: FTP
  - 16: mDNS
- **Example**: `$70=15` enables Telnet, WebSocket, HTTP, FTP

#### $71 - Network Mode (WiFi)
- **Range**: 0-2
- **Values**: 0=Off, 1=Station, 2=Access Point
- **Description**: WiFi operating mode

#### $72 - WiFi SSID
- **Range**: String (max 32 characters)
- **Description**: WiFi network name for station mode

#### $73 - WiFi Password  
- **Range**: String (max 63 characters)
- **Description**: WiFi network password

#### $74 - WiFi Country Code
- **Range**: String (2 characters)
- **Default**: "US"
- **Description**: WiFi regulatory domain

### IP Configuration

#### $75 - IP Mode
- **Range**: 0-2
- **Values**: 0=Static, 1=DHCP, 2=AutoIP
- **Default**: 1 (DHCP)
- **Description**: IP address assignment method

#### $76 - IP Address
- **Range**: IPv4 address string
- **Default**: "192.168.1.210"
- **Description**: Static IP address (when IP mode = 0)

#### $77 - Gateway Address
- **Range**: IPv4 address string  
- **Default**: "192.168.1.1"
- **Description**: Network gateway address

#### $78 - Subnet Mask
- **Range**: IPv4 address string
- **Default**: "255.255.255.0"
- **Description**: Network subnet mask

### Access Point Configuration

#### $311 - AP SSID
- **Range**: String (max 32 characters)
- **Default**: "grblHAL_AP"
- **Description**: Access Point network name

#### $312 - AP Password
- **Range**: String (8-63 characters)
- **Default**: "grblhal"
- **Description**: Access Point password

#### $313 - AP IP Address
- **Range**: IPv4 address string
- **Default**: "192.168.4.1"
- **Description**: Access Point IP address

#### $314 - AP Gateway
- **Range**: IPv4 address string
- **Default**: "192.168.4.1"
- **Description**: Access Point gateway address

#### $315 - AP Subnet Mask
- **Range**: IPv4 address string
- **Default**: "255.255.255.0"
- **Description**: Access Point subnet mask

### Protocol Port Configuration

#### $305 - Telnet Port
- **Range**: 1-65535
- **Default**: 23
- **Description**: TCP port for Telnet protocol

#### $306 - HTTP Port
- **Range**: 1-65535  
- **Default**: 80
- **Description**: TCP port for HTTP protocol

#### $307 - WebSocket Port
- **Range**: 1-65535
- **Default**: 81
- **Description**: TCP port for WebSocket protocol

#### $308 - FTP Port
- **Range**: 1-65535
- **Default**: 21
- **Description**: TCP port for FTP protocol

### Network Security and Features

#### $316 - Network Hostname
- **Range**: String (max 32 characters)
- **Default**: "grblHAL"
- **Description**: mDNS hostname for network discovery

#### $317 - WebDAV Authentication
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables WebDAV authentication

---

## Spindle Plugin Settings

The spindle plugin adds support for VFD control via Modbus and multiple spindle management.

### VFD Communication Settings

#### $360 - Modbus Address (Legacy)
- **Range**: 1-247
- **Default**: 1
- **Description**: Modbus slave address for single VFD configuration
- **Note**: Used when only one VFD spindle is configured

#### $395 - Active Spindle Selection
- **Range**: 0-15 (spindle ID)
- **Default**: 0
- **Description**: Selects active spindle when multiple spindles configured
- **Usage**: Changes with `$$=395` to see available spindles

#### $461 - RPM to Hz Relationship  
- **Range**: 1-3600
- **Default**: 60
- **Description**: RPM to frequency conversion factor
- **Calculation**: Motor RPM = Frequency × 60 ÷ Pole pairs
- **Usage**: Depends on motor specifications

### VFD Modbus Configuration

#### $462 - Run/Stop Register
- **Range**: 1-65535
- **Default**: 8192
- **Description**: Modbus register for spindle run/stop control

#### $463 - Set Frequency Register
- **Range**: 1-65535
- **Default**: 8193  
- **Description**: Modbus register for frequency setpoint

#### $464 - Get Frequency Register
- **Range**: 1-65535
- **Default**: 8451
- **Description**: Modbus register for frequency readback

### Multi-Spindle Configuration

#### $476 - VFD Address Spindle 0
- **Range**: 1-247
- **Default**: 1
- **Description**: Modbus address for VFD bound to spindle 0

#### $477 - VFD Address Spindle 1
- **Range**: 1-247
- **Default**: 2
- **Description**: Modbus address for VFD bound to spindle 1

#### $478 - VFD Address Spindle 2
- **Range**: 1-247
- **Default**: 3
- **Description**: Modbus address for VFD bound to spindle 2

#### $479 - VFD Address Spindle 3
- **Range**: 1-247
- **Default**: 4
- **Description**: Modbus address for VFD bound to spindle 3

### Spindle Selection by Tool Number

#### $511 - Spindle 1 Enable
- **Range**: 0-15 (spindle ID)
- **Description**: Binds spindle number 1 to specified spindle ID

#### $512 - Spindle 2 Enable
- **Range**: 0-15 (spindle ID)
- **Description**: Binds spindle number 2 to specified spindle ID

#### $513 - Spindle 3 Enable
- **Range**: 0-15 (spindle ID)
- **Description**: Binds spindle number 3 to specified spindle ID

#### $520 - Tool Number Range Spindle 0
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 0
- **Usage**: Tools ≥ this number select spindle 0

#### $521 - Tool Number Range Spindle 1
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 1

#### $522 - Tool Number Range Spindle 2
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 2

#### $523 - Tool Number Range Spindle 3
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 3

---

## Trinamic Plugin Settings

The Trinamic plugin provides comprehensive support for TMC2130, TMC2209, and TMC5160 stepper drivers.

### Driver Enable and Basic Configuration

Settings are per-axis with standard axis numbering (X=0, Y=1, Z=2, etc.).

#### Driver Enable Settings
- **$480**: X-axis Trinamic driver enable (0=disabled, 1=enabled)
- **$481**: Y-axis Trinamic driver enable
- **$482**: Z-axis Trinamic driver enable
- **$483**: A-axis Trinamic driver enable
- **$484**: B-axis Trinamic driver enable
- **$485**: C-axis Trinamic driver enable

#### Current Settings (mA)
- **$486**: X-axis motor current
- **$487**: Y-axis motor current  
- **$488**: Z-axis motor current
- **$489**: A-axis motor current
- **$490**: B-axis motor current
- **$491**: C-axis motor current

**Range**: 50-3000 mA (driver dependent)
**Note**: Current is not permanently stored, reset on power cycle

#### Microstep Settings
- **$492**: X-axis microsteps
- **$493**: Y-axis microsteps
- **$494**: Z-axis microsteps
- **$495**: A-axis microsteps  
- **$496**: B-axis microsteps
- **$497**: C-axis microsteps

**Valid Values**: 1, 2, 4, 8, 16, 32, 64, 128, 256
**Default**: 16
**Effect**: Higher values = smoother motion, lower torque

### StallGuard Settings (Sensorless Homing)

#### StallGuard Threshold
- **$498**: X-axis StallGuard threshold
- **$499**: Y-axis StallGuard threshold
- **$500**: Z-axis StallGuard threshold
- **$501**: A-axis StallGuard threshold
- **$502**: B-axis StallGuard threshold  
- **$503**: C-axis StallGuard threshold

**Range**: 
- TMC2130/TMC5160: -64 to 63
- TMC2209: 0 to 255
**Usage**: Lower values = more sensitive detection
**Tuning**: Requires careful calibration per axis

#### Homing Feed Sensitivity
- **$504**: X-axis homing sensitivity
- **$505**: Y-axis homing sensitivity
- **$506**: Z-axis homing sensitivity
- **$507**: A-axis homing sensitivity
- **$508**: B-axis homing sensitivity
- **$509**: C-axis homing sensitivity

**Range**: Same as StallGuard threshold
**Description**: StallGuard threshold during homing
**Usage**: Often different from normal operation threshold

### Advanced Trinamic Settings

#### Hybrid Threshold (TMC2130/TMC5160)
- **$510**: X-axis hybrid threshold
- **$511**: Y-axis hybrid threshold
- **$512**: Z-axis hybrid threshold
- **$513**: A-axis hybrid threshold
- **$514**: B-axis hybrid threshold
- **$515**: C-axis hybrid threshold

**Range**: 0-65535 (steps/second)
**Description**: Speed threshold for switching to SpreadCycle mode
**Usage**: 0 = always StealthChop, high value = always SpreadCycle

#### ChopConf Settings (Extended Settings)
When `TRINAMIC_EXTENDED_SETTINGS` is enabled:

**$516**: TOFF - Off time setting
**$517**: TBL - Blanking time  
**$518**: CHM - Chopper mode
**$519**: HSTRT - Hysteresis start
**$520**: HEND - Hysteresis end
**$521**: HDEC - Hysteresis decrement
**$522**: RNDTF - Random TOFF

**Usage**: Advanced motor tuning parameters
**Caution**: Incorrect values can damage drivers or motors

#### CoolConf Settings (Extended Settings)
**$523**: SEMIN - Minimum StallGuard value
**$524**: SEUP - Current increment steps
**$525**: SEMAX - Maximum StallGuard value  
**$526**: SEDN - Current decrement steps
**$527**: SEIMIN - Minimum current flag

**Description**: Automatic current adjustment based on load
**Usage**: Optimizes current consumption and motor heating

---

## SD Card Plugin Settings

The SD card plugin enables G-code file storage and execution from SD cards.

### Basic SD Card Configuration

#### $675 - Macro ATC Options
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Options for macro-based Automatic Tool Change
- **Flags**:
  - 1: Enable execution of M6T0 to unload tool
- **Usage**: Controls ATC macro behavior

### SD Card Commands

The SD card plugin adds several `# grblHAL Configuration Settings Reference Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Core Motion Settings ($0-$199)](#core-motion-settings-0-199)
3. [Extended grblHAL Settings ($200-$399)](#extended-grblhal-settings-200-399)
4. [Plugin Settings ($400+)](#plugin-settings-400)
5. [Networking Plugin Settings](#networking-plugin-settings)
6. [Spindle Plugin Settings](#spindle-plugin-settings)
7. [Trinamic Plugin Settings](#trinamic-plugin-settings)
8. [SD Card Plugin Settings](#sd-card-plugin-settings)
9. [Additional Plugin Settings](#additional-plugin-settings)

---

## Introduction

This reference guide provides comprehensive documentation of all grblHAL configuration settings, including core functionality and plugin extensions. Settings are accessed via the `$$` command and modified using `$<number>=<value>` format.

### Key Features of grblHAL Settings
- **Backward Compatibility**: Maintains compatibility with grbl 1.1 settings
- **Dynamic Extensions**: Plugin-specific settings allocated automatically
- **Real-time Updates**: Most settings take effect immediately
- **Validation**: Built-in range checking and validation
- **Persistence**: Settings stored in non-volatile memory

### Important Notes
- **Settings Version**: Current version is 22 (as of build 20250604)
- **Reset Behavior**: Settings reset may occur on firmware updates
- **Backup/Restore**: Use `$EXPORT` and `$IMPORT` commands for backup
- **Range Validation**: Invalid values are rejected with error messages

---

## Core Motion Settings ($0-$199)

### Step Pulse and Timing

#### $0 - Step Pulse Time (microseconds)
- **Range**: 2-1000
- **Default**: 10
- **Description**: Duration of step pulse signals
- **Usage**: High step rates (>80kHz) require values <10μs
- **Example**: `$0=5` for 200kHz step rates

#### $1 - Step Idle Delay (milliseconds)
- **Range**: 0-65535, 255=infinite
- **Default**: 25
- **Description**: Delay before disabling steppers after motion stops
- **Usage**: Set to 255 to keep steppers always enabled
- **Power Saving**: Lower values save power but may cause position loss

#### $29 - Step Pulse Delay (grblHAL Extension)
- **Range**: 0-1000 (microseconds)
- **Default**: 0
- **Description**: Delay between direction and step signals
- **Usage**: Required for some stepper drivers that need setup time

### Direction and Inversion Control

#### $2 - Step Port Invert Mask
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts step pulse polarity per axis
- **Calculation**: X=1, Y=2, Z=4, A=8, B=16, C=32
- **Example**: `$2=5` inverts X and Z axes (1+4=5)

#### $3 - Direction Port Invert Mask
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts direction signal polarity per axis
- **Usage**: Change axis direction without rewiring
- **Example**: `$3=2` inverts Y-axis direction only

#### $4 - Step Enable Invert
- **Range**: 0 (normal low) or 1 (inverted high)
- **Default**: 0
- **Description**: Inverts stepper enable pin logic
- **Usage**: Some drivers require high signal to enable

#### $8 - Ganged Direction Invert Mask (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts direction for ganged/dual motor axes
- **Usage**: Compensates for reversed motor mounting

### Input Signal Configuration

#### $5 - Limit Pins Invert Mask
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts limit switch input logic
- **Note**: grblHAL defaults to normally closed (NC) switches
- **Quick Setup**: Use with pull-up disable settings for NO switches

#### $6 - Probe Pin Invert
- **Range**: 0 (NC) or 1 (NO)
- **Default**: 0
- **Description**: Inverts probe input logic
- **Usage**: Set to 1 for normally open probe switches

#### $14 - Control Signals Invert Mask
- **Range**: 0-1023 (bitmask)
- **Default**: 0
- **Description**: Inverts control input signals
- **Signals**: Reset=1, Feed Hold=2, Cycle Start=4, Safety Door=8, Block Delete=16, Optional Stop=32, EStop=64, Motor Warning=128, Motor Fault=256, Probe Disconnect=512
- **Quick Fix**: `$14=73` for NO switches when testing without connections

#### $17 - Control Pull-up Disable Mask (grblHAL Extension)
- **Range**: 0-1023 (bitmask)
- **Default**: 0
- **Description**: Disables internal pull-up resistors for control inputs
- **Warning**: Requires external pull-down resistors when disabled

#### $18 - Limit Pull-up Disable Mask (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Disables internal pull-up resistors for limit switches
- **Warning**: Requires external pull-down resistors when disabled

#### $19 - Probe Pull-up Disable (grblHAL Extension)
- **Range**: 0 (enabled) or 1 (disabled)
- **Default**: 0
- **Description**: Disables internal pull-up resistor for probe input

### Spindle and PWM Control

#### $7 - Spindle PWM Options (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Various spindle PWM behavior options
- **Flags**: Implementation-dependent, consult driver documentation

#### $9 - Spindle PWM Extended Options (grblHAL Extension)
- **Range**: 0-255 (bitmask)  
- **Default**: 0
- **Description**: Additional spindle PWM configuration options

#### $30 - Spindle PWM Maximum
- **Range**: 0-1000
- **Default**: 1000
- **Description**: Maximum spindle speed for PWM output
- **Usage**: Corresponds to S1000 command

#### $31 - Spindle PWM Minimum  
- **Range**: 0-1000
- **Default**: 0
- **Description**: Minimum spindle speed for PWM output
- **Usage**: Prevents stalling at low speeds

#### $32 - Laser Mode Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables laser-specific behaviors
- **Features**: Dynamic power scaling, M4 support

#### $33 - Spindle PWM Frequency (grblHAL Extension)
- **Range**: Driver-dependent (typically 100-40000 Hz)
- **Default**: Driver-dependent
- **Description**: PWM frequency for spindle control
- **Usage**: Higher frequency reduces motor noise

#### $34 - Spindle PWM Off Value (grblHAL Extension)
- **Range**: 0-100 (percentage)
- **Default**: 0
- **Description**: PWM duty cycle when spindle is off

#### $35 - Spindle PWM Min Value (grblHAL Extension)
- **Range**: 0-100 (percentage)
- **Default**: 0
- **Description**: Minimum PWM duty cycle percentage

#### $36 - Spindle PWM Max Value (grblHAL Extension)
- **Range**: 0-100 (percentage) 
- **Default**: 100
- **Description**: Maximum PWM duty cycle percentage

#### $15 - Coolant Invert Mask
- **Range**: 0-7 (bitmask)
- **Default**: 0
- **Description**: Inverts coolant control outputs
- **Signals**: Flood=1, Mist=2, Additional outputs driver-dependent

#### $16 - Spindle Invert Mask
- **Range**: 0-7 (bitmask)
- **Default**: 0
- **Description**: Inverts spindle control outputs
- **Signals**: Enable=1, Direction=2, Additional signals driver-dependent

### Motion Planning and Control

#### $10 - Status Report Mask
- **Range**: 0-2047 (bitmask)
- **Default**: 1
- **Description**: Configures real-time status report contents
- **Flags**:
  - 0: Work position (WPos)
  - 1: Machine position (MPos) 
  - 2: Planner buffer status
  - 3: RX buffer status
  - 4: Limit pin status
  - 5: Current feed and speed
  - 6: Pin states on mode change
  - 7: Override values
  - 8: Probe coordinates
  - 9: Buffer sync on WPos change
  - 10: Parser state on WPos change
  - 11: Substates in Run mode (grblHAL)

#### $11 - Junction Deviation
- **Range**: 0.001-2.0 (mm)
- **Default**: 0.01
- **Description**: Cornering junction deviation for path planning
- **Effect**: Higher values = faster cornering, lower precision
- **Tuning**: Start with 0.01mm, increase gradually if needed

#### $12 - Arc Tolerance
- **Range**: 0.002-1.0 (mm)
- **Default**: 0.002  
- **Description**: Arc segmentation tolerance
- **Effect**: Smaller values = smoother arcs, more processing
- **Recommendation**: Keep at default unless arc quality issues

#### $13 - Report Inches
- **Range**: 0 (mm) or 1 (inches)
- **Default**: 0
- **Description**: Units for status reports
- **Note**: Independent of G-code units (G20/G21)

### Limits and Homing

#### $20 - Soft Limits Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables software-based travel limits
- **Requirement**: Homing must be completed first
- **Safety**: Prevents crashes when properly configured

#### $21 - Hard Limits Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables limit switch monitoring during operation
- **Effect**: Triggers alarm on limit switch activation

#### $22 - Homing Cycle Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 1
- **Description**: Enables homing cycle functionality
- **Requirement**: Must have limit switches properly configured

#### $23 - Homing Direction Invert
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Sets homing direction for each axis
- **Usage**: 1=positive direction, 0=negative direction per axis

#### $24 - Homing Feed Rate
- **Range**: 1-65535 (mm/min)
- **Default**: 25
- **Description**: Slow precision approach rate during homing
- **Usage**: Final approach speed to limit switches

#### $25 - Homing Seek Rate
- **Range**: 1-65535 (mm/min)
- **Default**: 500
- **Description**: Fast search rate during homing
- **Usage**: Initial search speed to find limit switches

#### $26 - Homing Debounce Delay
- **Range**: 0-65535 (milliseconds)
- **Default**: 250
- **Description**: Delay for limit switch debouncing
- **Usage**: Increase if false triggers occur

#### $27 - Homing Pull-off Distance
- **Range**: 0-1000 (mm)
- **Default**: 1.0
- **Description**: Distance to back off from limit switches after homing
- **Safety**: Ensures switches are not left activated

### Per-Axis Settings ($100-$199)

Each axis has four dedicated settings with 10-number increments:

#### Travel Resolution (Steps per mm)
- **$100**: X-axis steps per mm
- **$101**: Y-axis steps per mm  
- **$102**: Z-axis steps per mm
- **$103**: A-axis steps per mm (if configured)
- **$104**: B-axis steps per mm (if configured)
- **$105**: C-axis steps per mm (if configured)

**Range**: 0.1-10000.0
**Calculation**: (Motor steps/rev × Microstepping) ÷ Travel per revolution
**Examples**:
- Belt drive: (200 × 16) ÷ (2mm × 20 teeth) = 80 steps/mm
- Leadscrew: (200 × 16) ÷ 8mm = 400 steps/mm

#### Maximum Rate (mm/min)
- **$110**: X-axis maximum rate
- **$111**: Y-axis maximum rate
- **$112**: Z-axis maximum rate
- **$113**: A-axis maximum rate
- **$114**: B-axis maximum rate
- **$115**: C-axis maximum rate

**Range**: 1-65535 mm/min
**Usage**: Limits maximum feed rate for rapid moves
**Safety**: Set conservatively initially, increase as verified

#### Acceleration (mm/sec²)
- **$120**: X-axis acceleration
- **$121**: Y-axis acceleration
- **$122**: Z-axis acceleration
- **$123**: A-axis acceleration
- **$124**: B-axis acceleration
- **$125**: C-axis acceleration

**Range**: 1-65535 mm/sec²
**Usage**: Controls acceleration and deceleration rates
**Tuning**: Balance speed vs. mechanical stress

#### Maximum Travel (mm)
- **$130**: X-axis maximum travel
- **$131**: Y-axis maximum travel
- **$132**: Z-axis maximum travel
- **$133**: A-axis maximum travel
- **$134**: B-axis maximum travel
- **$135**: C-axis maximum travel

**Range**: 0-1000000 mm
**Usage**: Defines soft limit boundaries
**Requirement**: Requires homing and soft limits enabled

---

## Extended grblHAL Settings ($200-$399)

### Advanced Motion Control

#### $28 - G73 Retract Distance (grblHAL Extension)
- **Range**: 0-1000 (mm)
- **Default**: 1.0
- **Description**: Retract distance for G73 peck drilling cycle

#### $37 - Steppers Deenergize Mask (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 255 (all axes)
- **Description**: Defines which steppers to deenergize when motion completes
- **Usage**: X=1, Y=2, Z=4, etc. Sum for multiple axes
- **Power Saving**: Reduces motor heating and power consumption

#### $38 - Spindle Encoder PPR (grblHAL Extension)
- **Range**: 1-10000
- **Default**: Driver-dependent
- **Description**: Spindle encoder pulses per revolution
- **Usage**: Required for spindle-synchronized motion (threading)

#### $39 - Enable Legacy RT Commands (grblHAL Extension)
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 1
- **Description**: Enables printable real-time command characters
- **Characters**: `?` (status), `!` (feed hold), `~` (cycle start)
- **Safety**: Disable if characters appear in G-code comments

#### $40 - Limit Jog Commands (grblHAL Extension)
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables soft limits for jogging commands
- **Safety**: Prevents jogging beyond travel limits

### Homing Configuration Extensions

#### $43 - Homing Locate Cycles (grblHAL Extension)
- **Range**: 0-255
- **Default**: 1
- **Description**: Number of homing locate cycles per axis
- **Usage**: Multiple cycles can improve precision

#### $44-$49 - Homing Order (grblHAL Extension)
Priority-based homing order (lowest number executed first):
- **$44**: 1st priority axis mask
- **$45**: 2nd priority axis mask  
- **$46**: 3rd priority axis mask
- **$47**: 4th priority axis mask
- **$48**: 5th priority axis mask
- **$49**: 6th priority axis mask

**Range**: 0-255 (bitmask)
**Usage**: Allows custom homing sequences
**Example**: `$44=4` (Z first), `$45=3` (X,Y together)

### Jogging Defaults

#### $50 - Jogging Step Speed (grblHAL Extension)
- **Range**: 1-65535 (mm/min)
- **Default**: 100
- **Description**: Default speed for step jogging
- **Usage**: Used by jogging interfaces

#### $51 - Jogging Slow Speed (grblHAL Extension)
- **Range**: 1-65535 (mm/min)
- **Default**: 25
- **Description**: Default speed for slow continuous jogging

#### $52 - Jogging Fast Speed (grblHAL Extension)
- **Range**: 1-65535 (mm/min)
- **Default**: 1000
- **Description**: Default speed for fast continuous jogging

#### $53 - Jogging Step Distance (grblHAL Extension)
- **Range**: 0.001-1000 (mm)
- **Default**: 0.1
- **Description**: Default distance for step jogging

#### $54 - Jogging Slow Distance (grblHAL Extension)
- **Range**: 0.001-1000 (mm)
- **Default**: 1.0
- **Description**: Default distance for slow jogging

#### $55 - Jogging Fast Distance (grblHAL Extension)
- **Range**: 0.001-1000 (mm)
- **Default**: 10.0
- **Description**: Default distance for fast jogging

### System Configuration

#### $338 - Planner Buffer Blocks (grblHAL Extension)
- **Range**: 16-1000
- **Default**: 100
- **Description**: Number of linear motions in planner buffer
- **Impact**: More blocks = better lookahead, more RAM usage
- **Tuning**: Increase if complex paths have speed variations

#### $376 - Disable G92 Persistence (grblHAL Extension)
- **Range**: 0 (enabled) or 1 (disabled)
- **Default**: 0 (when COMPATIBILITY_LEVEL ≤ 1)
- **Description**: Controls whether G92 offsets are saved to NVS
- **Compatibility**: Disabled by default for grbl 1.1 compatibility

---

## Plugin Settings ($400+)

Plugin settings are dynamically allocated based on enabled plugins. Setting numbers may vary depending on plugin loading order.

---

## Networking Plugin Settings

The networking plugin provides Ethernet, WiFi, and various protocol support.

### Network Interface Configuration

#### $70 - Network Services Enable
- **Range**: 0-31 (bitmask)
- **Default**: 0
- **Description**: Enables network services
- **Services**:
  - 1: Telnet
  - 2: WebSocket  
  - 4: HTTP
  - 8: FTP
  - 16: mDNS
- **Example**: `$70=15` enables Telnet, WebSocket, HTTP, FTP

#### $71 - Network Mode (WiFi)
- **Range**: 0-2
- **Values**: 0=Off, 1=Station, 2=Access Point
- **Description**: WiFi operating mode

#### $72 - WiFi SSID
- **Range**: String (max 32 characters)
- **Description**: WiFi network name for station mode

#### $73 - WiFi Password  
- **Range**: String (max 63 characters)
- **Description**: WiFi network password

#### $74 - WiFi Country Code
- **Range**: String (2 characters)
- **Default**: "US"
- **Description**: WiFi regulatory domain

### IP Configuration

#### $75 - IP Mode
- **Range**: 0-2
- **Values**: 0=Static, 1=DHCP, 2=AutoIP
- **Default**: 1 (DHCP)
- **Description**: IP address assignment method

#### $76 - IP Address
- **Range**: IPv4 address string
- **Default**: "192.168.1.210"
- **Description**: Static IP address (when IP mode = 0)

#### $77 - Gateway Address
- **Range**: IPv4 address string  
- **Default**: "192.168.1.1"
- **Description**: Network gateway address

#### $78 - Subnet Mask
- **Range**: IPv4 address string
- **Default**: "255.255.255.0"
- **Description**: Network subnet mask

### Access Point Configuration

#### $311 - AP SSID
- **Range**: String (max 32 characters)
- **Default**: "grblHAL_AP"
- **Description**: Access Point network name

#### $312 - AP Password
- **Range**: String (8-63 characters)
- **Default**: "grblhal"
- **Description**: Access Point password

#### $313 - AP IP Address
- **Range**: IPv4 address string
- **Default**: "192.168.4.1"
- **Description**: Access Point IP address

#### $314 - AP Gateway
- **Range**: IPv4 address string
- **Default**: "192.168.4.1"
- **Description**: Access Point gateway address

#### $315 - AP Subnet Mask
- **Range**: IPv4 address string
- **Default**: "255.255.255.0"
- **Description**: Access Point subnet mask

### Protocol Port Configuration

#### $305 - Telnet Port
- **Range**: 1-65535
- **Default**: 23
- **Description**: TCP port for Telnet protocol

#### $306 - HTTP Port
- **Range**: 1-65535  
- **Default**: 80
- **Description**: TCP port for HTTP protocol

#### $307 - WebSocket Port
- **Range**: 1-65535
- **Default**: 81
- **Description**: TCP port for WebSocket protocol

#### $308 - FTP Port
- **Range**: 1-65535
- **Default**: 21
- **Description**: TCP port for FTP protocol

### Network Security and Features

#### $316 - Network Hostname
- **Range**: String (max 32 characters)
- **Default**: "grblHAL"
- **Description**: mDNS hostname for network discovery

#### $317 - WebDAV Authentication
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables WebDAV authentication

---

## Spindle Plugin Settings

The spindle plugin adds support for VFD control via Modbus and multiple spindle management.

### VFD Communication Settings

#### $360 - Modbus Address (Legacy)
- **Range**: 1-247
- **Default**: 1
- **Description**: Modbus slave address for single VFD configuration
- **Note**: Used when only one VFD spindle is configured

#### $395 - Active Spindle Selection
- **Range**: 0-15 (spindle ID)
- **Default**: 0
- **Description**: Selects active spindle when multiple spindles configured
- **Usage**: Changes with `$$=395` to see available spindles

#### $461 - RPM to Hz Relationship  
- **Range**: 1-3600
- **Default**: 60
- **Description**: RPM to frequency conversion factor
- **Calculation**: Motor RPM = Frequency × 60 ÷ Pole pairs
- **Usage**: Depends on motor specifications

### VFD Modbus Configuration

#### $462 - Run/Stop Register
- **Range**: 1-65535
- **Default**: 8192
- **Description**: Modbus register for spindle run/stop control

#### $463 - Set Frequency Register
- **Range**: 1-65535
- **Default**: 8193  
- **Description**: Modbus register for frequency setpoint

#### $464 - Get Frequency Register
- **Range**: 1-65535
- **Default**: 8451
- **Description**: Modbus register for frequency readback

### Multi-Spindle Configuration

#### $476 - VFD Address Spindle 0
- **Range**: 1-247
- **Default**: 1
- **Description**: Modbus address for VFD bound to spindle 0

#### $477 - VFD Address Spindle 1
- **Range**: 1-247
- **Default**: 2
- **Description**: Modbus address for VFD bound to spindle 1

#### $478 - VFD Address Spindle 2
- **Range**: 1-247
- **Default**: 3
- **Description**: Modbus address for VFD bound to spindle 2

#### $479 - VFD Address Spindle 3
- **Range**: 1-247
- **Default**: 4
- **Description**: Modbus address for VFD bound to spindle 3

### Spindle Selection by Tool Number

#### $511 - Spindle 1 Enable
- **Range**: 0-15 (spindle ID)
- **Description**: Binds spindle number 1 to specified spindle ID

#### $512 - Spindle 2 Enable
- **Range**: 0-15 (spindle ID)
- **Description**: Binds spindle number 2 to specified spindle ID

#### $513 - Spindle 3 Enable
- **Range**: 0-15 (spindle ID)
- **Description**: Binds spindle number 3 to specified spindle ID

#### $520 - Tool Number Range Spindle 0
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 0
- **Usage**: Tools ≥ this number select spindle 0

#### $521 - Tool Number Range Spindle 1
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 1

#### $522 - Tool Number Range Spindle 2
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 2

#### $523 - Tool Number Range Spindle 3
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 3

---

## Trinamic Plugin Settings

The Trinamic plugin provides comprehensive support for TMC2130, TMC2209, and TMC5160 stepper drivers.

### Driver Enable and Basic Configuration

Settings are per-axis with standard axis numbering (X=0, Y=1, Z=2, etc.).

#### Driver Enable Settings
- **$480**: X-axis Trinamic driver enable (0=disabled, 1=enabled)
- **$481**: Y-axis Trinamic driver enable
- **$482**: Z-axis Trinamic driver enable
- **$483**: A-axis Trinamic driver enable
- **$484**: B-axis Trinamic driver enable
- **$485**: C-axis Trinamic driver enable

#### Current Settings (mA)
- **$486**: X-axis motor current
- **$487**: Y-axis motor current  
- **$488**: Z-axis motor current
- **$489**: A-axis motor current
- **$490**: B-axis motor current
- **$491**: C-axis motor current

**Range**: 50-3000 mA (driver dependent)
**Note**: Current is not permanently stored, reset on power cycle

#### Microstep Settings
- **$492**: X-axis microsteps
- **$493**: Y-axis microsteps
- **$494**: Z-axis microsteps
- **$495**: A-axis microsteps  
- **$496**: B-axis microsteps
- **$497**: C-axis microsteps

**Valid Values**: 1, 2, 4, 8, 16, 32, 64, 128, 256
**Default**: 16
**Effect**: Higher values = smoother motion, lower torque

### StallGuard Settings (Sensorless Homing)

#### StallGuard Threshold
- **$498**: X-axis StallGuard threshold
- **$499**: Y-axis StallGuard threshold
- **$500**: Z-axis StallGuard threshold
- **$501**: A-axis StallGuard threshold
- **$502**: B-axis StallGuard threshold  
- **$503**: C-axis StallGuard threshold

**Range**: 
- TMC2130/TMC5160: -64 to 63
- TMC2209: 0 to 255
**Usage**: Lower values = more sensitive detection
**Tuning**: Requires careful calibration per axis

#### Homing Feed Sensitivity
- **$504**: X-axis homing sensitivity
- **$505**: Y-axis homing sensitivity
- **$506**: Z-axis homing sensitivity
- **$507**: A-axis homing sensitivity
- **$508**: B-axis homing sensitivity
- **$509**: C-axis homing sensitivity

**Range**: Same as StallGuard threshold
**Description**: StallGuard threshold during homing
**Usage**: Often different from normal operation threshold

### Advanced Trinamic Settings

#### Hybrid Threshold (TMC2130/TMC5160)
- **$510**: X-axis hybrid threshold
- **$511**: Y-axis hybrid threshold
- **$512**: Z-axis hybrid threshold
- **$513**: A-axis hybrid threshold
- **$514**: B-axis hybrid threshold
- **$515**: C-axis hybrid threshold

**Range**: 0-65535 (steps/second)
**Description**: Speed threshold for switching to SpreadCycle mode
**Usage**: 0 = always StealthChop, high value = always SpreadCycle

#### ChopConf Settings (Extended Settings)
When `TRINAMIC_EXTENDED_SETTINGS` is enabled:

**$516**: TOFF - Off time setting
**$517**: TBL - Blanking time  
**$518**: CHM - Chopper mode
**$519**: HSTRT - Hysteresis start
**$520**: HEND - Hysteresis end
**$521**: HDEC - Hysteresis decrement
**$522**: RNDTF - Random TOFF

**Usage**: Advanced motor tuning parameters
**Caution**: Incorrect values can damage drivers or motors

#### CoolConf Settings (Extended Settings)
**$523**: SEMIN - Minimum StallGuard value
**$524**: SEUP - Current increment steps
**$525**: SEMAX - Maximum StallGuard value  
**$526**: SEDN - Current decrement steps
**$527**: SEIMIN - Minimum current flag

**Description**: Automatic current adjustment based on load
**Usage**: Optimizes current consumption and motor heating

---

## SD Card Plugin Settings

The SD card plugin enables G-code file storage and execution from SD cards.

 commands for file operations:

#### File Management Commands
- **$F**: Mount SD card as root directory
- **$F+**: List CNC files recursively (.nc, .ncc, .ngc, .cnc, .gcode, .txt, .text, .tap, .macro)
- **$F***: List all files recursively regardless of type
- **$F=<filename>**: Run G-code file
- **$F~**: Enable rewind mode for next file
- **$F?<filename>**: Display file contents
- **$F-<filename>**: Delete file

#### File Execution Features
- **Rewind Mode**: Automatically restart file after M2 or completion
- **Pause/Resume**: Standard grblHAL pause commands work during file execution
- **Error Handling**: Stops execution on errors, maintains position

### Macro System Integration

#### Named Macros
- **Extension**: .macro files for reusable G-code snippets
- **Execution**: Can be called from G-code or commands
- **Parameters**: Support for parameterized macros

#### Pallet Shuttle Support
- **M60 Integration**: Calls named macro on M60 command
- **Configuration**: Macro name configured via plugin settings

---

## Additional Plugin Settings

### EEPROM/FRAM Plugin

#### External Storage Configuration
- **Capacity**: Supports various EEPROM and FRAM sizes
- **Interface**: I2C communication
- **Addresses**: Configurable I2C addresses
- **Usage**: Extends settings storage, odometer data

### Keypad Plugin

#### I2C Keypad Support
- **Interface**: I2C communication protocol
- **Functions**: Override controls, jogging, macro execution
- **Mapping**: Configurable key assignments
- **Display**: Optional LCD/OLED display support

#### Settings Range
- Plugin allocates settings dynamically
- Configuration via dedicated settings menu
- Real-time key mapping updates

### Encoder Plugin

#### Encoder Configuration
- **Types**: Quadrature encoders for overrides
- **Resolution**: Configurable pulses per detent
- **Functions**: Feed rate, spindle speed, rapid override
- **Acceleration**: Variable sensitivity based on rotation speed

### Bluetooth Plugin

#### Wireless Communication
- **Protocol**: Serial over Bluetooth
- **Pairing**: Standard Bluetooth pairing process
- **Range**: Typical Bluetooth range limitations
- **Compatibility**: Works with standard Bluetooth terminal apps

### Laser Plugin

#### Laser-Specific Settings
- **PPI Control**: Pulses Per Inch for raster engraving
- **Power Scaling**: Dynamic power adjustment
- **Mode Selection**: Raster vs. vector operation
- **Safety**: Laser safety interlocks

#### Coolant Integration
- **Air Assist**: Coolant output controls air assist
- **Safety**: Interlocked with laser operation
- **Flow Monitoring**: Optional flow switch monitoring

### Plasma Plugin

#### Torch Height Control (THC)
- **Voltage Monitoring**: Arc voltage feedback
- **Height Adjustment**: Automatic Z-axis correction
- **Pierce Control**: Pierce delay and current
- **Cut Quality**: Voltage-based cut quality monitoring

#### Safety Features
- **Arc Transfer**: Confirms arc establishment
- **Fault Detection**: Monitors for arc loss
- **Emergency Stop**: Immediate shutdown on faults

### Odometer Plugin

#### Distance Tracking
- **Total Distance**: Cumulative distance per axis
- **Machining Time**: Actual cutting time tracking
- **Storage**: Uses EEPROM/FRAM for persistence
- **Reset**: Commands to reset counters

#### Settings
- **Enable/Disable**: Per-axis tracking control
- **Display**: Integration with status reports
- **Backup**: Settings for data backup intervals

### Fan Plugin

#### Cooling Control
- **PWM Control**: Variable speed fan control
- **Temperature**: Temperature-based automatic control
- **Manual Override**: Manual fan speed setting
- **Multiple Fans**: Support for multiple fan outputs

### WebUI Plugin

#### Web Interface Configuration
- **Port Settings**: HTTP and WebSocket port configuration
- **File System**: Integration with SD card or flash storage
- **Security**: Optional authentication
- **Features**: File upload, settings management, control interface

#### Installation Requirements
- **HTTP Port**: $306 = 80
- **WebSocket Port**: $307 = 81  
- **Services**: $70 = 15 (HTTP + WebSocket enabled)
- **Storage**: SD card or flash file system required

---

## Setting Categories and Ranges

### Core Settings Summary

| Range | Category | Description |
|-------|----------|-------------|
| $0-$49 | Core Motion | Basic motion control and I/O |
| $50-$99 | Extensions | grblHAL-specific extensions |
| $100-$199 | Per-Axis | Individual axis configuration |
| $200-$299 | Reserved | Future core extensions |
| $300-$399 | System | Advanced system configuration |
| $400+ | Plugins | Plugin-specific settings |

### Plugin Setting Allocation

Plugin settings are dynamically allocated based on:
- **Load Order**: Settings assigned as plugins initialize
- **Plugin Type**: Different plugins get different setting ranges
- **Conflicts**: System prevents setting number conflicts
- **Persistence**: Plugin settings saved to NVS like core settings

### Setting Validation

#### Range Checking
- **Automatic**: All settings have defined valid ranges
- **Rejection**: Invalid values rejected with error messages
- **Defaults**: Invalid settings reset to defaults on error

#### Dependencies
- **Prerequisites**: Some settings require others to be enabled
- **Conflicts**: Mutually exclusive settings prevented
- **Hardware**: Settings validated against hardware capabilities

### Backup and Restore

#### Export/Import Commands
- **$EXPORT**: Export all settings to text format
- **$IMPORT**: Import settings from text format
- **Validation**: Imported settings validated before application
- **Safety**: Invalid imports rejected, system remains stable

#### Version Compatibility
- **Version Tracking**: Settings tagged with firmware version
- **Migration**: Automatic migration between compatible versions
- **Reset Protection**: Settings preserved across firmware updates when possible

---

## Configuration Examples

### Basic 3-Axis Mill Setup

```
# Motion settings
$0=10          # Step pulse time
$1=255         # Keep steppers enabled
$2=0           # No step inversion
$3=0           # No direction inversion

# Axis configuration (example values)
$100=80        # X steps/mm
$101=80        # Y steps/mm  
$102=400       # Z steps/mm (leadscrew)
$110=6000      # X max rate
$111=6000      # Y max rate
$112=1000      # Z max rate
$120=400       # X acceleration
$121=400       # Y acceleration
$122=200       # Z acceleration
$130=300       # X max travel
$131=200       # Y max travel
$132=100       # Z max travel

# Homing setup
$20=1          # Soft limits
$21=1          # Hard limits  
$22=1          # Homing enabled
$23=0          # Home to negative
$24=50         # Homing feed rate
$25=1000       # Homing seek rate
```

### Ethernet-Enabled Setup

```
# Network configuration
$70=15         # Enable Telnet, WebSocket, HTTP, FTP
$75=1          # DHCP mode
$305=23        # Telnet port
$306=80        # HTTP port
$307=81        # WebSocket port
```

### VFD Spindle Configuration

```
# VFD setup
$395=1         # Select VFD spindle
$360=1         # Modbus address
$461=60        # RPM to Hz ratio
$30=24000      # Max spindle RPM
$31=1000       # Min spindle RPM
```

### Trinamic Driver Setup

```
# Enable Trinamic drivers
$480=1         # X-axis TMC enabled
$481=1         # Y-axis TMC enabled
$482=1         # Z-axis TMC enabled

# Set motor current (mA)
$486=1000      # X-axis current
$487=1000      # Y-axis current
$488=800       # Z-axis current

# Microstep configuration
$492=16        # X-axis microsteps
$493=16        # Y-axis microsteps
$494=16        # Z-axis microsteps
```

---

## Best Practices

### Initial Configuration

1. **Start Conservative**: Use low speeds and accelerations initially
2. **Test Incrementally**: Increase settings gradually after testing
3. **Backup Settings**: Export settings before major changes
4. **Document Changes**: Keep notes on custom configurations

### Safety Considerations

1. **E-Stop Testing**: Verify emergency stop functionality
2. **Limit Switch Testing**: Test all limit switches before operation
3. **Soft Limits**: Configure and test soft limits with homing
4. **Current Limits**: Set appropriate motor currents to prevent overheating

### Performance Optimization

1. **Step Rates**: Optimize step pulse timing for high speeds
2. **Buffer Sizes**: Adjust planner buffer for complex toolpaths
3. **Network Settings**: Use Ethernet for critical applications
4. **Plugin Selection**: Enable only needed plugins to conserve resources

### Troubleshooting Settings

1. **Default Restoration**: Use `$RST=# grblHAL Configuration Settings Reference Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Core Motion Settings ($0-$199)](#core-motion-settings-0-199)
3. [Extended grblHAL Settings ($200-$399)](#extended-grblhal-settings-200-399)
4. [Plugin Settings ($400+)](#plugin-settings-400)
5. [Networking Plugin Settings](#networking-plugin-settings)
6. [Spindle Plugin Settings](#spindle-plugin-settings)
7. [Trinamic Plugin Settings](#trinamic-plugin-settings)
8. [SD Card Plugin Settings](#sd-card-plugin-settings)
9. [Additional Plugin Settings](#additional-plugin-settings)

---

## Introduction

This reference guide provides comprehensive documentation of all grblHAL configuration settings, including core functionality and plugin extensions. Settings are accessed via the `$$` command and modified using `$<number>=<value>` format.

### Key Features of grblHAL Settings
- **Backward Compatibility**: Maintains compatibility with grbl 1.1 settings
- **Dynamic Extensions**: Plugin-specific settings allocated automatically
- **Real-time Updates**: Most settings take effect immediately
- **Validation**: Built-in range checking and validation
- **Persistence**: Settings stored in non-volatile memory

### Important Notes
- **Settings Version**: Current version is 22 (as of build 20250604)
- **Reset Behavior**: Settings reset may occur on firmware updates
- **Backup/Restore**: Use `$EXPORT` and `$IMPORT` commands for backup
- **Range Validation**: Invalid values are rejected with error messages

---

## Core Motion Settings ($0-$199)

### Step Pulse and Timing

#### $0 - Step Pulse Time (microseconds)
- **Range**: 2-1000
- **Default**: 10
- **Description**: Duration of step pulse signals
- **Usage**: High step rates (>80kHz) require values <10μs
- **Example**: `$0=5` for 200kHz step rates

#### $1 - Step Idle Delay (milliseconds)
- **Range**: 0-65535, 255=infinite
- **Default**: 25
- **Description**: Delay before disabling steppers after motion stops
- **Usage**: Set to 255 to keep steppers always enabled
- **Power Saving**: Lower values save power but may cause position loss

#### $29 - Step Pulse Delay (grblHAL Extension)
- **Range**: 0-1000 (microseconds)
- **Default**: 0
- **Description**: Delay between direction and step signals
- **Usage**: Required for some stepper drivers that need setup time

### Direction and Inversion Control

#### $2 - Step Port Invert Mask
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts step pulse polarity per axis
- **Calculation**: X=1, Y=2, Z=4, A=8, B=16, C=32
- **Example**: `$2=5` inverts X and Z axes (1+4=5)

#### $3 - Direction Port Invert Mask
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts direction signal polarity per axis
- **Usage**: Change axis direction without rewiring
- **Example**: `$3=2` inverts Y-axis direction only

#### $4 - Step Enable Invert
- **Range**: 0 (normal low) or 1 (inverted high)
- **Default**: 0
- **Description**: Inverts stepper enable pin logic
- **Usage**: Some drivers require high signal to enable

#### $8 - Ganged Direction Invert Mask (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts direction for ganged/dual motor axes
- **Usage**: Compensates for reversed motor mounting

### Input Signal Configuration

#### $5 - Limit Pins Invert Mask
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Inverts limit switch input logic
- **Note**: grblHAL defaults to normally closed (NC) switches
- **Quick Setup**: Use with pull-up disable settings for NO switches

#### $6 - Probe Pin Invert
- **Range**: 0 (NC) or 1 (NO)
- **Default**: 0
- **Description**: Inverts probe input logic
- **Usage**: Set to 1 for normally open probe switches

#### $14 - Control Signals Invert Mask
- **Range**: 0-1023 (bitmask)
- **Default**: 0
- **Description**: Inverts control input signals
- **Signals**: Reset=1, Feed Hold=2, Cycle Start=4, Safety Door=8, Block Delete=16, Optional Stop=32, EStop=64, Motor Warning=128, Motor Fault=256, Probe Disconnect=512
- **Quick Fix**: `$14=73` for NO switches when testing without connections

#### $17 - Control Pull-up Disable Mask (grblHAL Extension)
- **Range**: 0-1023 (bitmask)
- **Default**: 0
- **Description**: Disables internal pull-up resistors for control inputs
- **Warning**: Requires external pull-down resistors when disabled

#### $18 - Limit Pull-up Disable Mask (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Disables internal pull-up resistors for limit switches
- **Warning**: Requires external pull-down resistors when disabled

#### $19 - Probe Pull-up Disable (grblHAL Extension)
- **Range**: 0 (enabled) or 1 (disabled)
- **Default**: 0
- **Description**: Disables internal pull-up resistor for probe input

### Spindle and PWM Control

#### $7 - Spindle PWM Options (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Various spindle PWM behavior options
- **Flags**: Implementation-dependent, consult driver documentation

#### $9 - Spindle PWM Extended Options (grblHAL Extension)
- **Range**: 0-255 (bitmask)  
- **Default**: 0
- **Description**: Additional spindle PWM configuration options

#### $30 - Spindle PWM Maximum
- **Range**: 0-1000
- **Default**: 1000
- **Description**: Maximum spindle speed for PWM output
- **Usage**: Corresponds to S1000 command

#### $31 - Spindle PWM Minimum  
- **Range**: 0-1000
- **Default**: 0
- **Description**: Minimum spindle speed for PWM output
- **Usage**: Prevents stalling at low speeds

#### $32 - Laser Mode Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables laser-specific behaviors
- **Features**: Dynamic power scaling, M4 support

#### $33 - Spindle PWM Frequency (grblHAL Extension)
- **Range**: Driver-dependent (typically 100-40000 Hz)
- **Default**: Driver-dependent
- **Description**: PWM frequency for spindle control
- **Usage**: Higher frequency reduces motor noise

#### $34 - Spindle PWM Off Value (grblHAL Extension)
- **Range**: 0-100 (percentage)
- **Default**: 0
- **Description**: PWM duty cycle when spindle is off

#### $35 - Spindle PWM Min Value (grblHAL Extension)
- **Range**: 0-100 (percentage)
- **Default**: 0
- **Description**: Minimum PWM duty cycle percentage

#### $36 - Spindle PWM Max Value (grblHAL Extension)
- **Range**: 0-100 (percentage) 
- **Default**: 100
- **Description**: Maximum PWM duty cycle percentage

#### $15 - Coolant Invert Mask
- **Range**: 0-7 (bitmask)
- **Default**: 0
- **Description**: Inverts coolant control outputs
- **Signals**: Flood=1, Mist=2, Additional outputs driver-dependent

#### $16 - Spindle Invert Mask
- **Range**: 0-7 (bitmask)
- **Default**: 0
- **Description**: Inverts spindle control outputs
- **Signals**: Enable=1, Direction=2, Additional signals driver-dependent

### Motion Planning and Control

#### $10 - Status Report Mask
- **Range**: 0-2047 (bitmask)
- **Default**: 1
- **Description**: Configures real-time status report contents
- **Flags**:
  - 0: Work position (WPos)
  - 1: Machine position (MPos) 
  - 2: Planner buffer status
  - 3: RX buffer status
  - 4: Limit pin status
  - 5: Current feed and speed
  - 6: Pin states on mode change
  - 7: Override values
  - 8: Probe coordinates
  - 9: Buffer sync on WPos change
  - 10: Parser state on WPos change
  - 11: Substates in Run mode (grblHAL)

#### $11 - Junction Deviation
- **Range**: 0.001-2.0 (mm)
- **Default**: 0.01
- **Description**: Cornering junction deviation for path planning
- **Effect**: Higher values = faster cornering, lower precision
- **Tuning**: Start with 0.01mm, increase gradually if needed

#### $12 - Arc Tolerance
- **Range**: 0.002-1.0 (mm)
- **Default**: 0.002  
- **Description**: Arc segmentation tolerance
- **Effect**: Smaller values = smoother arcs, more processing
- **Recommendation**: Keep at default unless arc quality issues

#### $13 - Report Inches
- **Range**: 0 (mm) or 1 (inches)
- **Default**: 0
- **Description**: Units for status reports
- **Note**: Independent of G-code units (G20/G21)

### Limits and Homing

#### $20 - Soft Limits Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables software-based travel limits
- **Requirement**: Homing must be completed first
- **Safety**: Prevents crashes when properly configured

#### $21 - Hard Limits Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables limit switch monitoring during operation
- **Effect**: Triggers alarm on limit switch activation

#### $22 - Homing Cycle Enable
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 1
- **Description**: Enables homing cycle functionality
- **Requirement**: Must have limit switches properly configured

#### $23 - Homing Direction Invert
- **Range**: 0-255 (bitmask)
- **Default**: 0
- **Description**: Sets homing direction for each axis
- **Usage**: 1=positive direction, 0=negative direction per axis

#### $24 - Homing Feed Rate
- **Range**: 1-65535 (mm/min)
- **Default**: 25
- **Description**: Slow precision approach rate during homing
- **Usage**: Final approach speed to limit switches

#### $25 - Homing Seek Rate
- **Range**: 1-65535 (mm/min)
- **Default**: 500
- **Description**: Fast search rate during homing
- **Usage**: Initial search speed to find limit switches

#### $26 - Homing Debounce Delay
- **Range**: 0-65535 (milliseconds)
- **Default**: 250
- **Description**: Delay for limit switch debouncing
- **Usage**: Increase if false triggers occur

#### $27 - Homing Pull-off Distance
- **Range**: 0-1000 (mm)
- **Default**: 1.0
- **Description**: Distance to back off from limit switches after homing
- **Safety**: Ensures switches are not left activated

### Per-Axis Settings ($100-$199)

Each axis has four dedicated settings with 10-number increments:

#### Travel Resolution (Steps per mm)
- **$100**: X-axis steps per mm
- **$101**: Y-axis steps per mm  
- **$102**: Z-axis steps per mm
- **$103**: A-axis steps per mm (if configured)
- **$104**: B-axis steps per mm (if configured)
- **$105**: C-axis steps per mm (if configured)

**Range**: 0.1-10000.0
**Calculation**: (Motor steps/rev × Microstepping) ÷ Travel per revolution
**Examples**:
- Belt drive: (200 × 16) ÷ (2mm × 20 teeth) = 80 steps/mm
- Leadscrew: (200 × 16) ÷ 8mm = 400 steps/mm

#### Maximum Rate (mm/min)
- **$110**: X-axis maximum rate
- **$111**: Y-axis maximum rate
- **$112**: Z-axis maximum rate
- **$113**: A-axis maximum rate
- **$114**: B-axis maximum rate
- **$115**: C-axis maximum rate

**Range**: 1-65535 mm/min
**Usage**: Limits maximum feed rate for rapid moves
**Safety**: Set conservatively initially, increase as verified

#### Acceleration (mm/sec²)
- **$120**: X-axis acceleration
- **$121**: Y-axis acceleration
- **$122**: Z-axis acceleration
- **$123**: A-axis acceleration
- **$124**: B-axis acceleration
- **$125**: C-axis acceleration

**Range**: 1-65535 mm/sec²
**Usage**: Controls acceleration and deceleration rates
**Tuning**: Balance speed vs. mechanical stress

#### Maximum Travel (mm)
- **$130**: X-axis maximum travel
- **$131**: Y-axis maximum travel
- **$132**: Z-axis maximum travel
- **$133**: A-axis maximum travel
- **$134**: B-axis maximum travel
- **$135**: C-axis maximum travel

**Range**: 0-1000000 mm
**Usage**: Defines soft limit boundaries
**Requirement**: Requires homing and soft limits enabled

---

## Extended grblHAL Settings ($200-$399)

### Advanced Motion Control

#### $28 - G73 Retract Distance (grblHAL Extension)
- **Range**: 0-1000 (mm)
- **Default**: 1.0
- **Description**: Retract distance for G73 peck drilling cycle

#### $37 - Steppers Deenergize Mask (grblHAL Extension)
- **Range**: 0-255 (bitmask)
- **Default**: 255 (all axes)
- **Description**: Defines which steppers to deenergize when motion completes
- **Usage**: X=1, Y=2, Z=4, etc. Sum for multiple axes
- **Power Saving**: Reduces motor heating and power consumption

#### $38 - Spindle Encoder PPR (grblHAL Extension)
- **Range**: 1-10000
- **Default**: Driver-dependent
- **Description**: Spindle encoder pulses per revolution
- **Usage**: Required for spindle-synchronized motion (threading)

#### $39 - Enable Legacy RT Commands (grblHAL Extension)
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 1
- **Description**: Enables printable real-time command characters
- **Characters**: `?` (status), `!` (feed hold), `~` (cycle start)
- **Safety**: Disable if characters appear in G-code comments

#### $40 - Limit Jog Commands (grblHAL Extension)
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables soft limits for jogging commands
- **Safety**: Prevents jogging beyond travel limits

### Homing Configuration Extensions

#### $43 - Homing Locate Cycles (grblHAL Extension)
- **Range**: 0-255
- **Default**: 1
- **Description**: Number of homing locate cycles per axis
- **Usage**: Multiple cycles can improve precision

#### $44-$49 - Homing Order (grblHAL Extension)
Priority-based homing order (lowest number executed first):
- **$44**: 1st priority axis mask
- **$45**: 2nd priority axis mask  
- **$46**: 3rd priority axis mask
- **$47**: 4th priority axis mask
- **$48**: 5th priority axis mask
- **$49**: 6th priority axis mask

**Range**: 0-255 (bitmask)
**Usage**: Allows custom homing sequences
**Example**: `$44=4` (Z first), `$45=3` (X,Y together)

### Jogging Defaults

#### $50 - Jogging Step Speed (grblHAL Extension)
- **Range**: 1-65535 (mm/min)
- **Default**: 100
- **Description**: Default speed for step jogging
- **Usage**: Used by jogging interfaces

#### $51 - Jogging Slow Speed (grblHAL Extension)
- **Range**: 1-65535 (mm/min)
- **Default**: 25
- **Description**: Default speed for slow continuous jogging

#### $52 - Jogging Fast Speed (grblHAL Extension)
- **Range**: 1-65535 (mm/min)
- **Default**: 1000
- **Description**: Default speed for fast continuous jogging

#### $53 - Jogging Step Distance (grblHAL Extension)
- **Range**: 0.001-1000 (mm)
- **Default**: 0.1
- **Description**: Default distance for step jogging

#### $54 - Jogging Slow Distance (grblHAL Extension)
- **Range**: 0.001-1000 (mm)
- **Default**: 1.0
- **Description**: Default distance for slow jogging

#### $55 - Jogging Fast Distance (grblHAL Extension)
- **Range**: 0.001-1000 (mm)
- **Default**: 10.0
- **Description**: Default distance for fast jogging

### System Configuration

#### $338 - Planner Buffer Blocks (grblHAL Extension)
- **Range**: 16-1000
- **Default**: 100
- **Description**: Number of linear motions in planner buffer
- **Impact**: More blocks = better lookahead, more RAM usage
- **Tuning**: Increase if complex paths have speed variations

#### $376 - Disable G92 Persistence (grblHAL Extension)
- **Range**: 0 (enabled) or 1 (disabled)
- **Default**: 0 (when COMPATIBILITY_LEVEL ≤ 1)
- **Description**: Controls whether G92 offsets are saved to NVS
- **Compatibility**: Disabled by default for grbl 1.1 compatibility

---

## Plugin Settings ($400+)

Plugin settings are dynamically allocated based on enabled plugins. Setting numbers may vary depending on plugin loading order.

---

## Networking Plugin Settings

The networking plugin provides Ethernet, WiFi, and various protocol support.

### Network Interface Configuration

#### $70 - Network Services Enable
- **Range**: 0-31 (bitmask)
- **Default**: 0
- **Description**: Enables network services
- **Services**:
  - 1: Telnet
  - 2: WebSocket  
  - 4: HTTP
  - 8: FTP
  - 16: mDNS
- **Example**: `$70=15` enables Telnet, WebSocket, HTTP, FTP

#### $71 - Network Mode (WiFi)
- **Range**: 0-2
- **Values**: 0=Off, 1=Station, 2=Access Point
- **Description**: WiFi operating mode

#### $72 - WiFi SSID
- **Range**: String (max 32 characters)
- **Description**: WiFi network name for station mode

#### $73 - WiFi Password  
- **Range**: String (max 63 characters)
- **Description**: WiFi network password

#### $74 - WiFi Country Code
- **Range**: String (2 characters)
- **Default**: "US"
- **Description**: WiFi regulatory domain

### IP Configuration

#### $75 - IP Mode
- **Range**: 0-2
- **Values**: 0=Static, 1=DHCP, 2=AutoIP
- **Default**: 1 (DHCP)
- **Description**: IP address assignment method

#### $76 - IP Address
- **Range**: IPv4 address string
- **Default**: "192.168.1.210"
- **Description**: Static IP address (when IP mode = 0)

#### $77 - Gateway Address
- **Range**: IPv4 address string  
- **Default**: "192.168.1.1"
- **Description**: Network gateway address

#### $78 - Subnet Mask
- **Range**: IPv4 address string
- **Default**: "255.255.255.0"
- **Description**: Network subnet mask

### Access Point Configuration

#### $311 - AP SSID
- **Range**: String (max 32 characters)
- **Default**: "grblHAL_AP"
- **Description**: Access Point network name

#### $312 - AP Password
- **Range**: String (8-63 characters)
- **Default**: "grblhal"
- **Description**: Access Point password

#### $313 - AP IP Address
- **Range**: IPv4 address string
- **Default**: "192.168.4.1"
- **Description**: Access Point IP address

#### $314 - AP Gateway
- **Range**: IPv4 address string
- **Default**: "192.168.4.1"
- **Description**: Access Point gateway address

#### $315 - AP Subnet Mask
- **Range**: IPv4 address string
- **Default**: "255.255.255.0"
- **Description**: Access Point subnet mask

### Protocol Port Configuration

#### $305 - Telnet Port
- **Range**: 1-65535
- **Default**: 23
- **Description**: TCP port for Telnet protocol

#### $306 - HTTP Port
- **Range**: 1-65535  
- **Default**: 80
- **Description**: TCP port for HTTP protocol

#### $307 - WebSocket Port
- **Range**: 1-65535
- **Default**: 81
- **Description**: TCP port for WebSocket protocol

#### $308 - FTP Port
- **Range**: 1-65535
- **Default**: 21
- **Description**: TCP port for FTP protocol

### Network Security and Features

#### $316 - Network Hostname
- **Range**: String (max 32 characters)
- **Default**: "grblHAL"
- **Description**: mDNS hostname for network discovery

#### $317 - WebDAV Authentication
- **Range**: 0 (disabled) or 1 (enabled)
- **Default**: 0
- **Description**: Enables WebDAV authentication

---

## Spindle Plugin Settings

The spindle plugin adds support for VFD control via Modbus and multiple spindle management.

### VFD Communication Settings

#### $360 - Modbus Address (Legacy)
- **Range**: 1-247
- **Default**: 1
- **Description**: Modbus slave address for single VFD configuration
- **Note**: Used when only one VFD spindle is configured

#### $395 - Active Spindle Selection
- **Range**: 0-15 (spindle ID)
- **Default**: 0
- **Description**: Selects active spindle when multiple spindles configured
- **Usage**: Changes with `$$=395` to see available spindles

#### $461 - RPM to Hz Relationship  
- **Range**: 1-3600
- **Default**: 60
- **Description**: RPM to frequency conversion factor
- **Calculation**: Motor RPM = Frequency × 60 ÷ Pole pairs
- **Usage**: Depends on motor specifications

### VFD Modbus Configuration

#### $462 - Run/Stop Register
- **Range**: 1-65535
- **Default**: 8192
- **Description**: Modbus register for spindle run/stop control

#### $463 - Set Frequency Register
- **Range**: 1-65535
- **Default**: 8193  
- **Description**: Modbus register for frequency setpoint

#### $464 - Get Frequency Register
- **Range**: 1-65535
- **Default**: 8451
- **Description**: Modbus register for frequency readback

### Multi-Spindle Configuration

#### $476 - VFD Address Spindle 0
- **Range**: 1-247
- **Default**: 1
- **Description**: Modbus address for VFD bound to spindle 0

#### $477 - VFD Address Spindle 1
- **Range**: 1-247
- **Default**: 2
- **Description**: Modbus address for VFD bound to spindle 1

#### $478 - VFD Address Spindle 2
- **Range**: 1-247
- **Default**: 3
- **Description**: Modbus address for VFD bound to spindle 2

#### $479 - VFD Address Spindle 3
- **Range**: 1-247
- **Default**: 4
- **Description**: Modbus address for VFD bound to spindle 3

### Spindle Selection by Tool Number

#### $511 - Spindle 1 Enable
- **Range**: 0-15 (spindle ID)
- **Description**: Binds spindle number 1 to specified spindle ID

#### $512 - Spindle 2 Enable
- **Range**: 0-15 (spindle ID)
- **Description**: Binds spindle number 2 to specified spindle ID

#### $513 - Spindle 3 Enable
- **Range**: 0-15 (spindle ID)
- **Description**: Binds spindle number 3 to specified spindle ID

#### $520 - Tool Number Range Spindle 0
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 0
- **Usage**: Tools ≥ this number select spindle 0

#### $521 - Tool Number Range Spindle 1
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 1

#### $522 - Tool Number Range Spindle 2
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 2

#### $523 - Tool Number Range Spindle 3
- **Range**: 0-999
- **Default**: 0 (disabled)
- **Description**: Lowest tool number for selecting spindle 3

---

## Trinamic Plugin Settings

The Trinamic plugin provides comprehensive support for TMC2130, TMC2209, and TMC5160 stepper drivers.

### Driver Enable and Basic Configuration

Settings are per-axis with standard axis numbering (X=0, Y=1, Z=2, etc.).

#### Driver Enable Settings
- **$480**: X-axis Trinamic driver enable (0=disabled, 1=enabled)
- **$481**: Y-axis Trinamic driver enable
- **$482**: Z-axis Trinamic driver enable
- **$483**: A-axis Trinamic driver enable
- **$484**: B-axis Trinamic driver enable
- **$485**: C-axis Trinamic driver enable

#### Current Settings (mA)
- **$486**: X-axis motor current
- **$487**: Y-axis motor current  
- **$488**: Z-axis motor current
- **$489**: A-axis motor current
- **$490**: B-axis motor current
- **$491**: C-axis motor current

**Range**: 50-3000 mA (driver dependent)
**Note**: Current is not permanently stored, reset on power cycle

#### Microstep Settings
- **$492**: X-axis microsteps
- **$493**: Y-axis microsteps
- **$494**: Z-axis microsteps
- **$495**: A-axis microsteps  
- **$496**: B-axis microsteps
- **$497**: C-axis microsteps

**Valid Values**: 1, 2, 4, 8, 16, 32, 64, 128, 256
**Default**: 16
**Effect**: Higher values = smoother motion, lower torque

### StallGuard Settings (Sensorless Homing)

#### StallGuard Threshold
- **$498**: X-axis StallGuard threshold
- **$499**: Y-axis StallGuard threshold
- **$500**: Z-axis StallGuard threshold
- **$501**: A-axis StallGuard threshold
- **$502**: B-axis StallGuard threshold  
- **$503**: C-axis StallGuard threshold

**Range**: 
- TMC2130/TMC5160: -64 to 63
- TMC2209: 0 to 255
**Usage**: Lower values = more sensitive detection
**Tuning**: Requires careful calibration per axis

#### Homing Feed Sensitivity
- **$504**: X-axis homing sensitivity
- **$505**: Y-axis homing sensitivity
- **$506**: Z-axis homing sensitivity
- **$507**: A-axis homing sensitivity
- **$508**: B-axis homing sensitivity
- **$509**: C-axis homing sensitivity

**Range**: Same as StallGuard threshold
**Description**: StallGuard threshold during homing
**Usage**: Often different from normal operation threshold

### Advanced Trinamic Settings

#### Hybrid Threshold (TMC2130/TMC5160)
- **$510**: X-axis hybrid threshold
- **$511**: Y-axis hybrid threshold
- **$512**: Z-axis hybrid threshold
- **$513**: A-axis hybrid threshold
- **$514**: B-axis hybrid threshold
- **$515**: C-axis hybrid threshold

**Range**: 0-65535 (steps/second)
**Description**: Speed threshold for switching to SpreadCycle mode
**Usage**: 0 = always StealthChop, high value = always SpreadCycle

#### ChopConf Settings (Extended Settings)
When `TRINAMIC_EXTENDED_SETTINGS` is enabled:

**$516**: TOFF - Off time setting
**$517**: TBL - Blanking time  
**$518**: CHM - Chopper mode
**$519**: HSTRT - Hysteresis start
**$520**: HEND - Hysteresis end
**$521**: HDEC - Hysteresis decrement
**$522**: RNDTF - Random TOFF

**Usage**: Advanced motor tuning parameters
**Caution**: Incorrect values can damage drivers or motors

#### CoolConf Settings (Extended Settings)
**$523**: SEMIN - Minimum StallGuard value
**$524**: SEUP - Current increment steps
**$525**: SEMAX - Maximum StallGuard value  
**$526**: SEDN - Current decrement steps
**$527**: SEIMIN - Minimum current flag

**Description**: Automatic current adjustment based on load
**Usage**: Optimizes current consumption and motor heating

---

## SD Card Plugin Settings

The SD card plugin enables G-code file storage and execution from SD cards.

 to restore defaults
2. **Setting Verification**: Use `$` to verify all settings
3. **Range Checking**: Check setting ranges if values rejected
4. **Plugin Dependencies**: Verify plugin requirements are met

---

## Conclusion

This comprehensive reference covers all grblHAL configuration settings, from core motion control to advanced plugin functionality. The modular architecture allows extensive customization while maintaining compatibility and reliability.

### Key Takeaways

- **Systematic Approach**: Configure settings in logical order (basic motion first, then advanced features)
- **Documentation**: Keep records of working configurations for future reference
- **Testing**: Thoroughly test each subsystem before moving to the next
- **Safety First**: Always prioritize safety features and emergency stops

### Resources

- **Live Help**: Use `$=<number>` to see allowed values for any setting
- **Plugin Documentation**: Check individual plugin repositories for detailed information
- **Community Support**: Active community forums for troubleshooting and optimization
- **Regular Updates**: Monitor grblHAL releases for new features and settings

For the most current information, always refer to the official grblHAL repositories and documentation at https://github.com/grblHAL/

---
