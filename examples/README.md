# ESPHome Snapclient Examples

Examples showing how to combine board packages with application logic.

## 📦 Package Structure

This repo uses **ESPHome packages** for modularity:

```
boards/
├── sonocotta/          # Hardware-specific board profiles
│   ├── amped-esp32-s3.yaml
│   ├── amped-esp32-s3-plus.yaml 🆕
│   ├── louder-esp32-s3.yaml
│   ├── louder-esp32-s3-plus.yaml 🆕
│   └── ...
└── software/           # Software features (board-agnostic)
    └── soft-eq-18band.yaml
```

## 🚀 Quick Start

### **NEW**: [amped-esp32-s3-plus-example.yaml](amped-esp32-s3-plus-example.yaml)
Complete example using **remote GitHub packages** with:
- Amped-ESP32-S3-Plus board profile
- 18-band software EQ
- USB Serial for ESP32-S3
- All monitoring sensors
- secrets.yaml integration

**Perfect for:** ESPHome in Home Assistant without cloning repo!

## 🎯 Usage Patterns

### Pattern 1: Board + Software EQ + Snapclient
For boards **without hardware DSP** (Amped, HiFi, Loud):

**Local includes:**
```yaml
packages:
  board: !include boards/sonocotta/amped-esp32-s3-plus.yaml
  soft_eq: !include boards/software/soft-eq-18band.yaml

# Then add: wifi, api, ota, snapclient
```

**Remote GitHub packages:**
```yaml
packages:
  - url: https://github.com/farmed-switch/c-MM-esphome-snapclient
    ref: board-profiles
    files: [boards/sonocotta/amped-esp32-s3-plus.yaml]
    refresh: 0s
  - url: https://github.com/farmed-switch/c-MM-esphome-snapclient
    ref: board-profiles
    files: [boards/software/soft-eq-18band.yaml]
    refresh: 0s
```

**Example:** [amped-esp32-s3-plus-example.yaml](amped-esp32-s3-plus-example.yaml)

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

Create `secrets.yaml` in ESPHome directory (see [secrets.yaml.example](secrets.yaml.example)):

```yaml
# WiFi
wifi_ssid: "YourSSID"
wifi_password: "YourPassword"

# Snapserver
snapserver_host: "192.168.1.10"  # IP or hostname

# Security
ota_password: "your-ota-password"
api_encryption_key: "your-32-char-api-key"
```

---

## ⚠️ ESP32-S3 Important Notes

**ALL ESP32-S3 boards MUST include USB Serial configuration for debug output:**

```yaml
logger:
  level: DEBUG
  hardware_uart: USB_SERIAL_JTAG  # REQUIRED for ESP32-S3

esp32:
  framework:
    type: esp-idf
    sdkconfig_options:
      CONFIG_ESP_CONSOLE_USB_SERIAL_JTAG: "y"
      CONFIG_ESP_CONSOLE_SECONDARY_NONE: "y"
      # ... other settings
```

**And always set:**
```yaml
substitutions:
  task_stack_in_psram: "true"  # REQUIRED for ESP32-S3
```

**Without these settings:**
- ❌ No serial output visible
- ❌ Debugging impossible
- ❌ Boot issues may occur

**This applies to:**
- amped-esp32-s3.yaml
- amped-esp32-s3-plus.yaml 🆕
- louder-esp32-s3.yaml
- louder-esp32-s3-plus.yaml 🆕
- hifi-esp32-s3.yaml
- hifi-esp32-s3-plus.yaml
- loud-esp32-s3.yaml

---

## 🎛️ Optional Hardware Features (Plus Models)

**Plus models** include additional hardware that can be activated:
- **Amped-ESP32-S3-Plus**: PCM5122 DAC + RGB LED + IR + OLED + Ethernet
- **Louder-ESP32-S3-Plus**: TAS5825M DAC + RGB LED + IR + OLED + Ethernet
- **HiFi-ESP32-S3-Plus**: PCM5122 DAC + RGB LED + IR + OLED

### Activating RGB LED (GPIO21)

Add to your configuration:
```yaml
light:
  - platform: esp32_rmt_led_strip
    id: rgb_front_led
    name: "${friendly_name} LED"
    rgb_order: GRB
    pin: GPIO21
    num_leds: 1
    chipset: ws2812
    effects:
      - pulse:
      - strobe:
```

**Use cases:**
- Visual status indicator (playing, paused, error)
- Audio reactive LED (sync with music)
- Home Assistant integration for notifications

### Activating IR Receiver (GPIO07)

Add to your configuration:
```yaml
remote_receiver:
  pin:
    number: GPIO07
    inverted: true
    mode:
      input: true
      pullup: true
  dump: all  # Shows all received codes in log

# React to specific remote buttons
binary_sensor:
  - platform: remote_receiver
    name: "${friendly_name} IR Play Button"
    nec:  # or rc_switch, lg, sony, etc.
      address: 0x00FF
      command: 0x00FF  # Your remote's code
    on_press:
      - homeassistant.service:
          service: media_player.media_play_pause
```

**Setup:**
1. Enable `dump: all`
2. Press remote buttons
3. Check logs for codes (e.g., `address: 0x00FF, command: 0x00FF`)
4. Replace in configuration
5. Remove `dump: all` after setup

### Activating OLED Display (SPI - SSD1306 128x64)

Add to your configuration:
```yaml
spi:
  clk_pin: GPIO12
  mosi_pin: GPIO11
  miso_pin: GPIO13

display:
  - platform: ssd1306_spi
    model: "SSD1306 128x64"
    cs_pin: GPIO47
    dc_pin: GPIO38
    reset_pin: GPIO48
    rotation: 0
    lambda: |-
      it.printf(0, 0, id(font), "${friendly_name}");
      it.printf(0, 16, id(font), "IP: %s", id(wifi_ip).state.c_str());
      it.printf(0, 32, id(font), "WiFi: %.0fdBm", id(wifi_signal_db).state);
      it.printf(0, 48, id(font), "Uptime: %.0f min", id(uptime_sec).state / 60);

font:
  - file: "fonts/arial.ttf"
    id: font
    size: 12
```

**Display info:**
- IP address, WiFi signal
- Playback status, volume
- System stats (CPU temp, heap, uptime)
- Custom messages from Home Assistant

### Activating Ethernet (W5500)

**⚠️ If using Ethernet, disable or remove WiFi section!**

Add to your configuration:
```yaml
spi:  # If not already defined for display
  clk_pin: GPIO12
  mosi_pin: GPIO11
  miso_pin: GPIO13

ethernet:
  type: W5500
  clk_pin: GPIO12   # Shared with SPI
  mosi_pin: GPIO11  # Shared with SPI
  miso_pin: GPIO13  # Shared with SPI
  cs_pin: GPIO10
  interrupt_pin: GPIO6
  reset_pin: GPIO5
  # Optional: Static IP
  # manual_ip:
  #   static_ip: 192.168.1.200
  #   gateway: 192.168.1.1
  #   subnet: 255.255.255.0
```

**Benefits:**
- More stable than WiFi (no packet loss)
- Lower latency for audio
- No wireless interference
- PoE possible with adapter

**🔗 Complete example with all features:** [amped-esp32-s3-plus-example.yaml](amped-esp32-s3-plus-example.yaml)

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
