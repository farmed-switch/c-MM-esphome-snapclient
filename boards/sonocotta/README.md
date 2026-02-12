# Sonocotta ESP32 Audio Boards - ESPHome Board Packages

Flexibla hardware packages för [Sonocotta ESP32 Audio Dock](https://github.com/sonocotta/esp32-audio-dock) serie.  
Designade av Artem Sokolov (sonocotta) - professionella ESP32 audio boards.

**💡 Designed for:** [c-MM-esphome-snapclient](https://github.com/farmed-switch/c-MM-esphome-snapclient) - Flash from ESPHome (standalone or via Home Assistant).

## 🎵 Equalizer Options

**ALL boards** can use software EQ (18-band graphic EQ processed on ESP32):
- ✅ **HiFi-ESP32** (all variants) - Soft EQ available
- ✅ **Amped-ESP32** (all variants) - Soft EQ available  
- ✅ **Loud-ESP32** (all variants) - Soft EQ available
- ✅ **Louder-ESP32** (all variants) - **Hardware 15-band EQ** (TAS5805M) **OR** soft EQ

💡 **Hardware EQ (Louder boards):**
- Processed in DAC chip before D/A conversion (better quality)
- Zero ESP32 CPU usage
- 15 ISO bands (20Hz-16kHz)

💡 **Software EQ (all boards):**
- 18 bands (11 bass: 40-140Hz, 7 higher: 200-5000Hz)
- ~18ms processing time
- 7 presets included (Subwoofer, Bookshelf, etc.)

📖 See [`examples/`](examples/) folder for complete configuration examples.

## 📋 Board Overview

| Board | DAC | Output | Power | DSP/EQ | Profiles Available |
|-------|-----|--------|-------|--------|-------------------|
| **HiFi-ESP32** | PCM5100A | Line 2.1V | - | ❌ | esp32, esp32-s3, s3-mic, esp32-plus, s3-plus |
| **Loud-ESP32** | MAX98357 | 3-5W @ 4Ω | USB-C | ❌ | esp32, esp32-s3 |
| **Louder-ESP32** | TAS5805M | 25W @ 8Ω | 26V | ✅ 15-band | esp32, esp32-mic, s3, s3-mic |
| **Louder-ESP32-Plus** | TAS5825M | 25W @ 8Ω | 26V | ✅ 15-band | **s3-plus** (NEW!) |
| **Amped-ESP32** | PCM5100+TPA | 22W @ 8Ω | 26V | ❌ | esp32, tpa3110, esp32-s3 |
| **Amped-ESP32-S3-Plus** | PCM5122+TPA | 22W @ 8Ω | 26V | ❌ | **s3-plus** (NEW!) |

🚧 = Component under development by sonocotta

## 🔍 Which Profile Should I Use?

### Quick Selection Guide

**Amped-ESP32:**
- ✅ `amped-esp32.yaml` - Rev H/H1/J (TPA3128 with MUTE pin)
- ✅ `amped-esp32-tpa3110.yaml` - Rev G/G1/G2 (older TPA3110 amp)
- ✅ `amped-esp32-s3.yaml` - All S3 revisions (Rev J+)
- ✅ `amped-esp32-s3-plus.yaml` - **NEW!** ESP32-S3 Plus (PCM5122 DAC, OLED, RGB LED, IR receiver)

**Louder-ESP32:**
- ✅ `louder-esp32.yaml` - Rev H through H5
- ✅ `louder-esp32-mic.yaml` - Rev H6 (with mic header)
- ✅ `louder-esp32-s3.yaml` - Rev J through J4
- ✅ `louder-esp32-s3-mic.yaml` - Rev K0/K1 (with mic header)
- ✅ `louder-esp32-s3-plus.yaml` - **NEW!** ESP32-S3 with TAS5825M DAC (OLED, RGB, IR, Ethernet)

**HiFi-ESP32:**
- ✅ `hifi-esp32.yaml` - All ESP32 revisions (PCM5100A)
- ✅ `hifi-esp32-s3.yaml` - Rev G/G1 (PCM5100A)
- ✅ `hifi-esp32-s3-mic.yaml` - Rev G2+ (with mic header)
- ✅ `hifi-esp32-plus.yaml` - ESP32 with PCM5122
- ✅ `hifi-esp32-s3-plus.yaml` - ESP32-S3 with PCM5122

**Loud-ESP32:**
- ✅ `loud-esp32.yaml` - All ESP32 revisions
- ✅ `loud-esp32-s3.yaml` - All S3 revisions

💡 **Not sure which revision you have?** See [REVISIONS.md](REVISIONS.md) for identification tips.

## 📦 Usage Example

### Minimal Snapclient (Basic Setup)

```yaml
substitutions:
  name: livingroom-snapclient
  friendly_name: "Living Room Audio"

# Include board package
packages:
  board: !include boards/sonocotta/louder-esp32-s3.yaml

esphome:
  name: ${name}
  friendly_name: ${friendly_name}
  platformio_options:
    board_build.flash_mode: dio

# Network
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  power_save_mode: none  # CRITICAL for audio!

api:
ota:
  platform: esphome

# Snapclient
external_components:
  - source: github://farmed-switch/c-MM-esphome-snapclient@main
    components: [ snapclient ]

snapclient:
  hostname: 192.168.1.10  # Your snapserver IP
  port: 1704
  name: ${friendly_name}
  i2s_dout_pin: GPIO16  # GPIO16 for S3, GPIO25 for ESP32
  webserver_port: 8080

# Application logic (LED, switches, etc.)
switch:
  - platform: tas5805m  # For Louder boards with TAS5805M
    id: enable_dac
    name: "${friendly_name} Enable"

light:
  - platform: esp32_rmt_led_strip
    pin: ${led_pin}  # From board package
    num_leds: 1
    chipset: ws2812
```

### Med Software EQ (18-band)

```yaml
# ... (include same base config as above)

# Enable DSP processor
esp32:
  framework:
    sdkconfig_options:
      CONFIG_USE_DSP_PROCESSOR: "y"

# EQ Master Switch
switch:
  - platform: template
    name: "${friendly_name} EQ Enable"
    id: eq_enable
    restore_mode: RESTORE_DEFAULT_ON
    on_turn_on:
      - lambda: |-
          extern void enable_eq(bool enable);
          enable_eq(true);

# EQ Bands (18 number components)
number:
  - platform: template
    name: "${friendly_name} EQ 40Hz"
    min_value: -15
    max_value: 15
    step: 1
    mode: slider
    on_value:
      - lambda: |-
          extern void set_eq_band(int band, float gain_db);
          set_eq_band(0, x);
  # ... repeat for bands 1-17 (see examples/snapclient_with_soft_eq.yaml)

# EQ Presets
select:
  - platform: template
    name: "${friendly_name} EQ Preset"
    options:
      - "Flat (Bypass)"
      - "Subwoofer"
      - "Bookshelf"
      - "Floor Standing"
    on_value:
      - lambda: |-
          extern void apply_eq_preset(const char* preset);
      - lambda: |-
          extern void apply_eq_preset(const char* preset);
          apply_eq_preset(x.c_str());
```

📖 **Complete examples:** See [`examples/`](examples/) folder for full configurations.

## Pin Mappings

### ESP32 vs ESP32-S3 Differences

| Component | ESP32 | ESP32-S3 |
|-----------|-------|----------|
| **I2S BCK** | GPIO26 | GPIO14 |
| **I2S WS** | GPIO22 | GPIO15 |
| **I2S DOUT** | GPIO25 | GPIO16 |
| **I2C SDA** | GPIO16 | GPIO8/GPIO35 |
| **I2C SCL** | GPIO17/GPIO1 | GPIO9/GPIO36 |
| **LED** | GPIO4 | GPIO9/GPIO21 |
| **IR Receiver** | GPIO32 | GPIO7 |
| **task_stack_in_psram** | `"false"` | `"true"` |
| **PSRAM mode** | quad @ 40MHz | octal @ 80MHz |

### Board-Specific Pins

#### Amped-ESP32 / Amped-ESP32-S3
- **MUTE/ENABLE**: GPIO13 (ESP32) / GPIO17 (ESP32-S3)
- Simple PCM5100 DAC, no I2C needed

#### Louder-ESP32 / Louder-ESP32-S3
- **PWDN (Enable)**: GPIO21 (ESP32) / GPIO17 (ESP32-S3)
- **I2C**: Required for TAS5805M control
- **Fault monitoring**: Built-in via TAS5805M component

#### HiFi-ESP32 / HiFi-ESP32-S3
- Line-level output only
- No enable pin needed
- No I2C needed

#### Loud-ESP32 / Loud-ESP32-S3
- **ENABLE**: GPIO13 (ESP32) / GPIO8 (ESP32-S3)
- Simple MAX98357 DAC

## Critical Differences

### ESP32 vs ESP32-S3

**NEVER mix these settings!**

```yaml
# ESP32 (vanilla)
task_stack_in_psram: "false"
esp32:
  board: esp32dev
  # no variant
psram:
  mode: quad
  speed: 40MHz

# ESP32-S3
task_stack_in_psram: "true"
esp32:
  board: esp32-s3-devkitc-1
  variant: ESP32S3
psram:
  mode: octal
  speed: 80MHz
```

## 🎨 Customizing for Your Revision

All profiles use substitutions for pins that vary between revisions:

```yaml
# Example: Override LED pin for your specific revision
packages:
  board: !include boards/sonocotta/amped-esp32.yaml

substitutions:
  led_pin: "GPIO12"  # Your revision uses GPIO12 instead of default GPIO21
  ir_pin: "GPIO35"   # Your revision uses GPIO35 instead of default GPIO32
```

### Common Pin Variations

**LED pins by revision:**
- Early revisions: GPIO4
- Mid revisions: GPIO12 or GPIO21
- Late revisions: GPIO27

**IR receiver pins:**
- ESP32: Usually GPIO32, GPIO35, or GPIO19
- ESP32-S3: Usually GPIO7

**See [PINOUT.md](PINOUT.md) for complete pin reference per board.**

## ⚙️ Optional Features

All boards include:
- RGB LED (WS2812) - pin varies by revision
- IR receiver support
- SPI ethernet header (W5500)

### Ethernet Configuration

```yaml
ethernet:
  type: W5500
  clk_pin: GPIO12    # Same for ESP32/S3
  mosi_pin: GPIO11   # S3 only (ESP32: GPIO23)
  miso_pin: GPIO13
  cs_pin: GPIO10     # S3 only (ESP32: GPIO5)
  interrupt_pin: GPIO6  # S3 only (ESP32: GPIO35)
  reset_pin: GPIO5   # S3 only (ESP32: GPIO14)
```

## Credits

Board designs by [Artem Sokolov (sonocotta)](https://github.com/sonocotta)
- Hardware: https://github.com/sonocotta/esp32-audio-dock
- Purchase: [Tindie](https://www.tindie.com/stores/sonocotta/) | [Elecrow](https://www.elecrow.com/)

---

**Legend:**
- ✅ Available with profile
- 🚧 Coming soon
- ❌ Not applicable
