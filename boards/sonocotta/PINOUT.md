# Sonocotta ESP32 Audio Boards - Complete Pinout Reference

## 📍 Revision-Specific Pins

**IMPORTANT:** Some pins vary between hardware revisions. This table shows **default values** used in board profiles.

| Pin Type | ESP32 Default | ESP32-S3 Default | Variations by Revision |
|----------|---------------|------------------|------------------------|
| **LED (WS2812)** | GPIO4 | GPIO9 | Early: 4/9, Mid: 12/21, Late: 21/27 |
| **IR Receiver** | GPIO32 | GPIO7 | ESP32: 32/35/19, S3: Always 7 |
| **MUTE/Enable** | GPIO13 | GPIO17 | Only TPA3128 (Rev H+) |
| **TAS5805M PWDN** | GPIO21 | GPIO17 | Consistent across revisions |

💡 **Use substitutions in your config to override pins for your specific revision** - see [README.md](README.md#customizing-for-your-revision).

🔍 **Not sure which revision you have?** See [REVISIONS.md](REVISIONS.md) for identification guide.

## Quick Reference Table

| Board Family | Chip | I2S BCK | I2S WS | I2S DOUT | I2C SDA | I2C SCL | Enable/PWDN | LED | IR | Other |
|--------------|------|---------|--------|----------|---------|---------|-------------|-----|----|----|
| **HiFi-ESP32** | ESP32 | 26 | 22 | 25 | - | - | - | 4 | 32 | - |
| **HiFi-ESP32** | S3 | 14 | 16 | 15 | - | - | - | 9 | 7 | - |
| **Loud-ESP32** | ESP32 | 26 | 22 | 25 | - | - | 13 | 4 | 32 | - |
| **Loud-ESP32** | S3 | 14 | 16 | 15 | - | - | 8 | 9 | 7 | - |
| **Louder-ESP32** | ESP32 | 26 | 22 | 25 | 16 | 1 | 21 | 27 | 34 | FAULT:27, CLIP:34 |
| **Louder-ESP32** | S3 | 14 | 16 | 15 | 8 | 9 | 17 | 21 | 7 | FAULT:9, CLIP:18 |
| **Amped-ESP32** | ESP32 | 26 | 22 | 25 | - | - | 13 | 4 | 32 | - |
| **Amped-ESP32** | S3 | 14 | 16 | 15 | - | - | 17 | 9 | 7 | - |

## Detailed Pinouts

### HiFi-ESP32 / HiFi-ESP32-Plus

#### ESP32 Variant
```yaml
I2S:
  BCK:  GPIO26
  WS:   GPIO22
  DOUT: GPIO25

Optional:
  LED:  GPIO4   (WS2812)
  IR:   GPIO32  (TSOP)
  
Ethernet (W5500):
  MOSI: GPIO23
  CLK:  GPIO18
  MISO: GPIO19
  CS:   GPIO5
  INT:  GPIO35
  RST:  GPIO14
```

#### ESP32-S3 Variant
```yaml
I2S:
  BCK:  GPIO14
  WS:   GPIO16
  DOUT: GPIO15

I2C (Plus model only):
  SDA:  GPIO35
  SCL:  GPIO36

Optional:
  LED:  GPIO9   (WS2812)
  IR:   GPIO7   (TSOP)
  
Ethernet (W5500):
  MOSI: GPIO11
  CLK:  GPIO12
  MISO: GPIO13
  CS:   GPIO10
  INT:  GPIO6
  RST:  GPIO5
```

**Notes:**
- Plus models have PCM5122 DAC with DSP (requires I2C) 🚧
- Standard models have PCM5100A (no I2C needed)
- **Rev G2+ (S3 only)** adds mic header: WS=GPIO17, BCK=GPIO18, DATA=GPIO8
- LED pin varies: GPIO4 (early) → GPIO9 (mid) → GPIO21 (late)

---

### Loud-ESP32

#### ESP32 Variant
```yaml
I2S:
  BCK:  GPIO26
  WS:   GPIO22
  DOUT: GPIO25

DAC Control:
  EN:   GPIO13  (MAX98357 enable)

Optional:
  LED:  GPIO4   (WS2812)
  IR:   GPIO32  (TSOP)
  
Ethernet (W5500):
  MOSI: GPIO23
  CLK:  GPIO18
  MISO: GPIO19
  CS:   GPIO5
  INT:  GPIO35
  RST:  GPIO14
```

#### ESP32-S3 Variant
```yaml
I2S:
  BCK:  GPIO14
  WS:   GPIO16
  DOUT: GPIO15

DAC Control:
  EN:   GPIO8   (MAX98357 enable)

Optional:
  LED:  GPIO9   (WS2812)
  IR:   GPIO7   (TSOP)
  
Ethernet (W5500):
  MOSI: GPIO11
  CLK:  GPIO12
  MISO: GPIO13
  CS:   GPIO10
  INT:  GPIO6
  RST:  GPIO5
```

**Notes:**
- Dual MAX98357 DAC
- 3-5W per channel @ 8Ω
- Powered from USB-C (5V)

---

### Louder-ESP32 / Louder-ESP32-Plus

#### ESP32 Variant
```yaml
I2S:
  BCK:  GPIO26
  WS:   GPIO22
  DOUT: GPIO25

I2C (TAS5805M):
  SDA:  GPIO16
  SCL:  GPIO1

DAC Control:
  PWDN: GPIO21  (TAS5805M power down)
  FAULT:GPIO27  (Fault indicator)
  CLIP: GPIO34  (Clipping/OTW)

Optional:
  LED:  GPIO27  (WS2812)
  IR:   GPIO34  (TSOP)
  
Ethernet (W5500):
  MOSI: GPIO23
  CLK:  GPIO18
  MISO: GPIO19
  CS:   GPIO5
  INT:  GPIO35
  RST:  GPIO14
```

#### ESP32-S3 Variant
```yaml
I2S:
  BCK:  GPIO14
  WS:   GPIO16
  DOUT: GPIO15

I2C (TAS5805M):
  SDA:  GPIO8
  SCL:  GPIO9

DAC Control:
  PWDN: GPIO17  (TAS5805M power down)
  FAULT:GPIO9   (Fault indicator - shared with I2C)
  CLIP: GPIO18  (Clipping/OTW)

Optional:
  LED:  GPIO21  (WS2812)
  IR:   GPIO7   (TSOP)
  
Ethernet (W5500):
  MOSI: GPIO11
  CLK:  GPIO12
  MISO: GPIO13
  CS:   GPIO10
  INT:  GPIO6
  RST:  GPIO5
```

**Notes:**
- TAS5805M with 15-band EQ
- I2C required for DAC configuration
- 25W per channel @ 8Ω
- Fault monitoring built-in
- **Plus model** (TAS5825M) has advanced DSP 🚧
- **Rev H6 (ESP32)** adds mic header: WS=GPIO13, BCK=GPIO5, DATA=GPIO25
- **Rev K0+ (ESP32-S3)** adds mic header: WS=GPIO17, BCK=GPIO18, DATA=GPIO8
  - ⚠️ **WARNING:** S3 mic pins conflict with TAS5805M enable (GPIO17) and I2C SDA (GPIO8)
- LED pin varies: GPIO9 (H-H2) → GPIO21 (H3-H5) → GPIO27 (H6+)

---

### Amped-ESP32 / Amped-ESP32-Plus

#### ESP32 Variant (Rev H+)
```yaml
I2S:
  BCK:  GPIO26
  WS:   GPIO22
  DOUT: GPIO25

DAC Control:
  MUTE: GPIO13  (TPA3128 MUTE pin, Rev H+)

Optional:
  LED:  GPIO4   (WS2812)
  IR:   GPIO32  (TSOP)
  
Ethernet (W5500):
  MOSI: GPIO23
  CLK:  GPIO18
  MISO: GPIO19
  CS:   GPIO5
  INT:  GPIO35
  RST:  GPIO14
```

#### ESP32-S3 Variant (Rev J+)
```yaml
I2S:
  BCK:  GPIO14
  WS:   GPIO16
  DOUT: GPIO15

DAC Control:
  MUTE: GPIO17  (TPA3128 MUTE pin, Rev J+)

I2C (Plus model only):
  SDA:  GPIO35
  SCL:  GPIO36

Optional:
  LED:  GPIO9   (WS2812)
  IR:   GPIO7   (TSOP)
  
Ethernet (W5500):
  MOSI: GPIO11
  CLK:  GPIO12
  MISO: GPIO13
  CS:   GPIO10
  INT:  GPIO6
  RST:  GPIO5
```

**Notes:**
- PCM5100A DAC + TPA3128 amp (Rev H+ / J+)
- 22W @ 8Ω (stereo), 40W @ 4Ω (mono bridged)
- **Plus models** have PCM5122 with DSP (requires I2C) 🚧
- **Rev G/G1/G2** used TPA3110 (NO MUTE pin) - use `amped-esp32-tpa3110.yaml`
- **Rev H/H1/J** use TPA3128 (with MUTE on GPIO13/17) - use standard profiles
- LED pin varies: GPIO4 (Rev G) → GPIO12 (G1/G2) → GPIO21 (H+) → GPIO27 (late)
- IR pin varies: GPIO32 (default) → GPIO35 or GPIO19 on some revisions

---

## Microphone Header (Selected Models)

### HiFi-ESP32 (Rev G2+) / Louder-ESP32 (Rev H6+ / K0+)

#### ESP32 Variant
```yaml
I2S Microphone:
  BCK:  GPIO26  (shared with main I2S)
  WS:   GPIO25  (shared)
  DIN:  GPIO13  (dedicated mic data in)
```

#### ESP32-S3 Variant (Louder)
```yaml
I2S Microphone (dedicated bus):
  BCK:  GPIO41
  WS:   GPIO40
  DIN:  GPIO39
```

#### ESP32-S3 Variant (HiFi Rev G2+)
```yaml
I2S Microphone (dedicated bus):
  BCK:  GPIO17
  WS:   GPIO18
  DIN:  GPIO8
```

**Compatible with:** INMP441 MEMS microphone

---

## OLED Display (Optional)

All boards support 128x64 OLED via SPI (30-pin 0.5mm ribbon connector).

### ESP32 Variant
```yaml
SPI Display:
  MOSI: GPIO23  (shared with ethernet SPI)
  CLK:  GPIO18  (shared)
  MISO: GPIO19  (shared)
  CS:   GPIO15
  DC:   GPIO4
  RST:  GPIO32
  
Driver: SH1106 or SSD1306
```

### ESP32-S3 Variant
```yaml
SPI Display:
  MOSI: GPIO11  (shared with ethernet SPI)
  CLK:  GPIO12  (shared)
  MISO: GPIO13  (shared)
  CS:   GPIO47
  DC:   GPIO38
  RST:  GPIO48
  
Driver: SH1106 or SSD1306
```

---

## Power Supply Requirements

| Board | Min Voltage | Max Voltage | Connector | Notes |
|-------|-------------|-------------|-----------|-------|
| HiFi-ESP32 | 5V | 5V | USB-C | Line-level only |
| Loud-ESP32 | 5V | 5V | USB-C | Up to 2.5A |
| Louder-ESP32 | 5V | 26V | Barrel jack 5.5/2.5mm | 3A+ recommended |
| Amped-ESP32 | 5V | 26V | Barrel jack 5.5/2.5mm | 3A+ recommended |

**Power Calculation:**
- 10W @ 8Ω = ~9V @ 3A minimum
- 25W @ 8Ω = ~15V @ 3A recommended
- Efficiency: ~80% typical

---

## Revision History

### Amped-ESP32
- **Rev G and earlier:** TPA3110 amp (no MUTE pin)
- **Rev H+:** TPA3128 amp (true MUTE pin on GPIO13/17)

### Louder-ESP32
- **Rev H6+ (ESP32):** Added mic header
- **Rev K0+ (S3):** Added mic header

### HiFi-ESP32
- **Rev G2+ (S3):** Added mic header with dedicated I2S bus

---

## Sources

Official documentation: https://github.com/sonocotta/esp32-audio-dock
Hardware files: https://github.com/sonocotta/esp32-audio-dock/tree/main/hardware
