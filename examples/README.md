# ESPHome Snapclient Examples

Examples showing how to combine board packages with application logic.

## 📦 Package Structure

This repo uses **ESPHome packages** for modularity:

```
boards/
├── sonocotta/          # Hardware-specific board profiles
│   ├── amped-esp32-s3.yaml
│   ├── louder-esp32-s3.yaml
│   └── ...
└── software/           # Software features (board-agnostic)
    └── soft-eq-18band.yaml
```

## 🎯 Usage Patterns

### Pattern 1: Board + Software EQ + Snapclient
For boards **without hardware DSP** (Amped, HiFi, Loud):

```yaml
packages:
  board: !include boards/sonocotta/amped-esp32-s3.yaml
  soft_eq: !include boards/software/soft-eq-18band.yaml

# Then add: wifi, api, ota, snapclient
```

**Example:** [amped-esp32-s3-with-eq.yaml](amped-esp32-s3-with-eq.yaml)

**Requirements:**
- Must enable DSP in `esp32.framework.sdkconfig_options`:
  ```yaml
  CONFIG_SNAPCLIENT_DSP_FLOW_BASS_TREBLE_EQ: "n"
  CONFIG_USE_DSP_PROCESSOR: "y"
  CONFIG_DSP_OPTIMIZED: "y"
  CONFIG_DSP_MAX_FFT_SIZE: "4096"
  ```

**Features:**
- 18-band parametric EQ (40Hz-5kHz)
- CPU: ~18ms loop time @ 240MHz
- 6 presets + custom
- Full Home Assistant integration

---

### Pattern 2: Board Only + Snapclient (Minimal)
For lowest CPU usage, no EQ:

```yaml
packages:
  board: !include boards/sonocotta/amped-esp32-s3.yaml

# Then add: wifi, api, ota, snapclient (NO soft-eq package)
```

**Example:** [amped-esp32-s3-minimal.yaml](amped-esp32-s3-minimal.yaml)

**Use when:**
- You don't need equalization
- Want lowest latency
- Prefer clean audio path
- Minimal dependencies

---

### Pattern 3: Board with Hardware EQ
For boards **with TAS5805M** (Louder series):

```yaml
packages:
  board: !include boards/sonocotta/louder-esp32-s3.yaml

# Hardware EQ already included in board package!
# Add number entities for 15-band control
```

**Example:** [louder-esp32-s3-hardware-eq.yaml](louder-esp32-s3-hardware-eq.yaml)

**Features:**
- 15-band hardware EQ (20Hz-16kHz ISO frequencies)
- 0% CPU usage (DSP in TAS5805M chip)
- Better signal quality (EQ before D/A conversion)
- Fault monitoring built-in
- **DO NOT** include soft-eq package with Louder boards!

---

## 🚀 Quick Start

### 1. Choose Your Board
See [boards/sonocotta/README.md](../boards/sonocotta/README.md) for board selection guide.

Not sure which revision? Check [boards/sonocotta/REVISIONS.md](../boards/sonocotta/REVISIONS.md)

### 2. Pick an Example
- **With EQ:** `amped-esp32-s3-with-eq.yaml`
- **No EQ:** `amped-esp32-s3-minimal.yaml`  
- **Hardware EQ:** `louder-esp32-s3-hardware-eq.yaml`

### 3. Customize
```yaml
substitutions:
  name: your-snapclient-name
  friendly_name: "Your Friendly Name"

# Override board pins if needed (rare):
substitutions:
  led_pin: "GPIO27"  # Your revision uses different LED pin
```

### 4. Flash from Home Assistant
1. Copy example to `esphome/` directory in HA
2. Edit `!secret` values in `secrets.yaml`
3. Flash via ESPHome dashboard

---

## 🔧 Configuration Secrets

Create `secrets.yaml` in ESPHome directory:

```yaml
# WiFi
wifi_ssid: "YourSSID"
wifi_password: "YourPassword"

# Snapserver
snapserver_ip: "192.168.1.10"

# Security
ota_password: "your-ota-password"
api_key: !secret your_api_key  # Generate in HA
```

---

## 📊 Comparison: Software vs Hardware EQ

| Feature | Software EQ | Hardware EQ (TAS5805M) |
|---------|-------------|------------------------|
| **Bands** | 18 (40Hz-5kHz) | 15 (20Hz-16kHz) |
| **CPU Usage** | ~18ms loop time | 0% (chip DSP) |
| **Latency** | +minimal | None |
| **Signal Quality** | Good (post-DAC) | Better (pre-DAC) |
| **Range** | ±15dB | ±14dB |
| **Boards** | Amped, HiFi, Loud | Louder only |
| **Cost** | Free | ~$10 more |

---

## 🎚️ EQ Usage Tips

### Software EQ (18-band)
**Best for:**
- Bass-heavy use cases (subwoofers)
- Frequency range: 40Hz-5kHz optimized
- Budget builds (Amped/HiFi boards)

**Presets included:**
- Flat (Bypass)
- Subwoofer Boost
- Bookshelf Speakers
- Floor Standing
- Near Field Monitors
- Small/Portable

### Hardware EQ (15-band TAS5805M)
**Best for:**
- Full-range speakers
- Professional applications
- Lowest possible latency
- ISO standard frequencies (20Hz-16kHz)

---

## 📂 Files in This Directory

| File | Description | EQ Type | Connectivity |
|------|-------------|---------|--------------|
| `amped-esp32-s3-with-eq.yaml` | Amped S3 + Software EQ | 18-band soft | WiFi |
| `amped-esp32-s3-minimal.yaml` | Amped S3 minimal (no EQ) | None | WiFi |
| `louder-esp32-s3-hardware-eq.yaml` | Louder S3 + Hardware EQ | 15-band TAS5805M | Ethernet |

---

## 🔍 Need Help?

1. **Board selection:** [boards/sonocotta/README.md](../boards/sonocotta/README.md)
2. **Hardware revision:** [boards/sonocotta/REVISIONS.md](../boards/sonocotta/REVISIONS.md)
3. **Pinout reference:** [boards/sonocotta/PINOUT.md](../boards/sonocotta/PINOUT.md)
4. **Issues:** [GitHub Issues](https://github.com/farmed-switch/c-MM-esphome-snapclient/issues)

---

**Pro Tip:** Start with minimal example to verify hardware, then add soft EQ if needed!
