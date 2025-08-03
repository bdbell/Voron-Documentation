---
layout: default
title: grblHAL Status Report Reference Guide
parent: grblHAL Guide
nav_order: 6
has_children: false
---

# grblHAL Status Report Reference Guide

## Overview

This reference guide details the grblHAL status report format for experienced users. The status report provides real-time information about the controller's state and is critical for monitoring CNC operations. Status reports are requested using the `?` command or automatically generated based on configuration.

## Status Report Format

```
[<status>{:<substatus>}|<position>|{<optional elements>}]
```

### Basic Structure

- **Brackets**: `[ ]` - Enclose the entire status report
- **Pipe separators**: `|` - Separate report elements
- **Optional elements**: `{ }` - May or may not be present based on configuration
- **Variables**: `< >` - Placeholder for actual values

## Status Fields

### Primary Status

The first field indicates the machine's current operational state:

| Status | Description | Behavior |
|--------|-------------|----------|
| `Idle` | Ready for commands | Machine is stationary and ready to accept new G-code |
| `Run` | Executing G-code | Machine is actively running a program |
| `Hold` | Feed hold active | Motion paused but spindle may continue running |
| `Jog` | Jogging motion | Manual jogging commands being executed |
| `Alarm` | Alarm state | Controller locked due to error condition |
| `Door` | Safety door open | Safety door interlock triggered |
| `Check` | G-code check mode | Parsing G-code without motion |
| `Home` | Homing sequence | Executing homing cycle |
| `Sleep` | Sleep mode | Low-power state |
| `Tool` | Tool change | Manual or automatic tool change in progress |

### Substatus Indicators

Optional substatus provides additional context (format: `Status:Substatus`):

#### Run Substatus
- `Run:1` - Feed hold pending (delayed for spindle sync operations)
- `Run:2` - Probe motion active

#### Alarm Substatus
- `Alarm:<code>` - Specific alarm code (enabled by configuration)

### Position Reporting

Position data appears in the second field:

#### Position Types
- `MPos:` - Machine position (absolute coordinates from machine home)
- `WPos:` - Work position (coordinates relative to current work coordinate system)

#### Format
```
<MPos:|WPos:><X>,<Y>,<Z>{,<A>,<B>,<C>,<U>,<V>,<W>}
```

**Example**: `MPos:12.500,-5.200,0.000`

## Optional Report Elements

### Buffer Status (`Bf`)
```
|Bf:<block buffers free>,<RX characters free>
```
Reports available planner block buffers and serial receive buffer space.

**Example**: `|Bf:14,128`

### Line Number (`Ln`)
```
|Ln:<line number>
```
Current line number being executed from G-code program.

**Example**: `|Ln:1542`

### Feed and Speed (`FS`)
```
|FS:<feed rate>,<programmed rpm>{,<actual rpm>}
```
- Current feed rate in units/min
- Programmed spindle RPM
- Actual RPM (if encoder feedback available)

**Example**: `|FS:500,12000,11980`

### Input Pins (`Pn`)
```
|Pn:<signals>
```

Active input signals using single-letter codes:

| Letter | Signal | Description |
|--------|--------|-------------|
| `P` | Probe | Probe input triggered |
| `O` | Probe disconnected | Probe connectivity fault |
| `X`,`Y`,`Z` | Limit switches | Axis limit switches |
| `A`,`B`,`C`,`U`,`V`,`W` | Additional limits | Extended axis limits |
| `D` | Door | Safety door switch |
| `R` | Reset | Reset switch |
| `H` | Hold | Feed hold switch |
| `S` | Start | Cycle start switch |
| `E` | E-Stop | Emergency stop |
| `L` | Block delete | Block delete switch |
| `T` | Optional stop | Program stop switch |
| `M` | Motor warning | Motor fault warning |
| `F` | Motor fault | Motor fault condition |
| `Q` | Single step | Single block mode |

**Example**: `|Pn:XYZ` (all three axis limits triggered)

### Work Coordinate Offset (`WCO`)
```
|WCO:<axis offsets>
```
Offsets between machine and work coordinates for all axes.

**Example**: `|WCO:0.000,0.000,0.000`

### Work Coordinate System (`WCS`)
```
|WCS:G<coordinate system>
```
Active coordinate system (G54-G59.3).

**Example**: `|WCS:G54`

### Override Values (`Ov`)
```
|Ov:<feed override>,<rapid override>,<spindle override>
```
Current override percentages (100% = normal speed).

**Example**: `|Ov:100,100,100`

### Accessory Status (`A`)
```
|A:<accessory status>
```

Active accessories using single-letter codes:

| Letter | Accessory | Description |
|--------|-----------|-------------|
| `S` | Spindle CW | Spindle running clockwise (M3) |
| `C` | Spindle CCW | Spindle running counterclockwise (M4) |
| `M` | Mist coolant | Mist coolant active (M7) |
| `F` | Flood coolant | Flood coolant active (M8) |
| `T` | Tool change | Tool change pending (M6) |

**Example**: `|A:SFM` (spindle CW, flood coolant, mist coolant)

## grblHAL Extensions

### MPG Control (`MPG`)
```
|MPG:<0|1>
```
Manual Pulse Generator (pendant) control status:
- `0` - Pendant released control, normal operation
- `1` - Pendant has control, sender UI should disable

### SD Card Streaming (`SD`)
```
|SD:<pct complete>{,<filename>}
```
SD card streaming progress and optional filename.

**Example**: `|SD:45,program.nc`

### Homing Status (`H`)
```
|H:<0|1>{,<axis bitmask>}
```
- First value: Overall homing status (0=incomplete, 1=complete)
- Optional bitmask: Individual axis homing status (bit 0=X, bit 1=Y, etc.)

**Example**: `|H:1,7` (homing complete, XYZ axes homed)

### Diameter Mode (`D`)
```
|D:<0|1>
```
Lathe diameter/radius mode (0=radius/G8, 1=diameter/G7).

### Scaling Status (`Sc`)
```
|Sc:<axisletters>
```
Reports which axes have scaling active.

**Example**: `|Sc:XY`

### Tool Length Reference (`TLR`)
```
|TLR:<0|1>
```
Tool length reference offset status (0=not set, 1=set).

### Firmware Identification (`FW`)
```
|FW:<firmware>
```
Always reports `|FW:grblHAL` when compatibility level < 2.

### Input Status (`In`)
```
|In:<r>
```
Last M66 (Wait for input) command result:
- `-1` - Command failed
- `0` - Digital input low
- `1` - Digital input high

## Request Methods

### Standard Status Request
Send `?` character to request current status report.

### Full Status Request
Send `0x87` (ASCII 135) to request complete status with all available elements.

## Configuration Dependencies

Status report elements are controlled by settings:

- `$10` - Status report options bitfield
- `$13` - Report inches (affects position units)
- Individual element enable/disable flags in configuration

## Example Status Reports

### Basic Idle State
```
<Idle|MPos:0.000,0.000,0.000|FS:0,0>
```

### Running with Full Status
```
<Run|MPos:10.250,-5.500,2.100|Bf:14,128|Ln:542|FS:1500,12000,11995|WCO:0.000,0.000,0.000|Ov:100,100,100|A:SF>
```

### Alarm with Subcode
```
<Alarm:1|MPos:0.000,0.000,0.000|Pn:XYZ>
```

### Tool Change in Progress
```
<Tool|MPos:50.000,25.000,10.000|A:T|MPG:0>
```

## Integration Notes

- Senders must handle variable report length due to optional elements
- Report frequency depends on configuration and state changes
- Some elements only appear on state changes to minimize bandwidth
- Extensions from plugins/drivers may add additional elements
- All responses are terminated with `ok` on a new line

## Performance Considerations

- Status reports are generated in real-time with minimal controller overhead
- Optional elements reduce bandwidth when disabled
- Full status request (`0x87`) should be used sparingly
- Report parsing should be tolerant of unknown elements for future compatibility

---
