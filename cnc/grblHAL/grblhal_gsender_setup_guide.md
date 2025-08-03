---
layout: default
title: grblHAL & gSender CNC Machine Setup Guide
parent: grblHAL Guide
nav_order: 2
has_children: false
---

# grblHAL & gSender CNC Machine Setup Guide

## Complete Initial Setup, Configuration, and Testing

### Table of Contents
1. [Prerequisites and Requirements](#prerequisites-and-requirements)
2. [Hardware Preparation](#hardware-preparation)
3. [Software Installation](#software-installation)
4. [Initial Connection](#initial-connection)
5. [Basic Configuration](#basic-configuration)
6. [System Testing and Validation](#system-testing-and-validation)
7. [Advanced Configuration](#advanced-configuration)
8. [Troubleshooting Common Issues](#troubleshooting-common-issues)
9. [Reference Tables](#reference-tables)

---

## Prerequisites and Requirements

### Hardware Requirements
- **Controller Board**: grblHAL-compatible board (GRBLHAL2000, Flexi-HAL, etc.)
- **Microcontroller**: Teensy 4.1 or compatible ARM processor
- **Host Computer**: Raspberry Pi 4 (4GB RAM recommended) or Raspberry Pi 5
- **Storage**: MicroSD card (32GB+ recommended, Class 10 or better)
- **Cables**: USB Type-A to Type-B cable (shielded with ferrite cores)
- **CNC Machine**: 3+ axis machine with stepper motors
- **Power Supply**: 5V 3A+ USB-C power supply for Raspberry Pi
- **Optional**: Case, cooling fan, and heat sinks for Raspberry Pi

### Software Requirements
- **Operating System**: Raspberry Pi OS (64-bit recommended)
- **gSender**: Linux ARM64 version from [Sienci Labs](https://sienci.com/gsender/)
- **grblHAL Firmware**: Pre-compiled or custom build
- **Node.js**: Required for gSender operation

### Safety Requirements
- Emergency stop button properly wired
- Limit switches installed (recommended)
- Proper electrical grounding
- Safety glasses and hearing protection

---

## Hardware Preparation

### Step 1: Controller Board Setup

**For GRBLHAL2000/Flexi-HAL boards:**

1. **Install Teensy 4.1** into the breakout board socket
2. **Check jumper settings**:
   - SPIN POWER jumper: Set to 5V or 12V based on spindle requirements
   - USB/Ethernet jumper: Set according to connection method
3. **Verify power connections**:
   - 12V or 24V DC power supply connected
   - All ground connections secure

### Step 2: Raspberry Pi Setup

**Prepare the Raspberry Pi:**

1. **Install Raspberry Pi OS**:
   - Use Raspberry Pi Imager to flash 64-bit OS to microSD card
   - Enable SSH and configure WiFi during imaging (optional)
   - Insert SD card and boot the Pi

2. **Initial Pi Configuration**:
   ```bash
   sudo raspi-config
   ```
   - Enable SSH (Interface Options → SSH)
   - Expand filesystem (Advanced Options → Expand Filesystem)
   - Set GPU memory split to 64MB (Advanced Options → Memory Split)
   - Reboot when prompted

3. **Update System**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

4. **Install Required Dependencies**:
   ```bash
   sudo apt install -y nodejs npm git libgtk-3-0 libx11-xcb1 libdrm2 libxcomposite1 libxdamage1 libxrandr2 libasound2 libpangocairo-1.0-0 libatk1.0-0 libcairo-gobject2 libgtk-3-0 libgdk-pixbuf2.0-0
   ```

### Step 3: Wiring Verification

**Critical connections to verify:**

1. **Stepper Motor Connections**:
   - X, Y, Z axis motors properly connected
   - Motor enable lines connected (if used)
   - Check motor phase wiring for correct rotation

2. **Limit Switch Connections**:
   - Connect to appropriate limit switch inputs
   - Verify normally closed (NC) vs normally open (NO) configuration
   - Default grblHAL expects NC switches

3. **Emergency Stop**:
   - Wire E-stop to dedicated input
   - Ensure fail-safe operation

4. **Spindle Connections**:
   - PWM signal for speed control
   - Direction signal (if applicable)
   - RS485 connections for VFD control (if used)

5. **USB Connection to Raspberry Pi**:
   - Use shielded USB Type-A to Type-B cable
   - Connect to Raspberry Pi USB 3.0 port (blue connector)
   - Keep cable as short as possible to minimize EMI

### Step 3: Power System Check

1. **Measure supply voltages**:
   - Main power: 12V or 24V DC within tolerance
   - Logic power: 3.3V and 5V rails stable
   - Raspberry Pi: 5V 3A+ USB-C power supply
2. **Check current ratings** match motor specifications
3. **Verify proper grounding** throughout system
4. **Electromagnetic Interference (EMI) considerations**:
   - Keep Raspberry Pi away from stepper drivers and spindle
   - Use shielded cables for USB connection
   - Consider metal enclosure for Raspberry Pi if EMI issues persist

---

## Software Installation

### Step 1: Install gSender on Raspberry Pi

**Download and Install gSender:**

1. **Check system architecture**:
   ```bash
   uname -m
   # Should output: aarch64 (for 64-bit ARM)
   ```

2. **Download gSender for Linux ARM64**:
   ```bash
   cd ~/Downloads
   wget https://github.com/Sienci-Labs/gsender/releases/latest/download/gSender-*-linux-arm64.tar.gz
   ```

3. **Extract and install**:
   ```bash
   tar -xzf gSender-*-linux-arm64.tar.gz
   sudo mv gSender-*-linux-arm64 /opt/gsender
   sudo chmod +x /opt/gsender/gSender
   ```

4. **Create desktop shortcut**:
   ```bash
   cat > ~/Desktop/gSender.desktop << EOF
   [Desktop Entry]
   Version=1.0
   Type=Application
   Name=gSender
   Comment=CNC Control Software
   Exec=/opt/gsender/gSender
   Icon=/opt/gsender/resources/app/src/app/images/icon.png
   Terminal=false
   Categories=Development;
   EOF
   chmod +x ~/Desktop/gSender.desktop
   ```

5. **Enable desktop GUI (if running headless)**:
   ```bash
   sudo raspi-config
   # System Options → Boot / Auto Login → Desktop Autologin
   ```

### Step 2: Configure USB Permissions

**Set up USB device access:**

1. **Add user to dialout group**:
   ```bash
   sudo usermod -a -G dialout $USER
   ```

2. **Create udev rule for grblHAL devices**:
   ```bash
   sudo nano /etc/udev/rules.d/99-grblhal.rules
   ```
   Add this content:
   ```
   # grblHAL USB devices
   SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", MODE="0666", GROUP="dialout"
   SUBSYSTEM=="tty", ATTRS{manufacturer}=="Silicon Labs", MODE="0666", GROUP="dialout"
   SUBSYSTEM=="tty", ATTRS{manufacturer}=="Teensyduino", MODE="0666", GROUP="dialout"
   ```

3. **Reload udev rules**:
   ```bash
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   ```

4. **Reboot to apply changes**:
   ```bash
   sudo reboot
   ```

### Step 3: Firmware Preparation

**Option A: Use Pre-compiled Firmware**
- Download appropriate firmware for your board
- Use web builder at http://svn.io-engineering.com:8080/

**Option B: Compile Custom Firmware**
- Install PlatformIO and VSCode
- Clone appropriate driver repository
- Modify configuration files as needed
- Compile and generate .hex file

---

## Initial Connection

### Step 1: Physical Connection

**USB Connection to Raspberry Pi:**
1. **Connect USB cable** from controller board to Raspberry Pi
   - Use USB Type-A to Type-B cable with shielding and ferrite cores
   - Connect to USB 3.0 port (blue connector) on Raspberry Pi for best performance
   - Keep cable length under 3 feet to minimize EMI
   - Avoid USB hubs - connect directly to Pi

2. **Verify USB device detection**:
   ```bash
   lsusb
   # Look for Silicon Labs CP210x or Teensyduino device
   
   dmesg | tail -20
   # Check for USB device messages
   
   ls /dev/ttyUSB* /dev/ttyACM*
   # Should show available serial devices (e.g., /dev/ttyUSB0 or /dev/ttyACM0)
   ```

### Step 2: Launch gSender on Raspberry Pi

1. **Start desktop environment** (if using SSH with X11 forwarding):
   ```bash
   ssh -X pi@your_pi_ip
   DISPLAY=:0 /opt/gsender/gSender
   ```

2. **Launch directly on Pi desktop**:
   - Double-click gSender desktop icon, or
   - Open terminal and run: `/opt/gsender/gSender`

3. **First-time setup**:
   - **Accept or decline** anonymous usage tracking
   - gSender may take longer to start on Pi - be patient
   - **Navigate to connection area** in top-left corner

### Step 3: Establish USB Connection

1. **Click "Connect to CNC" dropdown**
2. **Select correct device**:
   - Look for `/dev/ttyUSB0` or `/dev/ttyACM0`
   - If multiple devices, try each until connection succeeds
3. **Set firmware type** to "grblHAL"
4. **Set baud rate** to 115200
5. **Click Connect**

**Troubleshooting USB connection on Pi:**
- If no devices appear, check USB cable and power
- Run `sudo dmesg | tail` to see recent USB events
- Verify user permissions with `groups $USER` (should include dialout)
- Try different USB port on Raspberry Pi

### Step 4: Verify Connection

**Successful connection indicators:**
- Plug icon turns green with checkmark
- Status changes from "Disconnected" to "Idle" or "Alarm"
- All controls become colored/active
- DRO displays current position

---

## Basic Configuration

### Step 1: Handle Initial Alarm State

**If controller starts in ALARM state:**

This is normal for grblHAL with no limit switches connected.

1. **Temporary fix**: Send `$14=73` to invert control inputs
2. **Send `$X`** to clear alarm state
3. **Alternative**: Temporarily short Reset, E-Stop, and Safety Door inputs to ground

### Step 2: Essential Machine Settings

**Configure these settings first:**

```
$0=10          # Step pulse time (microseconds)
$1=255         # Step idle delay (always enabled)
$2=0           # Step pulse invert mask
$3=0           # Step direction invert mask (adjust per motor)
$4=0           # Step enable invert
$5=0           # Limit pins invert
$6=0           # Probe pin invert
$10=1          # Status report mask
$11=0.010      # Junction deviation (mm)
$12=0.002      # Arc tolerance (mm)
$13=0          # Report in inches (0=mm, 1=inches)
$20=0          # Soft limits enable (enable after homing setup)
$21=1          # Hard limits enable (if limit switches installed)
$22=1          # Homing cycle enable (if limit switches installed)
$23=0          # Homing direction invert mask
$24=25.000     # Homing locate feed rate (mm/min)
$25=500.000    # Homing search seek rate (mm/min)
$26=250        # Homing switch debounce delay (ms)
$27=1.000      # Homing switch pull-off distance (mm)
```

### Step 3: Motor Configuration

**Calculate and set travel resolution for each axis:**

Formula: `steps_per_mm = (motor_steps_per_rev × microstepping) / (belt_pitch × pulley_teeth)`

For ball screws: `steps_per_mm = (motor_steps_per_rev × microstepping) / screw_pitch`

**Example for 1.8° motors, 16 microsteps, 5mm pitch ball screw:**
```
$100=640.000   # X-axis travel resolution (steps/mm)
$101=640.000   # Y-axis travel resolution (steps/mm)
$102=640.000   # Z-axis travel resolution (steps/mm)
```

**Set conservative speeds initially:**
```
$110=1000.000  # X-axis maximum rate (mm/min)
$111=1000.000  # Y-axis maximum rate (mm/min)
$112=500.000   # Z-axis maximum rate (mm/min)
$120=100.000   # X-axis acceleration (mm/sec²)
$121=100.000   # Y-axis acceleration (mm/sec²)
$122=50.000    # Z-axis acceleration (mm/sec²)
```

**Set maximum travel distances:**
```
$130=300.000   # X-axis maximum travel (mm)
$131=200.000   # Y-axis maximum travel (mm)
$132=100.000   # Z-axis maximum travel (mm)
```

---

## System Testing and Validation

### Step 1: Manual Movement Test

1. **Use jog controls** in gSender to test each axis
2. **Start with small movements** (1mm increments)
3. **Verify direction**: Positive jog should move in expected direction
4. **Check for binding** or unusual noises
5. **Test all axes** independently

**If direction is wrong:**
- Modify `$3` (step direction invert) setting
- Bit 0 = X axis, Bit 1 = Y axis, Bit 2 = Z axis
- Example: `$3=1` inverts X-axis direction

### Step 2: Measure Movement Accuracy

1. **Jog 100mm** in each axis direction
2. **Measure actual movement** with ruler or dial indicator
3. **Calculate actual steps/mm**: `actual_steps_per_mm = commanded_distance / actual_distance × current_setting`
4. **Update travel resolution** settings ($100, $101, $102)

### Step 3: Speed and Acceleration Testing

**Gradually increase speeds:**

1. **Start with 500mm/min** maximum rates
2. **Test rapid movements** across full travel
3. **Listen for motor stalling** or grinding
4. **Increase incrementally** until optimal speed found
5. **Test acceleration** values similarly

**Typical values for hobby CNCs:**
- **Small machines**: 1000-3000 mm/min
- **Medium machines**: 3000-6000 mm/min
- **Large/rigid machines**: 6000+ mm/min

### Step 4: Limit Switch Testing (If Installed)

1. **Manually trigger each limit switch**
2. **Verify alarm activation** in gSender
3. **Test pull-off distance** ($27 setting)
4. **Verify proper debounce** ($26 setting)

### Step 5: Homing Cycle Testing

**If limit switches are properly configured:**

1. **Position machine** away from home position
2. **Click Home button** in gSender
3. **Observe homing sequence**:
   - All axes should move toward limits
   - Should back off after contact
   - Should perform precision approach
4. **Verify final position** is consistent

---

## Advanced Configuration

### Step 1: Soft Limits Setup

**After homing is working properly:**

1. **Enable soft limits**: `$20=1`
2. **Test boundaries** by jogging near limits
3. **Verify soft limit alarms** trigger before hard limits

### Step 2: Spindle Configuration

**PWM Spindle Control:**
```
$30=1000       # Spindle max RPM
$31=0          # Spindle min RPM
$32=1          # Laser mode enable (for laser applications)
```

**VFD (RS485) Spindle Control:**
1. **Configure VFD settings** per manufacturer instructions
2. **Set grblHAL for VFD spindle** in gSender
3. **Test speed control** with M3 S commands

### Step 3: Probing Setup

**Configure touch plate probing:**

1. **Connect probe input** to appropriate pin
2. **Set probe invert** if needed: `$6=1`
3. **Test probe continuity** in gSender
4. **Calibrate probe offset** for touch plate thickness

### Step 4: Work Coordinate Systems

**Set up multiple work coordinate systems:**

```
G54    # Work coordinate system 1
G55    # Work coordinate system 2
G56    # Work coordinate system 3
```

Use G10 commands to set coordinate system origins.

### Step 5: Custom Macros

**Create useful macros in gSender:**

- **Home and zero**: G28, G10 L20 P0 X0 Y0 Z0
- **Safe height**: G0 Z5
- **Spindle warm-up**: M3 S1000, G4 P30, M5

---

## Troubleshooting Common Issues

### Connection Problems

**Issue**: Cannot connect to controller
**Solutions**:
- Verify USB cable and drivers
- Check COM port selection
- Try different USB port
- Ensure no other software is using port
- Check power supply to controller

**Issue**: Connects but shows "Disconnected" status
**Solutions**:
- Verify firmware type selection (grblHAL vs grbl)
- Check baud rate setting (115200)
- Try different USB cable
- Check controller board connections

### Movement Issues

**Issue**: Motors not moving or moving wrong direction
**Solutions**:
- Check motor connections and phases
- Verify step direction invert settings ($3)
- Check enable signal configuration ($4)
- Verify power supply voltage and current

**Issue**: Motors stalling or skipping steps
**Solutions**:
- Reduce maximum rate settings ($110-$112)
- Reduce acceleration settings ($120-$122)
- Check motor current settings on drivers
- Verify adequate power supply capacity

**Issue**: Inconsistent movement or position loss
**Solutions**:
- Check mechanical binding and belt tension
- Verify step pulse timing ($0)
- Check for electrical interference
- Ensure proper grounding

### Alarm States

**Issue**: Controller starts in ALARM state
**Solutions**:
- Send `$14=73` to invert control inputs
- Connect limit switches properly
- Send `$X` to clear alarm
- Check safety door and e-stop connections

**Issue**: Hard limit alarms during operation
**Solutions**:
- Check limit switch wiring and mounting
- Verify switch debounce settings ($26)
- Check for mechanical binding
- Verify soft limits are properly configured

**Issue**: Probe alarms during probing
**Solutions**:
- Check probe connectivity and wiring
- Verify probe invert setting ($6)
- Ensure probe plate is electrically connected
- Check for electrical interference

### Raspberry Pi Specific Issues

**Issue**: gSender won't start or crashes on launch
**Solutions**:
- Ensure sufficient RAM (4GB+ recommended)
- Close other applications to free memory
- Check for GPU memory split: `sudo raspi-config` → Advanced → Memory Split → 64
- Verify all dependencies installed
- Try running from terminal to see error messages

**Issue**: USB device not detected on Raspberry Pi
**Solutions**:
- Check device appears in `lsusb` output
- Verify user in dialout group: `groups $USER`
- Check udev rules are properly configured
- Try different USB port on Pi
- Verify USB cable and connections
- Check dmesg for USB error messages

**Issue**: Poor performance or lag on Raspberry Pi
**Solutions**:
- Use Raspberry Pi 4 with 4GB+ RAM
- Close unnecessary applications and services
- Disable WiFi and Bluetooth if not needed:
  ```bash
  sudo systemctl disable bluetooth
  sudo systemctl disable wifi
  ```
- Increase GPU memory split to 128MB if graphics are slow
- Use faster microSD card (Class 10 or better)
- Consider SSD via USB 3.0 for better I/O performance

**Issue**: Display issues on Raspberry Pi
**Solutions**:
- Ensure proper display configuration in `/boot/config.txt`
- Try different display resolution
- Enable full desktop: `sudo raspi-config` → System → Boot → Desktop Autologin
- For remote access, use VNC or X11 forwarding

**Issue**: USB communication drops or errors
**Solutions**:
- Use high-quality shielded USB cable
- Keep cable length under 3 feet
- Add ferrite cores to cable ends
- Move Pi away from stepper drivers and spindle
- Use USB 3.0 port on Pi for better power delivery
- Check power supply adequacy (3A+ recommended)
- Add powered USB hub if power is insufficient

### Performance Issues

**Issue**: Poor surface finish or dimensional accuracy
**Solutions**:
- Reduce feed rates and optimize speeds
- Check belt tension and mechanical rigidity
- Adjust acceleration and jerk settings
- Verify step resolution calculations

**Issue**: System feels sluggish or unresponsive
**Solutions**:
- Enable aggressive buffering in gSender
- Use Ethernet connection instead of USB
- Reduce status report frequency
- Check computer performance and USB drivers

### Communication Errors

**Issue**: Random errors during job execution
**Solutions**:
- Use shielded USB cable with ferrite cores
- Switch to Ethernet connection
- Move computer away from spindle/VFD
- Check electrical grounding of entire system
- Enable synchronous transfer mode in gSender

---

## Reference Tables

### Common grblHAL Settings

| Setting | Parameter | Description | Default |
|---------|-----------|-------------|---------|
| $0 | Step pulse time | Microseconds | 10 |
| $1 | Step idle delay | 255 = always on | 255 |
| $3 | Step direction invert | Bit mask for direction | 0 |
| $10 | Status report mask | Status report options | 1 |
| $20 | Soft limits | Enable/disable | 0 |
| $21 | Hard limits | Enable/disable | 0 |
| $22 | Homing cycle | Enable/disable | 0 |
| $100-$102 | Travel resolution | Steps per mm | Machine specific |
| $110-$112 | Maximum rate | mm/min | Machine specific |
| $120-$122 | Acceleration | mm/sec² | Machine specific |
| $130-$132 | Maximum travel | mm | Machine specific |

### Error Code Reference

| Code | Description | Common Causes |
|------|-------------|---------------|
| 1 | Expected command letter | Malformed G-code |
| 2 | Bad number format | Invalid numeric value |
| 3 | Invalid statement | Unsupported G-code |
| 9 | G-code locked | Controller in alarm state |
| 20 | Unsupported command | Wrong device type selected |
| 33 | Invalid target | Arc movement calculation error |

### Alarm Code Reference

| Code | Description | Action Required |
|------|-------------|-----------------|
| 1 | Hard limit | Check limit switches, re-home |
| 2 | Soft limit | Adjust G-code or work coordinates |
| 3 | Abort during cycle | Reset and investigate cause |
| 4 | Probe fail | Check probe connection and setup |
| 8 | Homing fail | Check limit switches and homing setup |

### Connection Troubleshooting

| Symptom | Possible Cause | Solution |
|---------|----------------|----------|
| No COM ports visible | Missing USB drivers | Install CP2102 drivers |
| Connection timeout | Wrong baud rate | Set to 115200 |
| Connects but no response | Wrong firmware type | Select grblHAL |
| Intermittent connection | Poor USB cable | Use shielded cable |
| Cannot connect via Ethernet | IP configuration | Check static IP settings |

---

## Final Verification Checklist

- [ ] Raspberry Pi boots and runs stable
- [ ] gSender starts without errors
- [ ] USB device detected and accessible (/dev/ttyUSB0 or /dev/ttyACM0)
- [ ] Controller powers up without alarms (or expected alarm cleared)
- [ ] USB connection establishes successfully in gSender
- [ ] All axes jog in correct directions
- [ ] Movement measurements match commanded distances
- [ ] Limit switches trigger properly (if installed)
- [ ] Homing cycle completes successfully (if configured)
- [ ] Spindle control responds to commands
- [ ] Emergency stop functions properly
- [ ] Soft limits prevent over-travel
- [ ] Settings saved and persist after power cycle
- [ ] Test G-code file runs without errors
- [ ] System remains stable during extended operation
- [ ] No USB communication errors during normal use

## Additional Resources

- **grblHAL Core Repository**: https://github.com/grblHAL/core
- **gSender Documentation**: https://resources.sienci.com/
- **grblHAL Wiki**: https://github.com/grblHAL/core/wiki
- **PrintNC Community**: https://wiki.printnc.info/
- **ioSender (Alternative)**: https://github.com/terjeio/Grbl-GCode-Sender

## Support and Community

For additional help:
- **gSender Support**: Sienci Labs community forums
- **grblHAL Support**: GitHub issues and discussions
- **General CNC Support**: CNC forums and maker communities

---

*This guide is based on grblHAL firmware and gSender software as of January 2025. Always refer to the latest documentation for the most current information.*

---
