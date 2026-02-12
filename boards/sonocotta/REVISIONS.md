# Hardware Revision Identification Guide

This guide helps you identify which hardware revision you have and which board profile to use.

## 🔍 Where to Find Your Revision

1. **Check the PCB silkscreen** - Look for text like "Rev H", "Rev G2", "Rev K0", etc.
2. **Look at the board marking** - Usually near the USB port or center of the board
3. **Check your order confirmation** - Tindie/Elecrow orders often list the revision
4. **Use visual identification** - See component differences below

## 📌 Amped-ESP32 Revisions

### Quick Identification

| Revision | Amplifier IC | LED Pin | Profile to Use |
|----------|-------------|---------|----------------|
| **Rev G, G1, G2** | TPA3110 | GPIO4/12 | `amped-esp32-tpa3110.yaml` |
| **Rev H, H1** | TPA3128 | GPIO21 | `amped-esp32.yaml` |
| **Rev J** (S3) | TPA3128 | GPIO21 | `amped-esp32-s3.yaml` |

### Visual Differences

**TPA3110 vs TPA3128:**
- **TPA3110** (Rev G): 28-pin HTSSOP package, NO proper MUTE pin
- **TPA3128** (Rev H+): 32-pin HTSSOP package, HAS MUTE pin on GPIO13/17

**Key Feature:** If your board has a working mute/enable control, it's Rev H+ with TPA3128.

## 📌 Louder-ESP32 Revisions

### Quick Identification

| Revision | ESP Chip | Mic Header | Profile to Use |
|----------|----------|------------|----------------|
| **Rev H - H5** | ESP32 | ❌ No | `louder-esp32.yaml` |
| **Rev H6** | ESP32 | ✅ Yes | `louder-esp32-mic.yaml` |
| **Rev J - J4** | ESP32-S3 | ❌ No | `louder-esp32-s3.yaml` |
| **Rev K0, K1** | ESP32-S3 | ✅ Yes | `louder-esp32-s3-mic.yaml` |

### Visual Differences

**Mic Header:**
- **3-pin header** labeled "MIC" or "I2S_MIC" near the edge
- Present on Rev H6+ (ESP32) and Rev K0+ (ESP32-S3)
- Pins: WS, BCK, DATA

**LED Position Changes:**
- Rev H-H2: GPIO9
- Rev H3-H5: GPIO21
- Rev H6+: GPIO27

## 📌 HiFi-ESP32 Revisions

### Quick Identification

| Revision | ESP Chip | Mic Header | DAC | Profile to Use |
|----------|----------|------------|-----|----------------|
| **Rev G, G1** | ESP32 | ❌ | PCM5100A | `hifi-esp32.yaml` |
| **Rev G** | ESP32-S3 | ❌ | PCM5100A | `hifi-esp32-s3.yaml` |
| **Rev G2+** | ESP32-S3 | ✅ | PCM5100A | `hifi-esp32-s3-mic.yaml` |
| **Plus models** | ESP32/S3 | Varies | PCM5122 | `hifi-esp32-plus.yaml` 🚧 |

### Visual Differences

**PCM5100A vs PCM5122:**
- **PCM5100A:** Simpler, no I2C connection needed
- **PCM5122:** Has I2C connection (SDA/SCL pins connected to ESP)
- **Plus models** have visibly more components near the DAC

**Mic Header (Rev G2+):**
- 3-pin header near ESP32-S3 module
- Only on S3 variants, Rev G2 and later

## 📌 Loud-ESP32 Revisions

### Quick Identification

All Loud-ESP32 boards are similar - main difference is ESP chip:
- **ESP32 variant** → Use `loud-esp32.yaml`
- **ESP32-S3 variant** → Use `loud-esp32-s3.yaml`

**Visual markers:**
- Two **MAX98357A** IC chips (stereo configuration)
- No external amplifier or I2C needed
- Simple design with fewer components than Louder

## 🔧 When You're Unsure

### Test Method 1: Check LED Pin

Flash a simple ESPHome config and try different LED pins:

```yaml
light:
  - platform: esp32_rmt_led_strip
    pin: GPIO4   # Try: GPIO4, GPIO9, GPIO12, GPIO21, GPIO27
    num_leds: 1
    chipset: ws2812
```

Whichever pin makes the LED light up is your LED pin.

### Test Method 2: Check Enable Pin (Amped boards)

For Amped boards, test which enable pin works:

```yaml
switch:
  - platform: gpio
    pin: GPIO13  # Try GPIO13 for ESP32, GPIO17 for S3
```

- **Works:** Your board is Rev H+ (TPA3128)
- **Doesn't work:** Your board is Rev G (TPA3110)

### Test Method 3: Check for I2C (Louder/Plus boards)

```yaml
i2c:
  sda: GPIO16  # ESP32: 16, ESP32-S3: 8
  scl: GPIO1   # ESP32: 1, ESP32-S3: 9
  scan: true
```

Check logs for detected I2C devices:
- **0x2D detected:** TAS5805M (Louder board)
- **0x4C detected:** PCM5122 (Plus board)
- **Nothing:** Regular HiFi/Amped/Loud board

## 📝 Revision History at a Glance

### Major Breaking Changes (Different Profiles Needed)

| Change | Affects | Old → New |
|--------|---------|-----------|
| **TPA3110 → TPA3128** | Amped-ESP32 | Rev G → Rev H |
| **Added Mic Header** | Louder-ESP32 | Rev H5 → Rev H6 |
| **Added Mic Header** | Louder-S3 | Rev J4 → Rev K0 |
| **Added Mic Header** | HiFi-S3 | Rev G1 → Rev G2 |
| **PCM5100A → PCM5122** | HiFi-Plus | Regular → Plus |

### Minor Changes (Same Profile, Different Substitutions)

| Change | Typical Revision Range | What Varies |
|--------|----------------------|-------------|
| **LED position** | All boards, all revisions | GPIO4 → GPIO12 → GPIO21 → GPIO27 |
| **IR position** | All boards, all revisions | GPIO32 → GPIO35 → GPIO19 (ESP32) |
| **OLED pins** | Rev J3+ | Different CS/DC/RST pins |
| **Power connector** | Various | Thicker pins, better retention |

## 🆘 Still Can't Identify?

1. **Post a photo** in [c-MM discussions](https://github.com/farmed-switch/c-MM-esphome-snapclient/discussions)
2. **Check sonocotta's repo** - Hardware folder has KiCad files: https://github.com/sonocotta/esp32-audio-dock/tree/main/hardware
3. **Email sonocotta** - Designer is very responsive: https://github.com/sonocotta

## 📚 Additional Resources

- **Official Hardware Repo:** https://github.com/sonocotta/esp32-audio-dock
- **Purchase Links:** [Tindie](https://www.tindie.com/stores/sonocotta/) | [Elecrow](https://www.elecrow.com/)
- **Pin Reference:** See [PINOUT.md](PINOUT.md)
- **Usage Examples:** See [README.md](README.md)

---

**Pro Tip:** Most users can start with the default profile for their board family and only customize if LED/IR doesn't work. The audio configuration (I2S pins) rarely changes between revisions!
