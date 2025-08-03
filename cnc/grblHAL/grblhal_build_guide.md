---
layout: default
title: grblHAL Firmware Build and Installation Guide
parent: grblHAL Guide
nav_order: 1
has_children: false
---

# grblHAL Firmware Build and Installation Guide

## Overview

grblHAL is a high-performance CNC control firmware that supports a wide range of microcontrollers and development boards. This guide covers two methods for building and installing grblHAL firmware:

- **Web Builder** - Browser-based tool for quick, customized builds
- **Manual Build** - Using PlatformIO for advanced customization and development

## Prerequisites

### Hardware Requirements
- Supported microcontroller board (ESP32, STM32, RP2040, etc.)
- USB cable for programming
- Computer with internet access

### Software Requirements (Manual Build Only)
- [Visual Studio Code](https://code.visualstudio.com/)
- [PlatformIO IDE extension](https://platformio.org/install/ide-vscode)
- Git (for cloning repositories)

## Method 1: Web Builder (Recommended for Most Users)

The grblHAL Web Builder provides an easy way to create custom firmware without installing development tools.

### Step 1: Access the Web Builder

1. Navigate to the [grblHAL Web Builder](https://github.com/grblHAL/core/wiki/Web-Builder-Installation)
2. Select your target board from the dropdown menu
3. Choose your desired configuration options

### Step 2: Configure Your Build

**Board Selection:**
- Choose your specific development board (e.g., ESP32 DevKit, Black Pill STM32F411)
- Verify the pin assignments match your hardware setup

**Feature Configuration:**
- **Spindle Control**: PWM, relay, or VFD options
- **Limit Switches**: Enable/disable, configure pull-up/pull-down
- **Probe Support**: Touch probe, tool length sensor
- **Safety Features**: Safety door, feed hold, cycle start
- **Communication**: USB, UART, WiFi, Bluetooth options
- **Additional Features**: Laser mode, plasma mode, gang/auto-squaring

### Step 3: Generate and Download

1. Click **"Build Firmware"** after configuring options
2. Wait for the build process to complete (typically 1-2 minutes)
3. Download the generated firmware file (.bin, .uf2, or .hex depending on your board)

### Step 4: Install Firmware

**For ESP32 boards:**
1. Download and install [ESP32 Flash Tool](https://www.espressif.com/en/support/download/other-tools)
2. Connect your ESP32 via USB
3. Put the board in programming mode (hold BOOT button, press RESET)
4. Flash the firmware at address 0x10000
5. Reset the board

**For RP2040 boards (Pico, etc.):**
1. Hold the BOOTSEL button while connecting USB
2. Board appears as USB mass storage device
3. Copy the .uf2 file to the mounted drive
4. Board automatically resets and runs new firmware

**For STM32 boards:**
1. Use [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) or [dfu-util](http://dfu-util.sourceforge.net/)
2. Put board in DFU mode (varies by board)
3. Flash the .bin file using your chosen tool

## Method 2: Manual Build with PlatformIO

Manual building provides maximum flexibility and is recommended for developers or users needing custom modifications.

### Step 1: Environment Setup

**Install Visual Studio Code:**
1. Download from [code.visualstudio.com](https://code.visualstudio.com/)
2. Install and launch VS Code

**Install PlatformIO:**
1. Open VS Code Extensions (Ctrl+Shift+X)
2. Search for "PlatformIO IDE"
3. Install the extension and reload VS Code
4. Wait for PlatformIO to complete initial setup

### Step 2: Clone grblHAL Repository

**Option A: Using VS Code Terminal**
```bash
git clone https://github.com/grblHAL/core.git grblHAL-core
cd grblHAL-core
git submodule update --init --recursive
```

**Option B: Using PlatformIO**
1. Open PlatformIO Home
2. Click "Clone Git Project"
3. Enter repository URL: `https://github.com/grblHAL/core.git`
4. Initialize submodules when prompted

### Step 3: Select Target Platform

Navigate to the appropriate driver folder for your hardware:
- `drivers/ESP32/` - ESP32 boards
- `drivers/STM32F4xx/` - STM32F4 series
- `drivers/RP2040/` - Raspberry Pi Pico and similar
- `drivers/STM32F1xx/` - STM32F1 series (Blue Pill, etc.)

### Step 4: Configure Build Environment

**Edit platformio.ini:**
1. Open the `platformio.ini` file in your chosen driver folder
2. Uncomment your specific board configuration
3. Modify build flags as needed:

```ini
[env:my_board]
platform = espressif32
board = esp32dev
framework = arduino
build_flags = 
    -D BOARD_NAME="My CNC Controller"
    -D ENABLE_WIFI
    -D SPINDLE_PWM_PIN=2
    -D COOLANT_FLOOD_PIN=4
```

### Step 5: Customize Configuration

**Edit my_machine.h (if exists):**
```c
// Example customizations
#define SPINDLE_PWM_PIN         2
#define COOLANT_FLOOD_PIN       4  
#define COOLANT_MIST_PIN        16
#define PROBE_PIN               15

// Enable features
#define ENABLE_SPINDLE_LINEARIZATION 1
#define LASER_MODE_ENABLE 1
```

**Board-specific configurations:**
- Check `Inc/` folder for configuration templates
- Copy and modify templates for your specific setup
- Verify pin assignments match your hardware

### Step 6: Build Firmware

1. Open PlatformIO terminal (Terminal > New Terminal)
2. Navigate to your driver folder
3. Build the project:
```bash
pio run
```

**Build for specific environment:**
```bash
pio run -e your_board_name
```

**Clean build (if needed):**
```bash
pio run -t clean
pio run
```

### Step 7: Upload Firmware

**Direct upload (if board is connected):**
```bash
pio run -t upload
```

**Manual upload:**
1. Locate built firmware in `.pio/build/[environment]/`
2. Use appropriate flashing tool for your board type
3. Follow board-specific upload procedures

## Verification and Testing

### Initial Connection Test

1. Connect board via USB
2. Open serial terminal (115200 baud, 8N1)
3. Send `$$` command
4. Verify grblHAL responds with settings list

### Basic Function Test

```
$I          // Show build info
$$          // Show all settings  
$#          // Show coordinate systems
G21 G90     // Set metric units, absolute positioning
G0 X10      // Test axis movement (without steppers connected)
```

### WiFi Setup (ESP32 only)

```
$WIFI/STA=your_ssid,your_password    // Connect to WiFi
$#                                   // Verify IP address assigned
```

## Troubleshooting

### Common Build Issues

**Missing dependencies:**
```bash
pio lib install
pio pkg update
```

**Submodule issues:**
```bash
git submodule update --init --recursive --force
```

**Memory issues:**
- Reduce enabled features in configuration
- Use optimization flags: `-Os` for size

### Upload Issues

**ESP32 not entering programming mode:**
- Hold BOOT button, press RESET, release RESET, release BOOT
- Try different USB cable or port
- Install ESP32 USB drivers

**STM32 DFU mode problems:**
- Verify BOOT0 jumper position
- Use STM32CubeProgrammer for automatic mode detection
- Check for proper USB-C cable (not charge-only)

**Permission issues (Linux/Mac):**
```bash
sudo usermod -a -G dialout $USER
# Log out and back in
```

### Runtime Issues

**No response on serial:**
- Verify baud rate (115200)
- Check USB cable and drivers
- Try different terminal software

**Axis not moving:**
- Verify pin assignments in configuration
- Check stepper driver connections
- Ensure adequate power supply

## Advanced Configuration

### Custom Pin Assignments

Create or modify board-specific header files:
```c
// In your_board_map.h
#define X_STEP_PIN              GPIO_NUM_2
#define X_DIRECTION_PIN         GPIO_NUM_3
#define Y_STEP_PIN              GPIO_NUM_4
#define Y_DIRECTION_PIN         GPIO_NUM_5
// ... continue for all required pins
```

### Plugin Development

1. Create plugin folder in `plugins/`
2. Implement required callback functions
3. Register plugin in main configuration
4. Build and test thoroughly

### Network Configuration

**Access Point mode:**
```c
#define WIFI_AP_SSID "grblHAL"
#define WIFI_AP_PASSWORD "grblHAL123"
```

**Static IP configuration:**
```c
#define WIFI_STA_IP "192.168.1.100"
#define WIFI_STA_GATEWAY "192.168.1.1"
#define WIFI_STA_MASK "255.255.255.0"
```

## Additional Resources

- [grblHAL Wiki](https://github.com/grblHAL/core/wiki) - Comprehensive documentation
- [grblHAL Discussion Forum](https://github.com/grblHAL/core/discussions) - Community support
- [PlatformIO Documentation](https://docs.platformio.org/) - Build system reference
- [Board-specific guides](https://github.com/grblHAL/core/wiki/Compiling-GrblHAL) - Detailed setup instructions

## Support

For additional help:
1. Check existing [GitHub issues](https://github.com/grblHAL/core/issues)
2. Search the [community discussions](https://github.com/grblHAL/core/discussions)
3. Join the [grblHAL Discord](https://discord.gg/8KdMSDcYJX) for real-time support

Remember to include your board type, build method, and specific error messages when seeking help.

---
