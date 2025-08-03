---
layout: default
title: grblHAL G100 VFD RS485 Configuration Guide
parent: grblHAL Guide
nav_order: 5
has_children: false
---

# grblHAL H100 VFD RS485 Configuration Guide

## Overview

This guide provides step-by-step instructions for configuring grblHAL to communicate with an H100 VFD (Variable Frequency Drive) over RS485 using the Modbus RTU protocol. The H100 VFD supports standard Modbus RTU communication protocol over RS485, and grblHAL has built-in support for H100 VFDs with dedicated driver support.

## Prerequisites

- grblHAL controller board with RS485 communication capability
- H100 VFD (any model in the H100 series)
- Spindle motor compatible with your H100 VFD
- RS485 to TTL converter module (if required by your controller)
- Twisted pair cable for RS485 communication
- ioSender or compatible grblHAL sender software

## Hardware Setup

### 1. Power and Motor Connections

Before configuring RS485 communication, ensure your VFD and spindle are properly connected:

1. **Power Input**: Connect power according to your VFD model - single-phase 220V models connect to L1/L2, three-phase models connect to L1/L2/L3
2. **Motor Output**: Connect spindle motor to U, V, W terminals
3. **Ground**: Connect PE terminal to proper ground - 220V grade requires third type earthing (resistance below 100Ω), 380V grade requires earthing resistance below 10Ω

### 2. RS485 Wiring

#### grblHAL Controller Side
Connect the RS485 A & B terminals from your grblHAL controller to the VFD:
- **A terminal** → **RS+** on VFD
- **B terminal** → **RS-** on VFD

#### H100 VFD Side
The H100 VFD has built-in RS485 terminals labeled 485+ and 485-:
- **485+** connects to **A** from controller
- **485-** connects to **B** from controller

#### Cable Requirements
- Use twisted pair cable for RS485 connections
- For shielded cable, connect shield to ground on VFD side only
- Keep RS485 cable separate from power cables to minimize interference

## grblHAL Firmware Configuration

### 1. Enable H100 VFD Support

In your `my_machine.h` file, enable H100 VFD support:

```c
#define VFD_SPINDLE SPINDLE_H100  // Enable H100 VFD support
#define MODBUS_ENABLE 1           // Enable Modbus communication
```

### 2. grblHAL Settings Configuration

After flashing firmware with H100 support, configure these settings:

#### Basic VFD Settings
```
$360=1          // Modbus address (default 1, must match VFD setting F163)
$395=1          // Bind spindle 0 to H100 VFD (if multiple spindles configured)
$461=60         // RPM to Hz relationship (default 60)
```

#### Spindle Speed Settings
```
$30=24000       // Maximum spindle speed (RPM)
$31=0           // Minimum spindle speed (RPM)
$32=0           // Laser mode enable (0=disabled for spindle)
```

## H100 VFD Parameter Configuration

Configure these essential parameters on your H100 VFD for RS485 communication:

### 1. Control Source Settings
```
F001=2          // Control mode: Communication port
F002=2          // Frequency setting: Communication port
```

### 2. Communication Parameters
```
F163=1          // Communication address (1-250, must match grblHAL $360)
F164=2          // Communication speed: 19200 bps
F165=4          // Communication data mode: 8E1 for RTU
F169=0          // Communication frequency decimal point: 1 bit
```

### 3. Motor Parameters
Configure these based on your spindle specifications:
```
F140=1.5        // Rated power of motor (kW) - match your spindle
F141=220        // Rated voltage of motor (V) - match your spindle  
F142=8.0        // Rated current of motor (A) - check spindle nameplate
F143=4          // Number of motor poles (typically 4)
F144=24000      // Motor rated speed (RPM) - match your spindle
```

### 4. Frequency and Voltage Settings
```
F004=400        // Reference frequency (Hz) - typically 400 for high-speed spindles
F005=400        // Maximum operating frequency (Hz)
F006=5.0        // Intermediate frequency (Hz)
F007=60         // Starting frequency (Hz) - minimum operating frequency
F008=220        // Maximum voltage (V) - match your input voltage
```

### 5. Additional Settings
```
F023=1          // Reverse allow (1=enabled)
F024=1          // Stop key valid
F025=0          // Start mode: Start from starting frequency
F026=0          // Stop mode: Ramp deceleration
```

## Testing and Verification

### 1. Initial Communication Test

1. **Power up both systems**: Start VFD and grblHAL controller
2. **Check communication**: When properly connected, you should see both red and green RS485 communication LEDs flashing on the grblHAL board
3. **Verify in ioSender**: 
   - Set Spindle type to "VFD Spindle" in grblHAL settings
   - ioSender should display the correct RPM when you control the VFD manually

### 2. Manual VFD Operation Test

Before testing with grblHAL:
1. Set VFD to manual mode (F001=0)
2. Test spindle operation using VFD panel controls
3. Verify proper rotation direction (clockwise when viewed from top)
4. Test speed control and verify current draw is within expected range

### 3. grblHAL Control Test

1. **Switch to communication mode**: Set F001=2 on VFD
2. **Test basic commands**:
   ```
   M3 S6000    // Start spindle at 6000 RPM clockwise
   M4 S8000    // Start spindle at 8000 RPM counterclockwise  
   M5          // Stop spindle
   ```
3. **Monitor feedback**: Verify ioSender shows actual spindle speed

## Troubleshooting

### Communication Issues

**No communication (LEDs not flashing)**:
- Verify RS485 wiring polarity (A to RS+, B to RS-)
- Check Modbus addresses match (grblHAL $360 = VFD F163)
- Verify communication parameters match on both systems
- Enable termination resistor on VFD if needed (J4 jumper ON)

**CRC Errors**:
- Common issue with H100 VFDs - may indicate communication timing issues
- Try reducing communication speed (F164=1 for 9600 bps)
- Verify cable quality and length
- Check for electromagnetic interference

### Spindle Control Issues

**Spindle won't start**:
- Verify VFD is in communication mode (F001=2, F002=2)
- Check minimum frequency setting (F007) isn't too high
- Ensure motor parameters (F140-F144) match your spindle

**Incorrect speed**:
- Verify RPM to Hz relationship ($461 setting)
- Check maximum frequency settings (F005)
- Calibrate speed feedback if available

**Wrong rotation direction**:
- For manual correction: swap any two motor wires (U, V, W)
- For software correction: use grblHAL spindle direction settings

## Advanced Configuration

### Multiple VFD Setup

If using multiple VFDs:
```
$476=1          // Modbus address for spindle 0
$477=2          // Modbus address for spindle 1  
$511=2          // Bind spindle 1 to VFD id
```

### Custom Register Settings

For advanced users, grblHAL supports custom Modbus register configuration:
```
$462=8192       // Run/stop register
$463=8193       // Set frequency register  
$464=8451       // Get frequency register
```

## Modbus Register Reference

### H100 VFD Key Registers

Based on the H100 manual, key Modbus registers include:

| Function | Register | Description |
|----------|----------|-------------|
| Control | 0x2000 | Main control register |
| Frequency Set | 0x2001 | Frequency command (0.1Hz resolution) |
| Frequency Read | 0x220H | Current output frequency |
| Current Read | 0x221H | Output current |
| Status | 0x210H | VFD status bits |

### Register Data Format

Data is transmitted as hexadecimal integers. The minimum unit depends on parameter decimal points. For frequency with 0.1Hz resolution, communication value 300 represents 30.0Hz.

## Safety Considerations

1. **Always test manual operation first** before enabling communication control
2. **Implement emergency stops** that bypass communication system
3. **Monitor initial operation closely** for unexpected behavior
4. **Verify proper grounding** of all components
5. **Use appropriate fusing and circuit protection**

## Conclusion

Following this guide should result in reliable RS485 communication between grblHAL and your H100 VFD. The key factors for success are:

- Proper physical connections with twisted pair cable
- Matching communication parameters on both systems  
- Correct motor parameter configuration
- Systematic testing from manual to automatic control

RS485 control is more robust than analog control as it allows bidirectional communication and actual speed feedback, making it the preferred method for professional CNC installations.

For additional support, consult the grblHAL community forums, PrintNC Discord, and the H100 VFD manual for your specific model.

---
