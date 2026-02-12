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

### 1. Choose Your Configuration

**With Software EQ (18 bands):**
```yaml
packages:
  board: !include boards/sonocotta/amped-esp32-s3.yaml
  soft_eq: !include boards/software/soft-eq-18band.yaml
# Add wifi, api, snapclient...
```

**Minimal (no EQ):**
```yaml
packages:
  board: !include boards/sonocotta/amped-esp32-s3.yaml
# Add wifi, api, snapclient...
```

**Hardware EQ (Louder boards):**
```yaml
packages:
  board: !include boards/sonocotta/louder-esp32-s3.yaml
# Hardware EQ built-in! Add number entities for control
```

### 2. Flash from Home Assistant
1. Copy example YAML from `examples/` directory
2. Edit `secrets.yaml` with your wifi and snapserver IP
3. Flash via ESPHome dashboard → Done!

**→ See [examples/](examples/) for complete ready-to-flash configs**

---

## 📦 What's Included

### Board Profiles (`boards/sonocotta/`)
Hardware-only packages for [Sonocotta ESP32 Audio Dock](https://github.com/sonocotta/esp32-audio-dock) series:

| Board Family | DAC/Amp | Output | EQ Type | Profiles |
|--------------|---------|--------|---------|----------|
| **Amped-ESP32** | PCM5100+TPA3128 | 22W @ 8Ω | Software | esp32, esp32-s3, tpa3110 |
| **Louder-ESP32** | TAS5805M | 25W @ 8Ω | Hardware 15-band | esp32, esp32-s3, mic |
| **HiFi-ESP32** | PCM5100A | Line 2.1V | Software | esp32, esp32-s3, s3-mic |
| **Loud-ESP32** | MAX98357 | 3-5W | Software | esp32, esp32-s3 |

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
