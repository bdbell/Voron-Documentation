---
layout: default
title: grblHAL G-Code and M-Code Reference Guide
parent: grblHAL Guide
nav_order: 8
has_children: false
---

# grblHAL G-Code and M-Code Reference Guide

## Quick Reference Table

### G-Codes
| Code | Purpose | Code | Purpose |
|------|---------|------|---------|
| G0 | Rapid positioning | G1 | Linear interpolation |
| G2 | Clockwise arc | G3 | Counterclockwise arc |
| G4 | Dwell/pause | G5 | Cubic spline |
| G5.1 | Quadratic spline | G7* | Lathe diameter mode |
| G8* | Lathe radius mode | G10 L1* | Set tool table |
| G10 L2 | Set coordinate system | G10 L10* | Set tool table |
| G10 L11* | Set tool table | G10 L20 | Set coordinate system |
| G17 | XY plane select | G18 | XZ plane select |
| G19 | YZ plane select | G20 | Inch units |
| G21 | Millimeter units | G28 | Return to home |
| G28.1 | Set home position | G30 | Return to position |
| G30.1 | Set position | G33* | Spindle sync motion |
| G38.2 | Probe toward | G38.3 | Probe toward (no error) |
| G38.4 | Probe away | G38.5 | Probe away (no error) |
| G40 | Cancel cutter compensation | G43* | Tool length offset |
| G43.1 | Dynamic tool offset | G43.2* | Additional tool offset |
| G49 | Cancel tool offset | G50 | Reset scaling |
| G51 | Scale axes | G53 | Machine coordinates |
| G54-G59 | Work coordinate systems | G59.1-G59.3 | Extended work coordinates |
| G61 | Exact path mode | G65***** | Call macro |
| G73 | High-speed peck drill | G76* | Threading cycle |
| G80 | Cancel canned cycles | G81 | Simple drilling |
| G82 | Drilling with dwell | G83 | Peck drilling |
| G85 | Boring cycle | G86 | Boring with stop |
| G89 | Boring with dwell | G90 | Absolute positioning |
| G91 | Incremental positioning | G91.1 | Arc IJK incremental |
| G92 | Set coordinate offset | G92.1 | Cancel offset |
| G93 | Inverse time feed | G94 | Feed per minute |
| G95* | Feed per revolution | G96* | Constant surface speed |
| G97* | Constant RPM | G98 | Initial point return |
| G99 | R-point return | | |

### M-Codes
| Code | Purpose | Code | Purpose |
|------|---------|------|---------|
| M0 | Program stop | M1 | Optional stop |
| M2 | Program end | M3 | Spindle CW |
| M4 | Spindle CCW | M5 | Spindle stop |
| M6* | Tool change | M7 | Mist coolant on |
| M8 | Flood coolant on | M9 | Coolant off |
| M30 | Program end/rewind | M48 | Enable overrides |
| M49 | Disable overrides | M50 | Feed override |
| M51 | Spindle override | M53 | Feed hold |
| M60 | Pallet change pause | M61 | Set tool number |
| M62*** | Digital output sync | M63*** | Digital output sync |
| M64*** | Digital output immediate | M65*** | Digital output immediate |
| M66*** | Wait on input | M67*** | Analog output sync |
| M68*** | Analog output immediate | M70-M73 | Save/restore state |
| M99 | Return from macro | | |

*Requires specific driver support or lathe mode  
**Manual tool change or ATC support required  
***Requires driver with I/O support  
****Supports multi-turn arcs  
*****Requires macros/SD card plugin

---

## Detailed Command Reference

### Non-Modal Commands

#### G4 - Dwell
**Purpose:** The machine will pause all motion for a specified time duration while maintaining position and keeping the spindle and coolant active if they were running.

**Syntax:** `G4 P[seconds]`

**Arguments:**
- `P` - Dwell time in seconds (can be decimal)

**Example:** `G4 P2.5` (pause for 2.5 seconds)

**Potential Errors:**
- Missing P value will cause a parameter error
- Negative P values are invalid

---

#### G10 L1 - Set Tool Table (requires tool table support)
**Purpose:** The machine will update the tool table entry with new tool length offset and radius values for the specified tool number.

**Syntax:** `G10 L1 P[tool] Z[offset] R[radius]`

**Arguments:**
- `P` - Tool number to modify
- `Z` - Tool length offset value
- `R` - Tool radius (optional, may not be implemented)

**Example:** `G10 L1 P1 Z-2.4` (set tool 1 length offset to -2.4)

**Potential Errors:**
- Invalid tool number (outside defined range)
- Tool table not available

---

#### G10 L2/L20 - Set Coordinate System Origin
**Purpose:** The machine will set the origin offset for the specified coordinate system (G54-G59.3) to position the work coordinate system relative to machine coordinates.

**Syntax:** `G10 L2 P[coord_sys] X[x_offset] Y[y_offset] Z[z_offset]...`  
**Syntax:** `G10 L20 P[coord_sys] X[x_pos] Y[y_pos] Z[z_pos]...`

**Arguments:**
- `P` - Coordinate system number (1-9 for G54-G59.3)
- L2: Offset values are absolute machine coordinates
- L20: Current position becomes the specified coordinate values
- Axis values (X,Y,Z,A,B,C,U,V,W) as applicable

**Example:** `G10 L2 P1 X10 Y10 Z5` (set G54 origin)

**Potential Errors:**
- Invalid coordinate system number
- Axis not available on machine

---

#### G28/G30 - Return to Reference Position
**Purpose:** The machine will move all specified axes to their predefined reference positions, optionally passing through an intermediate point for collision avoidance.

**Syntax:** `G28 [X] [Y] [Z]...` or `G30 [X] [Y] [Z]...`

**Arguments:**
- Axis letters without values: move those axes to reference
- Axis letters with values: move through intermediate point first
- No arguments: move all axes to reference

**Example:** `G28 Z0` (move Z to 0 first, then all axes to G28 position)

**Potential Errors:**
- Reference position not set (use G28.1/G30.1 first)
- Intermediate position causes limit switch activation

---

#### G28.1/G30.1 - Set Reference Position
**Purpose:** The machine will store the current position as the reference position for G28 or G30 commands respectively.

**Syntax:** `G28.1` or `G30.1`

**Example:** `G28.1` (set current position as G28 reference)

---

#### G53 - Machine Coordinate System
**Purpose:** The machine will move in absolute machine coordinates for this block only, ignoring all work coordinate offsets and tool length offsets.

**Syntax:** `G53 G0/G1 X[pos] Y[pos] Z[pos]...`

**Arguments:**
- Must be used with G0 or G1
- Positions are in machine coordinates
- Non-modal (only affects current line)

**Example:** `G53 G0 Z0` (rapid to machine Z zero)

**Potential Errors:**
- Used without motion command
- Position beyond machine limits

---

#### G92 - Coordinate System Offset
**Purpose:** The machine will apply a temporary offset to the current coordinate system, shifting the origin without changing work coordinate system settings.

**Syntax:** `G92 X[value] Y[value] Z[value]...`

**Arguments:**
- Axis values define what the current position should be called
- Creates temporary offset from current work coordinate system

**Example:** `G92 X0 Y0` (make current XY position the new origin)

**Potential Errors:**
- Can cause confusion with work coordinate systems
- Use G92.1 to cancel

---

#### G92.1 - Cancel G92 Offset
**Purpose:** The machine will remove any active G92 coordinate system offset, returning to the underlying work coordinate system.

**Syntax:** `G92.1`

---

### Motion Modes

#### G0 - Rapid Positioning
**Purpose:** The machine will move all specified axes simultaneously to the commanded position at maximum rapid traverse speed in a coordinated straight line motion.

**Syntax:** `G0 X[pos] Y[pos] Z[pos]...`

**Arguments:**
- Axis positions in current coordinate system and units
- All axes move simultaneously to complete motion together
- Speed determined by machine maximum rapid rate settings

**Example:** `G0 X10 Y5 Z2` (rapid move to position)

**Potential Errors:**
- Position beyond soft limits
- Would activate limit switches
- Collision with workpiece or fixtures

---

#### G1 - Linear Interpolation
**Purpose:** The machine will move all specified axes simultaneously to the commanded position at the programmed feed rate in a coordinated straight line motion.

**Syntax:** `G1 X[pos] Y[pos] Z[pos]... F[feedrate]`

**Arguments:**
- Axis positions in current coordinate system and units
- `F` - Feed rate in current units per minute (modal)
- Coordinated motion maintains programmed feed rate

**Example:** `G1 X20 Y10 F100` (linear move at 100 units/min)

**Potential Errors:**
- No feed rate specified and none currently active
- Feed rate exceeds machine maximum
- Position beyond limits

---

#### G2/G3 - Circular Interpolation
**Purpose:** The machine will move in a circular arc from the current position to the specified endpoint, with G2 creating clockwise motion and G3 creating counterclockwise motion when viewed from the positive direction of the current plane's perpendicular axis.

**Syntax:** `G2/G3 X[end_x] Y[end_y] I[center_x] J[center_y] F[feedrate]`  
**Syntax:** `G2/G3 X[end_x] Y[end_y] R[radius] F[feedrate]`

**Arguments:**
- Endpoint coordinates (X,Y for G17 plane)
- Arc center specified by either:
  - `I,J,K` - Incremental distance from start point to arc center
  - `R` - Arc radius (positive for minor arc, negative for major arc)
- Multi-turn arcs supported with additional P parameter

**Example:** `G2 X10 Y10 I5 J0 F100` (clockwise arc)

**Potential Errors:**
- Arc center calculation results in invalid geometry
- Start and end points different distances from center
- Arc would exceed machine limits

---

#### G5 - Cubic Spline
**Purpose:** The machine will create a smooth cubic spline curve through the specified control points, providing smooth motion for complex curved paths.

**Syntax:** `G5 X[pos] Y[pos] I[x1] J[y1] P[x2] Q[y2]`

**Arguments:**
- X,Y - Endpoint coordinates
- I,J - First control point
- P,Q - Second control point

**Potential Errors:**
- Invalid control point geometry
- Curve exceeds machine envelope

---

#### G5.1 - Quadratic Spline
**Purpose:** The machine will create a smooth quadratic spline curve to the specified endpoint using the given control point.

**Syntax:** `G5.1 X[pos] Y[pos] I[x1] J[y1]`

**Arguments:**
- X,Y - Endpoint coordinates
- I,J - Control point

---

#### G33 - Spindle Synchronized Motion (requires spindle encoder)
**Purpose:** The machine will move the Z-axis in perfect synchronization with spindle rotation to cut threads, maintaining precise pitch regardless of spindle speed variations.

**Syntax:** `G33 Z[end_pos] K[pitch]`

**Arguments:**
- Z - Ending Z position
- K - Thread pitch (distance per revolution)
- Requires spindle encoder for position feedback

**Example:** `G33 Z-20 K1.5` (cut 1.5mm pitch thread)

**Potential Errors:**
- No spindle encoder signal
- Spindle not running
- Insufficient Z-axis travel

---

#### G38.2/G38.3/G38.4/G38.5 - Probing
**Purpose:** The machine will move toward (G38.2/G38.3) or away from (G38.4/G38.5) the specified position until the probe input changes state, storing the contact position in parameters for touch-off and measurement operations.

**Syntax:** `G38.x X[pos] Y[pos] Z[pos] F[feedrate]`

**Arguments:**
- Endpoint coordinates (maximum probe distance)
- F - Probing feed rate
- .2/.4 - Error if probe doesn't trigger
- .3/.5 - No error if probe doesn't trigger

**Example:** `G38.2 Z-10 F50` (probe down until contact)

**Potential Errors:**
- No probe input configured
- Probe already triggered at start (G38.2/G38.3)
- Probe not triggered at start (G38.4/G38.5)
- Position would exceed limits

---

#### G80 - Cancel Canned Cycles
**Purpose:** The machine will cancel any active canned cycle mode and return to normal motion mode interpretation.

**Syntax:** `G80`

---

### Canned Cycles

#### G73 - High-Speed Peck Drilling
**Purpose:** The machine will drill a hole using a pecking motion with partial retraction between pecks, designed for chip breaking without full retraction for faster cycle times.

**Syntax:** `G73 X[pos] Y[pos] Z[depth] R[retract] Q[peck] F[feedrate]`

**Arguments:**
- X,Y - Hole position coordinates
- Z - Final hole depth (absolute coordinate)
- R - Retract plane (rapid approach level)
- Q - Peck increment depth
- F - Drilling feed rate

**Example:** `G73 X10 Y10 Z-20 R2 Q5 F100`

**Potential Errors:**
- Z depth below R retract plane
- Insufficient spindle speed
- Drill breaks due to insufficient chip clearing

---

#### G81 - Simple Drilling Cycle
**Purpose:** The machine will drill a straight hole by moving to the specified XY position, rapidly approaching the R plane, feeding to the Z depth, then retracting at rapid speed.

**Syntax:** `G81 X[pos] Y[pos] Z[depth] R[retract] F[feedrate]`

**Arguments:**
- X,Y - Hole position coordinates
- Z - Final hole depth (absolute coordinate) 
- R - Retract plane (rapid approach level)
- F - Drilling feed rate

**Example:** `G81 X10 Y10 Z-5 R1 F50`

**Potential Errors:**
- Z depth not below current position
- Feed rate too high causing drill breakage
- Inadequate chip evacuation for deep holes

---

#### G82 - Drilling Cycle with Dwell
**Purpose:** The machine will drill a hole similar to G81 but includes a dwell time at the bottom of the hole to improve surface finish and ensure complete cutting.

**Syntax:** `G82 X[pos] Y[pos] Z[depth] R[retract] P[dwell] F[feedrate]`

**Arguments:**
- X,Y - Hole position coordinates
- Z - Final hole depth (absolute coordinate)
- R - Retract plane (rapid approach level)
- P - Dwell time at bottom (seconds)
- F - Drilling feed rate

**Example:** `G82 X10 Y10 Z-8 R2 P1.0 F100`

**Potential Errors:**
- Excessive dwell time causing tool burning
- Missing dwell parameter

---

#### G83 - Peck Drilling Cycle
**Purpose:** The machine will drill deep holes using a full-retraction pecking cycle that completely withdraws the drill between each peck to clear chips and break stringy material.

**Syntax:** `G83 X[pos] Y[pos] Z[depth] R[retract] Q[peck] F[feedrate]`

**Arguments:**
- X,Y - Hole position coordinates
- Z - Final hole depth (absolute coordinate)
- R - Retract plane (rapid approach level)
- Q - Peck increment depth per cycle
- F - Drilling feed rate

**Example:** `G83 X10 Y10 Z-25 R2 Q3 F80`

**Potential Errors:**
- Peck depth too large causing excessive loads
- Rapid retraction hitting clamps
- Drill wandering on deep holes

---

#### G85 - Boring Cycle
**Purpose:** The machine will bore a precise hole by feeding down at the programmed rate and retracting at the same feed rate to prevent tool marks on the finished bore surface.

**Syntax:** `G85 X[pos] Y[pos] Z[depth] R[retract] F[feedrate]`

**Arguments:**
- X,Y - Hole position coordinates
- Z - Final hole depth (absolute coordinate)
- R - Retract plane
- F - Boring feed rate (used for both feed in and retract)

**Example:** `G85 X15 Y15 Z-12 R3 F40`

**Potential Errors:**
- Feed rate too fast causing poor surface finish
- Insufficient rigidity causing chatter

---

#### G86 - Boring Cycle with Spindle Stop
**Purpose:** The machine will bore a hole, stop the spindle at the bottom to prevent tool marks, retract at rapid rate, then restart the spindle for the next operation.

**Syntax:** `G86 X[pos] Y[pos] Z[depth] R[retract] F[feedrate]`

**Arguments:**
- X,Y - Hole position coordinates
- Z - Final hole depth (absolute coordinate)  
- R - Retract plane
- F - Boring feed rate

**Example:** `G86 X20 Y20 Z-8 R2 F60`

**Potential Errors:**
- Spindle restart delay affects cycle time
- Tool deflection during retract

---

#### G89 - Boring Cycle with Dwell
**Purpose:** The machine will bore a hole, dwell at the bottom with the spindle running to improve surface finish, then retract at feed rate.

**Syntax:** `G89 X[pos] Y[pos] Z[depth] R[retract] P[dwell] F[feedrate]`

**Arguments:**
- X,Y - Hole position coordinates
- Z - Final hole depth (absolute coordinate)
- R - Retract plane
- P - Dwell time at bottom (seconds)
- F - Boring feed rate

**Example:** `G89 X25 Y25 Z-10 R2 P0.5 F50`

**Potential Errors:**
- Excessive dwell causing work hardening
- Heat buildup affecting tool life

---

#### G98/G99 - Canned Cycle Return Level
**Purpose:** G98 sets canned cycles to return to the initial Z level before the cycle, while G99 returns only to the R retract plane for more efficient operation.

**Syntax:** `G98` or `G99`

**Arguments:** None - modal setting for subsequent canned cycles

---

### Feed Rate Modes

#### G93 - Inverse Time Feed Mode
**Purpose:** The machine will interpret the F word as the inverse of the time in minutes that the move should take, allowing precise timing control for complex coordinated motions.

**Syntax:** `G93`

**Arguments:** F value represents 1/time (where time is in minutes)

**Example:** `G93 G1 X10 F2.0` (move takes 0.5 minutes)

**Potential Errors:**
- F value of zero (infinite time)
- Extremely high F values causing impossible motion

---

#### G94 - Units per Minute Feed Mode
**Purpose:** The machine will interpret the F word as linear distance per minute in the current units (mm/min or inches/min), which is the standard feed rate mode.

**Syntax:** `G94`

**Arguments:** F value in current units per minute

---

#### G95 - Units per Revolution Feed Mode (requires spindle encoder)
**Purpose:** The machine will interpret the F word as linear distance per spindle revolution, automatically adjusting the feed rate based on current spindle speed.

**Syntax:** `G95`

**Arguments:** F value in current units per revolution

**Potential Errors:**
- No spindle encoder feedback
- Spindle not running
- Division by zero at spindle stop

---

### Unit Modes

#### G20 - Inch Programming
**Purpose:** The machine will interpret all coordinate and distance values as inches and switch internal calculations to inch-based units.

**Syntax:** `G20`

**Note:** Affects coordinates, feed rates, and offsets but not spindle speed

---

#### G21 - Millimeter Programming  
**Purpose:** The machine will interpret all coordinate and distance values as millimeters and switch internal calculations to metric units.

**Syntax:** `G21`

**Note:** Default unit mode for most grblHAL configurations

---

### Distance Modes

#### G90 - Absolute Distance Mode
**Purpose:** The machine will interpret all coordinate values as absolute positions relative to the current work coordinate system origin.

**Syntax:** `G90`

**Example:** `G90 G1 X10` always moves to position X=10

---

#### G91 - Incremental Distance Mode
**Purpose:** The machine will interpret all coordinate values as relative distances from the current position.

**Syntax:** `G91`

**Example:** `G91 G1 X10` moves 10 units in positive X direction

**Potential Errors:**
- Accumulated positioning errors over multiple moves
- Unexpected direction due to mode confusion

---

### Plane Selection

#### G17 - XY Plane Selection
**Purpose:** The machine will perform circular interpolation (G2/G3) in the XY plane with Z as the perpendicular axis, which is the standard configuration for most milling operations.

**Syntax:** `G17`

**Default plane for most operations**

---

#### G18 - XZ Plane Selection
**Purpose:** The machine will perform circular interpolation in the XZ plane with Y as the perpendicular axis, commonly used for lathe operations or vertical milling.

**Syntax:** `G18`

---

#### G19 - YZ Plane Selection
**Purpose:** The machine will perform circular interpolation in the YZ plane with X as the perpendicular axis, used for specialized operations.

**Syntax:** `G19`

---

### Coordinate Systems

#### G54-G59, G59.1-G59.3 - Work Coordinate Systems
**Purpose:** The machine will switch to the specified work coordinate system, which defines the relationship between part coordinates and machine coordinates through stored offset values.

**Syntax:** `G54` through `G59.3`

**Arguments:** None - selects the coordinate system

**Available Systems:**
- G54: Work coordinate system 1 (default)
- G55-G59: Work coordinate systems 2-6  
- G59.1-G59.3: Extended work coordinate systems 7-9

**Example:** `G55 G0 X0 Y0` (move to origin of coordinate system 2)

---

### Tool Length Offset

#### G43 - Tool Length Offset (requires tool table)
**Purpose:** The machine will apply the tool length offset from the tool table for the currently selected tool, adjusting Z-axis positions to compensate for different tool lengths.

**Syntax:** `G43 H[tool_number]`

**Arguments:**
- H - Tool number whose offset to apply (optional, defaults to current tool)

**Example:** `G43 H1` (apply tool 1 length offset)

**Potential Errors:**
- Tool number not in tool table
- Tool table not available

---

#### G43.1 - Dynamic Tool Length Offset
**Purpose:** The machine will apply an immediate tool length offset value without requiring a tool table entry.

**Syntax:** `G43.1 Z[offset_value]`

**Arguments:**
- Z - Immediate offset value to apply

**Example:** `G43.1 Z-2.540` (apply immediate offset)

---

#### G49 - Cancel Tool Length Offset
**Purpose:** The machine will cancel any active tool length offset and return to programming Z coordinates as actual machine positions.

**Syntax:** `G49`

---

### Scaling

#### G50 - Reset Scaling
**Purpose:** The machine will cancel any active coordinate scaling and return to 1:1 scaling factors for all axes.

**Syntax:** `G50`

---

#### G51 - Scale Coordinate System
**Purpose:** The machine will apply scaling factors to subsequent coordinate moves, allowing proportional resizing of programmed geometry.

**Syntax:** `G51 X[scale_x] Y[scale_y] Z[scale_z]...`

**Arguments:**
- Axis letters with scaling factors (1.0 = no scaling)
- Values less than 1.0 reduce size
- Values greater than 1.0 increase size

**Example:** `G51 X0.5 Y0.5` (scale XY to 50%)

**Potential Errors:**
- Zero scaling factors
- Negative scaling factors
- Scaled moves exceed machine limits

---

## M-Code Reference

### Program Flow

#### M0 - Program Stop
**Purpose:** The machine will pause program execution and wait for the operator to press cycle start before continuing with the next programmed line.

**Syntax:** `M0`

**Machine remains in automatic mode with spindle and coolant unchanged**

---

#### M1 - Optional Stop
**Purpose:** The machine will pause program execution only if the optional stop switch is enabled, otherwise the command is ignored and execution continues.

**Syntax:** `M1`

---

#### M2 - Program End
**Purpose:** The machine will end the current program, reset to G94, G17, G90, G54 default modes, turn off spindle and coolant, and prepare for a new program to be started.

**Syntax:** `M2`

**Effects:**
- Spindle stops (M5)
- Coolant turns off (M9)  
- Modal groups reset to defaults
- Program counter resets to beginning

---

#### M30 - Program End and Rewind
**Purpose:** The machine will end the current program with the same effects as M2 and automatically rewind to the beginning of the program.

**Syntax:** `M30`

---

#### M60 - Pallet Change Pause
**Purpose:** The machine will pause for a pallet change operation, similar to M0 but specifically intended for automatic pallet changer systems.

**Syntax:** `M60`

---

### Spindle Control

#### M3 - Spindle On Clockwise
**Purpose:** The machine will start the spindle rotating in the clockwise direction at the currently programmed speed (S value).

**Syntax:** `M3 [S[speed]]`

**Arguments:**
- S - Spindle speed (optional, uses last programmed value if omitted)

**Example:** `M3 S1200` (start spindle at 1200 RPM clockwise)

**Potential Errors:**
- No spindle speed programmed
- Speed exceeds maximum spindle capability
- Spindle drive fault

---

#### M4 - Spindle On Counterclockwise
**Purpose:** The machine will start the spindle rotating in the counterclockwise direction at the currently programmed speed.

**Syntax:** `M4 [S[speed]]`

**Arguments:**
- S - Spindle speed (optional)

**Example:** `M4 S800` (start spindle at 800 RPM counterclockwise)

---

#### M5 - Spindle Stop
**Purpose:** The machine will stop the spindle rotation and disable the spindle drive output.

**Syntax:** `M5`

**Spindle brake may be applied if configured**

---

### Tool Management

#### M6 - Tool Change (requires tool changer support)
**Purpose:** The machine will execute a tool change sequence, stopping the spindle and either prompting for manual tool change or executing an automatic tool changer sequence.

**Syntax:** `M6 [T[tool_number]]`

**Arguments:**
- T - Tool number to load (optional, uses last selected tool)

**Example:** `M6 T5` (change to tool 5)

**Manual Mode Effects:**
- Spindle stops
- Program pauses with prompt
- Operator changes tool
- Cycle start continues

**ATC Mode Effects:**
- Spindle stops and orients
- Current tool returned to magazine
- New tool retrieved from magazine
- Tool length offset may be measured

**Potential Errors:**
- Tool not found in magazine
- Tool changer mechanical fault
- Spindle orientation timeout

---

#### M61 - Set Current Tool Number
**Purpose:** The machine will update the current tool number register without performing a physical tool change, used for synchronizing software with actual tool in spindle.

**Syntax:** `M61 Q[tool_number]`

**Arguments:**
- Q - Tool number to set as current

**Example:** `M61 Q3` (tell system tool 3 is in spindle)

---

### Coolant Control

#### M7 - Mist Coolant On
**Purpose:** The machine will turn on the mist coolant output to provide light lubrication and cooling during machining operations.

**Syntax:** `M7`

**Potential Errors:**
- Coolant system not connected
- Low coolant level
- Coolant pump fault

---

#### M8 - Flood Coolant On  
**Purpose:** The machine will turn on the flood coolant output to provide heavy cooling and chip flushing during machining operations.

**Syntax:** `M8`

---

#### M9 - Coolant Off
**Purpose:** The machine will turn off all coolant outputs (both mist and flood).

**Syntax:** `M9`

---

### Override Controls

#### M48 - Enable Feed and Speed Override
**Purpose:** The machine will enable the feed rate and spindle speed override controls, allowing real-time adjustment during program execution.

**Syntax:** `M48`

---

#### M49 - Disable Feed and Speed Override
**Purpose:** The machine will disable override controls and force operation at exactly programmed feed rates and spindle speeds.

**Syntax:** `M49`

---

#### M50 - Feed Override Control
**Purpose:** The machine will enable or disable feed rate override capability while leaving spindle override unchanged.

**Syntax:** `M50 P[0/1]`

**Arguments:**
- P0 - Disable feed override
- P1 - Enable feed override

---

#### M51 - Spindle Speed Override Control
**Purpose:** The machine will enable or disable spindle speed override capability while leaving feed override unchanged.

**Syntax:** `M51 P[0/1]`

**Arguments:**
- P0 - Disable spindle override  
- P1 - Enable spindle override

---

#### M53 - Feed Hold Control
**Purpose:** The machine will enable or disable the feed hold function that allows pausing motion while maintaining spindle operation.

**Syntax:** `M53 P[0/1]`

**Arguments:**
- P0 - Disable feed hold
- P1 - Enable feed hold

---

### Digital I/O Control (requires I/O support)

#### M62/M63 - Digital Output Control (Synchronized)
**Purpose:** The machine will set digital outputs high (M62) or low (M63) synchronized with motion, with the output change occurring at the start of the next motion command.

**Syntax:** `M62 P[output_number]` or `M63 P[output_number]`

**Arguments:**
- P - Digital output number (0-3, expandable with driver support)

**Example:** `M62 P0` followed by `G1 X10` (output 0 goes high when X motion starts)

**Potential Errors:**
- Output number not available
- No motion command following (output change won't occur)
- Driver doesn't support digital I/O

---

#### M64/M65 - Digital Output Control (Immediate)
**Purpose:** The machine will immediately set digital outputs high (M64) or low (M65) without waiting for motion synchronization.

**Syntax:** `M64 P[output_number]` or `M65 P[output_number]`

**Arguments:**
- P - Digital output number (0-3, expandable)

**Example:** `M64 P1` (immediately turn on output 1)

**Note:** These commands break motion blending and execute immediately

---

#### M66 - Wait on Input
**Purpose:** The machine will pause program execution and wait for a specified digital or analog input to reach the desired state before continuing.

**Syntax:** `M66 P[digital_input] L[mode] Q[timeout]`  
**Syntax:** `M66 E[analog_input] L[mode] Q[timeout]`

**Arguments:**
- P - Digital input number (0-3)
- E - Analog input number (0-3)  
- L - Wait mode:
  - L0: Immediate (no wait, just read current value)
  - L1: Wait for input to go high
  - L2: Wait for input to go low
  - L3: Wait for rising edge
  - L4: Wait for falling edge
- Q - Timeout in seconds (optional)

**Example:** `M66 P0 L1 Q5.0` (wait up to 5 seconds for input 0 to go high)

**Result:** Input value stored in parameter #5399

**Potential Errors:**
- Timeout exceeded
- Input number not available
- Both P and E specified

---

#### M67/M68 - Analog Output Control
**Purpose:** M67 sets analog output synchronized with motion (like M62/M63), while M68 sets analog output immediately.

**Syntax:** `M67 E[output_number] Q[value]` (synchronized)  
**Syntax:** `M68 E[output_number] Q[value]` (immediate)

**Arguments:**
- E - Analog output number (0-3)
- Q - Output value (range depends on driver implementation)

**Example:** `M68 E0 Q50.5` (immediately set analog output 0 to 50.5)

---

### Modal State Control (requires NGC expressions)

#### M70/M71/M72/M73 - Save and Restore Modal State
**Purpose:** These commands allow saving and restoring the complete modal state of the G-code interpreter for subroutine operations.

**M70:** Save current modal state  
**M71:** Invalidate saved modal state  
**M72:** Restore saved modal state  
**M73:** Save state for automatic restoration on subroutine return  

**Syntax:** `M70`, `M71`, `M72`, `M73`

**Saved State Includes:**
- Active G-codes (motion mode, coordinate system, etc.)
- Feed rate and spindle speed
- Coolant state
- Tool offsets
- Scaling factors

---

### Macro Control (requires macro support)

#### G65 - Call Macro
**Purpose:** The machine will call a stored macro program with the specified parameters, allowing reusable subroutines for common operations.

**Syntax:** `G65 P[macro_id] [A-Z parameter values]`

**Arguments:**
- P - Macro identifier (100-65535)
- Letter parameters passed as variables to macro
- Available on systems with SD card or macro plugin

**Example:** `G65 P100 X10 Y20 Z5` (call macro 100 with parameters)

**Parameter Mapping:**
- A=#1, B=#2, C=#3, D=#7, E=#8, F=#9
- H=#11, I=#4, J=#5, K=#6, M=#13
- Q=#17, R=#18, S=#19, T=#20
- U=#21, V=#22, W=#23, X=#24, Y=#25, Z=#26

**Potential Errors:**
- Macro file not found (P<id>.macro)
- Macro contains syntax errors
- Nesting not allowed
- SD card or macro plugin not available

---

#### M99 - Return from Macro
**Purpose:** The machine will return from the current macro to the calling program, optionally continuing from a specified line number.

**Syntax:** `M99 [P[line_number]]`

**Arguments:**
- P - Line number to return to (optional)

---

### Advanced Features

#### G76 - Threading Cycle (lathe mode)
**Purpose:** The machine will execute a complete threading cycle with multiple passes, automatically calculating cutting depths and tapers for thread cutting on lathes.

**Syntax:** `G76 P[thread_data] Z[end_pos] I[taper] J[initial_depth] K[full_depth] Q[compound_angle] H[passes] E[taper_length] L[cut_angle] R[depth_degression]`

**Arguments:**
- Complex parameter structure for thread geometry
- Requires lathe configuration and spindle encoder
- Multiple spring passes for thread finishing

**Potential Errors:**
- Invalid thread geometry
- No spindle encoder
- Insufficient Z-axis travel

---

#### G96/G97 - Constant Surface Speed Control (lathe mode)
**Purpose:** G96 enables constant surface speed mode where spindle RPM automatically adjusts based on current tool position to maintain constant cutting speed. G97 returns to constant RPM mode.

**Syntax:** `G96 S[surface_speed]` or `G97 S[rpm]`

**Arguments:**
- G96: S value in surface units per minute
- G97: S value in RPM

**Requires lathe mode and position feedback**

---

## Error Conditions and Troubleshooting

### Common Error Messages

**"Unsupported command"**
- Command not available in current build
- Check if required plugins are installed
- Verify driver capabilities

**"Bad number format"**
- Invalid numeric parameter
- Missing required parameter
- Parameter out of valid range

**"Expected command letter"**
- Missing G or M code
- Invalid command syntax
- Line parsing error

**"Homing fail"**
- Limit switches not triggered
- Incorrect homing direction
- Mechanical obstruction

**"Soft limit"**
- Motion would exceed configured limits
- Check work coordinate system offsets
- Verify part positioning

**"Hard limit"**
- Limit switch activated during motion
- Emergency stop condition
- Machine position lost

### Safety Considerations

1. **Always verify coordinate systems** before running programs
2. **Test programs in simulation** when possible
3. **Use appropriate feeds and speeds** for material and tooling
4. **Ensure proper work holding** for all operations
5. **Check tool lengths** and offsets before machining
6. **Verify limit switch operation** regularly
7. **Keep emergency stop accessible** during operation

### Programming Best Practices

1. **Use consistent coordinate systems** throughout programs
2. **Cancel canned cycles with G80** when finished
3. **Specify feed rates explicitly** for all cutting moves
4. **Use safe Z heights** for rapid positioning
5. **Turn off spindle and coolant** at program end
6. **Include tool change sequences** where required
7. **Comment complex operations** for clarity

---

## Parameter Reference

### System Parameters (Read-Only)

- `#5399` - Result of M66 wait on input command
- Tool offset values stored in tool table
- Work coordinate system offsets (G54-G59.3)
- Current position values
- Modal state information

### Variable Scope

- **Local variables** - Available within current subroutine only
- **Global variables** - Available across all subroutines and main program
- **System parameters** - Maintained by grblHAL system

---

## Driver-Specific Extensions

Some M-codes and features require specific driver support:

- **Lathe operations** (G7, G8, G33, G76, G96, G97) require lathe-mode drivers
- **I/O control** (M62-M68) requires drivers with digital/analog I/O
- **Tool management** (M6, G43) requires tool changer or manual tool change support
- **Probing** (G38.x) requires probe input configuration
- **Spindle synchronization** (G33, G95) requires spindle encoder feedback

Check your specific grblHAL driver documentation for available features and configuration requirements.

---

**Note:** This reference covers the core grblHAL command set. Additional commands may be available through plugins or driver-specific extensions. Commands marked with asterisks (*) require specific hardware support or configuration options.

---
