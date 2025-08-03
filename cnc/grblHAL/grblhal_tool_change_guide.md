---
layout: default
title: grblHAL Tool Length Measurement and Tool Change Guide
parent: grblHAL Guide
nav_order: 6
has_children: false
---

# grblHAL Tool Length Measurement and Tool Change Guide

## Introduction

This guide covers how to perform initial tool length measurement and tool change operations using grblHAL firmware, gSender software, and a hardware tool setter switch. Tool length measurement is essential for maintaining precision when switching between tools of different lengths during CNC operations.

## Prerequisites

- CNC machine running grblHAL firmware (build 20200805 or later)
- gSender software installed and connected to your machine
- Hardware tool setter switch (normally closed type recommended)
- Machine properly homed and calibrated

## Hardware Setup

### Tool Setter Connection

Connect your tool setter to the probe input on your grblHAL controller. Use a normally closed (NC) type tool setter for best reliability. Wire the tool setter as follows:

- **Ground**: Connect to controller ground
- **Signal**: Connect to probe input pin
- **Power** (if 3-wire): Use same voltage as controller (typically 5V)

### Tool Setter Position

Position your tool setter at a fixed, accessible location on your wasteboard or machine table. Record the exact X, Y, and Z coordinates of this position - you'll need these for configuration.

## grblHAL Configuration

### Tool Change Mode Settings

grblHAL supports several tool change modes controlled by setting $341:

- **$341=0**: Normal/Basic mode (manual)
- **$341=1**: Manual touch off mode
- **$341=2**: Manual touch off @ G59.3
- **$341=3**: Automatic touch off @ G59.3 (recommended for tool setters)
- **$341=4**: Ignore M6 commands

### Recommended Settings for Tool Setter

For automatic tool length measurement with a hardware tool setter, configure:

```
$341=3    ; Automatic touch off @ G59.3 mode
$342=30   ; Maximum probing distance (mm)
$343=25   ; Probing feed rate (mm/min)
$344=50   ; Probing seek rate (mm/min)
```

### Set Tool Setter Position (G59.3)

The G59.3 coordinate system must be set to the exact location of your tool setter. To set this:

1. Manually jog to your tool setter position
2. Use the command: `G10 L2 P9 X[X-pos] Y[Y-pos] Z[Z-pos]`
   - Replace [X-pos], [Y-pos], [Z-pos] with actual coordinates
3. Verify with `$#` command to see G59.3 offsets

## Initial Tool Length Reference Setup

### Establishing the Reference Tool

Before using automatic tool changes, you must establish an initial tool length reference using the $TLR command:

1. **Install your first/reference tool** in the spindle
2. **Move to tool setter position** manually or with `G0 G59.3`
3. **Probe the tool manually**:
   - In gSender: Use Probing tab → Tool Length Offset
   - Check "Establish reference offset"
   - Start probing cycle
4. **Set reference** with `$TLR` command after successful probe
5. **Verify reference** is set with `$#` command (should show TLO values)

### Alternative: Using ioSender

If using ioSender instead of gSender, the process is similar but uses the probing tab with "Establish reference offset" checkbox.

## gSender Tool Change Configuration

### Tool Change Settings

In gSender, navigate to Settings → Tool Change to configure tool change behavior. Available options:

- **Ignore**: Skip M6 commands entirely
- **Pause**: Manual pause for tool changes
- **Standard Re-zero (Wizard)**: Manual re-zeroing process
- **Flexible Re-zero (Wizard)**: Flexible manual process
- **Fixed Tool Sensor (Wizard)**: Automatic with fixed tool setter

### Fixed Tool Sensor Setup

For hardware tool setters, select "Fixed Tool Sensor" which provides a wizard interface for automatic tool length measurement:

1. **Configure tool setter coordinates** in gSender settings
2. **Set probing parameters** (distance, speeds)
3. **Test the setup** with a simple tool change

## Operating Procedures

### Initial Tool Measurement

When starting a new job with the first tool:

1. **Install the tool** in spindle
2. **Home the machine** if not already done
3. **Set work coordinates** (G54) for your workpiece
4. **Perform initial tool measurement**:
   - Manual method: Jog to tool setter, probe, use `$TLR`
   - Automatic method: Issue `M6 T1` to trigger measurement

### Tool Change Operations

During operation, when M6 is encountered in G-code (with $341=3 configured):

1. **Machine automatically**:
   - Stops spindle and coolant
   - Moves to home position in Z-axis
   - Moves to G59.3 position (tool setter)
   - Waits for cycle start

2. **User manually**:
   - Changes the tool
   - Presses cycle start

3. **Machine automatically**:
   - Probes new tool length
   - Calculates offset difference
   - Returns to original cutting position
   - Restores spindle and coolant
   - Continues program

### Using gSender Wizards

When using gSender's Fixed Tool Sensor wizard:

1. **Wizard opens** automatically on M6
2. **Follow on-screen prompts** for tool change
3. **Change tool** when prompted
4. **Click "Run G-code"** to measure new tool
5. **Wizard handles** offset calculations automatically

## Troubleshooting

### Common Issues

**Tool setter not triggering**:
- Check wiring connections
- Verify probe input configuration
- Test with multimeter for continuity

**Incorrect probe positioning**:
- Verify G59.3 coordinates are correct
- Check tool setter physical position
- Ensure adequate clearance for probing

**TLO reference lost**:
Tool length offset reference is cleared on soft reset - re-establish with $TLR after successful probe

**gSender wizard not working**:
Ensure correct tool change mode is selected and firmware supports the feature

### Verification Commands

Check current settings and status:

```
$$            ; Show all settings
$#            ; Show coordinate systems and TLO
$G            ; Show current G-code state
```

### Settings Reset

If tool change settings become corrupted:

```
$RST=$        ; Reset all settings to defaults
```

Then reconfigure tool change settings as outlined above.

## Advanced Features

### Tool Table Support

grblHAL supports persistent tool tables with G10 L1, L10, and L11 commands for manipulating tool data. This allows storing tool lengths permanently.

### Macro-Based ATC

For full automatic tool changers, grblHAL supports macro-based implementations requiring expression and flow control with SD card support.

### Multiple Work Coordinate Systems

You can use different coordinate systems for different setups while maintaining consistent tool length management across all systems.

## Best Practices

1. **Always establish reference** before starting production
2. **Verify tool setter position** regularly
3. **Keep backup** of working settings
4. **Test tool changes** before critical jobs
5. **Monitor probe results** for consistency
6. **Use normally closed** tool setters for safety

## Safety Considerations

- Always ensure spindle is stopped during tool changes
- Verify tool is properly seated before resuming
- Keep hands clear of moving axes during automatic sequences
- Have emergency stop readily accessible
- Never bypass safety interlocks

This guide provides the foundation for reliable tool length measurement and tool changes with grblHAL and gSender. Adapt the procedures to your specific machine configuration and always test thoroughly before production use.

---
