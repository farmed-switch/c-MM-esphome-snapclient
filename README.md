# ESPHome Snapclient for ESP32 Audio Boards

ESPHome-based Snapcast client with **modular board profiles** and optional software EQ.  
Flash directly from **Home Assistant** or esphome.io web interface.

**New in this fork:**
- 🎛️ **Board profile system** - Reusable hardware packages for all Sonocotta boards
- 🎚️ **Optional software EQ** - 18-band parametric EQ (separate package)
- 🔌 **Plug-and-play** - Mix and match board + features with ESPHome packages
- 📦 **15+ board profiles** - Amped, Louder, HiFi, Loud series (ESP32 & ESP32-S3)

---

## 🚀 Quick Start

### 1. Choose Your Configuration Method

**Option A: Local includes** (when cloned repo):
```yaml
packages:
  board: !include boards/sonocotta/amped-esp32-s3-plus.yaml
  soft_eq: !include boards/software/soft-eq-18band.yaml
# Add wifi, api, snapclient...
```

**Option B: Remote GitHub packages** (ESPHome directly from web):
```yaml
packages:
  - url: https://github.com/farmed-switch/c-MM-esphome-snapclient
    ref: board-profiles  # or main
    files: [boards/sonocotta/amped-esp32-s3-plus.yaml]
    refresh: 0s
  - url: https://github.com/farmed-switch/c-MM-esphome-snapclient
    ref: board-profiles
    files: [boards/software/soft-eq-18band.yaml]
    refresh: 0s
```

**Minimal (no EQ):**
```yaml
packages:
  - url: https://github.com/farmed-switch/c-MM-esphome-snapclient
    ref: board-profiles
    files: [boards/sonocotta/amped-esp32-s3-plus.yaml]
    refresh: 0s
# Add wifi, api, snapclient...
```

### 2. Flash from Home Assistant
1. Copy example YAML from `examples/` directory (or use remote GitHub packages)
2. Create `secrets.yaml` with your WiFi and Snapserver details
3. Flash via ESPHome dashboard → Done!

**→ See [examples/](examples/) for complete ready-to-flash configs**  
**→ [amped-esp32-s3-plus-example.yaml](examples/amped-esp32-s3-plus-example.yaml)** - Complete example with remote packages

---

## 📦 What's Included

### Board Profiles (`boards/sonocotta/`)
Hardware-only packages for [Sonocotta ESP32 Audio Dock](https://github.com/sonocotta/esp32-audio-dock) series:

| Board Family | DAC/Amp | Output | EQ Type | Profiles |
|--------------|---------|--------|---------|----------|
| **Amped-ESP32** | PCM5100+TPA3128 | 22W @ 8Ω | Software | esp32, esp32-s3, tpa3110 |
| **Amped-ESP32-S3-Plus** 🆕 | PCM5122+TPA3128 | 22W @ 8Ω | Software | **s3-plus** |
| **Louder-ESP32** | TAS5805M | 25W @ 8Ω | Hardware 15-band | esp32, esp32-s3, mic |
| **Louder-ESP32-S3-Plus** 🆕 | TAS5825M | 25W @ 8Ω | Hardware 15-band | **s3-plus** |
| **HiFi-ESP32** | PCM5100A | Line 2.1V | Software | esp32, esp32-s3, s3-mic |
| **HiFi-ESP32-Plus** | PCM5122 | Line 2.1V | Software | esp32-plus, s3-plus |
| **Loud-ESP32** | MAX98357 | 3-5W | Software | esp32, esp32-s3 |

🆕 = Newly added Plus models with OLED display, RGB LED, IR receiver, Ethernet (W5500)

**→ [Full board selection guide](boards/sonocotta/README.md)**  
**→ [Hardware revision identification](boards/sonocotta/REVISIONS.md)**  
**→ [Complete pinout reference](boards/sonocotta/PINOUT.md)**

### Software EQ Package (`boards/software/soft-eq-18band.yaml`)
Optional 18-band parametric EQ for boards without hardware DSP:

- **Frequencies:** 40, 50, 60, 70, 80, 90, 100, 110, 120, 130, 140, 200, 315, 500, 800, 1250, 2000, 5000 Hz
- **Range:** ±15dB per band
- **Presets:** Flat, Subwoofer, Bookshelf, Floor Standing, Near Field, Small/Portable, Custom
- **Home Assistant:** Full integration with sliders + preset selector
- **CPU:** ~18ms loop time @ ESP32-S3 240MHz

**Compatible with:** Amped, HiFi, Loud boards (NOT Louder - it has hardware EQ)

---

## Configuration Examples

**Instead of long YAML snippets here, we now provide complete ready-to-flash examples:**

### 📄 Example 1: [Amped ESP32-S3 with Software EQ](examples/amped-esp32-s3-with-eq.yaml)
Generic I2S board + 18-band parametric EQ (40Hz-5kHz):
```yaml
packages:
  board: !include boards/sonocotta/amped-esp32-s3.yaml
  soft_eq: !include boards/software/soft-eq-18band.yaml
```
**Features:** WiFi, 18-band EQ, Home Assistant integration, 6 presets

---

### 📄 Example 2: [Amped ESP32-S3 Minimal](examples/amped-esp32-s3-minimal.yaml)
Generic I2S board WITHOUT EQ (lowest latency):
```yaml
packages:
  board: !include boards/sonocotta/amped-esp32-s3.yaml
```
**Features:** WiFi, clean audio path, minimal dependencies

---

### 📄 Example 3: [Louder ESP32-S3 with Hardware EQ](examples/louder-esp32-s3-hardware-eq.yaml)
TAS5805M professional DAC + 15-band hardware EQ (20Hz-16kHz):
```yaml
packages:
  board: !include boards/sonocotta/louder-esp32-s3.yaml
# Hardware EQ built-in! 0% CPU usage
```
**Features:** Ethernet, 15-band hardware EQ, fault monitoring, better signal quality

---

**→ All examples in [examples/](examples/) directory with detailed comments**

---

## 🎛️ Optional Hardware Features (Plus Models)

**Plus models** include additional hardware that can be activated in your config:

| Feature | Plus Models | GPIO | Purpose |
|---------|-------------|------|---------|
| **RGB LED** | All Plus | GPIO21 | Status indicator, audio reactive |
| **IR Receiver** | All Plus | GPIO07 | Remote control input |
| **OLED Display** | All Plus | SPI | Show IP, stats, playback info |
| **Ethernet** | All Plus | W5500 | Wired network (lower latency) |

**How to activate:** See [examples/README.md](examples/README.md#-optional-hardware-features-plus-models) for complete instructions with code examples.

---

## 🔧 Important Configuration Notes

### ESP32-S3 Requirements

**ALL ESP32-S3 boards MUST include these settings:**

```yaml
substitutions:
  task_stack_in_psram: "true"  # REQUIRED

logger:
  hardware_uart: USB_SERIAL_JTAG  # REQUIRED for serial output

esp32:
  framework:
    sdkconfig_options:
      CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG: "y"
      CONFIG_ESP_CONSOLE_SECONDARY_NONE: "y"
      # ... other ESP32-S3 settings
```

**Without these:** No serial output, debugging impossible, potential boot issues.

### Software EQ Requirements

When using soft-eq-18band.yaml package:

```yaml
esp32:
  framework:
    sdkconfig_options:
      CONFIG_USE_DSP_PROCESSOR: "y"
      CONFIG_DSP_OPTIMIZED: "y"  # ESP32-S3 optimizations
      CONFIG_DSP_MAX_FFT_SIZE: "4096"
      CONFIG_SNAPCLIENT_DSP_FLOW_BASS_TREBLE_EQ: "n"  # Use custom EQ
```

### Network Configuration

**WiFi (recommended for most users):**
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  power_save_mode: none  # CRITICAL for audio!
  output_power: 20dB     # Maximum signal strength
```

**Ethernet (Plus models, lower latency):**
```yaml
ethernet:
  type: W5500
  # ... see examples for full config
```

### Critical Snapclient Settings

```yaml
snapclient:
  hostname: !secret snapserver_host  # Your Snapcast server IP
  port: 1704
  name: ${friendly_name}
  i2s_dout_pin: GPIO16  # ESP32-S3: GPIO16, ESP32: GPIO25
  audio_dac: pcm5122_dac  # CRITICAL: Link to DAC from board profile
```

---

## 🐛 Troubleshooting

### No Serial Output (ESP32-S3)
- ✅ Add `hardware_uart: USB_SERIAL_JTAG` to logger
- ✅ Add `CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG: "y"` to sdkconfig
- ✅ Check baud rate: 115200
- ✅ Verify USB cable supports data (not just power)

### No Audio Output
- ✅ Verify Snapcast server is running: `systemctl status snapserver`
- ✅ Check server reachable: `ping 192.168.1.x`
- ✅ Confirm `audio_dac: pcm5122_dac` in snapclient config
- ✅ Check DAC enable switch in Home Assistant (should be ON)
- ✅ Verify amplifier enable switch (if needed)

### WiFi Connection Issues
- ✅ Use 2.4GHz WiFi only (ESP32-S3 doesn't support 5GHz)
- ✅ Set `power_save_mode: none`
- ✅ Check SSID/password in secrets.yaml
- ✅ Verify WiFi signal strength sensor (should be > -70dBm)

### EQ Not Working
- ✅ Verify soft-eq-18band.yaml package is included
- ✅ Check `CONFIG_USE_DSP_PROCESSOR: "y"` in sdkconfig
- ✅ Look for 18 EQ number entities in Home Assistant
- ✅ Enable EQ master switch

### High Loop Time / Audio Glitches
- ✅ Disable `debug:` component (adds overhead)
- ✅ Set logger `level: INFO` instead of DEBUG
- ✅ Check Free PSRAM sensor (should be > 7MB)
- ✅ Verify WiFi signal strength (> -70dBm)
- ✅ Consider Ethernet for critical installations

### Build/Flash Errors
- ✅ Use ESPHome 2026.1.0 or newer
- ✅ Check `min_version: 2025.7.0` in esphome section
- ✅ Verify all packages exist on correct branch
- ✅ Clear ESPHome build cache if issues persist

**More help:** Open an issue on GitHub with full log output.

---

## Hardware Configurations

**Amped ESP32-S3 Features:**
- Generic I2S DAC (PCM5102A or similar)  
- Software 18-band EQ (40Hz-5kHz optimized for bass/subwoofers)
- WiFi connectivity
- ~18ms loop time with EQ active
- 7 presets: Flat, Subwoofer, Bookshelf, Floor Standing, Near Field, Small/Portable

**Louder ESP32-S3 Features:**
- TAS5805M professional DAC with hardware DSP  
- Hardware 15-band EQ (20Hz-16kHz ISO standard frequencies)
- Ethernet (W5500) for reliable audio streaming
- 0% ESP32 CPU usage (EQ processed in DAC chip)
- Better signal quality (DSP before D/A conversion)
- Fault monitoring and diagnostics

## Secrets Configuration

Create `secrets.yaml` in your ESPHome directory:

```yaml
wifi_ssid: "YourSSID"
wifi_password: "YourPassword"
ota_password: "YourOTAPassword"
api_key: "YourAPIEncryptionKey"
snapserver_ip: "192.168.1.x"
```
