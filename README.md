# esphome-snapclient

This is a temporary repo to build an esphome based snapclient for the 
esp32-louder platform from Andriy Malyshenko (@anabolyc)

It is just for easy build in the esphome web interface.

It is based on the works from:
 - @CarlosDerSeher [esp snapclient](https://github.com/CarlosDerSeher/snapclient)
 - @luar123 [esphome snapclient](https://github.com/esphome/esphome/pull/8350)
 - @mrtoy-me [tas5805m driver](https://github.com/mrtoy-me/esphome-tas5805m)
 - @anabolyc [esp32 audio hardware](https://github.com/sonocotta/esp32-audio-dock)

## Configuration Examples

### Amped ESP32-S3 with Software Graphic EQ (18 bands)

For generic I2S DAC boards with ESP32-S3 and 18-band software parametric EQ (40-5000Hz, ideal for subwoofers or full-range speakers):

```yaml
substitutions:
  name: snapcast-client
  friendly_name: Snapcast Client
  task_stack_in_psram: "true"  # MUST be true for ESP32-S3

esphome:
  name: ${name}
  friendly_name: ${friendly_name}
  min_version: 2025.7.0
  name_add_mac_suffix: false
  platformio_options:
    board_build.flash_mode: dio
  on_boot: 
    priority: 600
    then:
      - lambda: |-
          esp_netif_set_hostname(esp_netif_get_handle_from_ifkey("WIFI_STA_DEF"), "${name}");
      - delay: 30s  # Wait for NTP sync
      - switch.turn_on: enable_dac
      
esp32:
  board: esp32-s3-devkitc-1
  variant: ESP32S3
  flash_size: 8MB
  cpu_frequency: 240MHz
  framework:
    type: esp-idf
    version: recommended
    sdkconfig_options:
      CONFIG_BT_ALLOCATION_FROM_SPIRAM_FIRST: "y"
      CONFIG_BT_BLE_DYNAMIC_ENV_MEMORY: "y"
      CONFIG_MBEDTLS_EXTERNAL_MEM_ALLOC: "y"
      CONFIG_MBEDTLS_SSL_PROTO_TLS1_3: "y"
      CONFIG_SNAPCLIENT_DSP_FLOW_BASS_TREBLE_EQ: "n"
      CONFIG_USE_DSP_PROCESSOR: "y"
      CONFIG_DSP_OPTIMIZED: "y"

logger:
  level: DEBUG

debug:
  update_interval: 5s
  
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  power_save_mode: none  # Critical for audio performance
  output_power: 20dB

ota:
  - platform: esphome
    password: !secret ota_password

web_server:
  port: 80

api:

psram:
  mode: octal
  speed: 80MHz

time:
  - platform: sntp
    id: sntp_time

external_components:
  - source: github://farmed-switch/c-MM-esphome-snapclient@main
    components: [ snapclient ]
    refresh: 0s

i2s_audio:
  i2s_lrclk_pin: GPIO15
  i2s_bclk_pin: GPIO14

snapclient:
  hostname: !secret snapserver_ip
  port: 1704
  name: ${friendly_name}
  i2s_dout_pin: GPIO16

switch:
  - platform: gpio
    name: "Enable DAC"
    id: enable_dac
    pin: GPIO17
    restore_mode: ALWAYS_OFF
  
  - platform: template
    name: "EQ Enable"
    id: eq_enable
    optimistic: true
    restore_mode: RESTORE_DEFAULT_ON
    on_turn_on:
      - lambda: |-
          extern void enable_eq(bool enable);
          enable_eq(true);
    on_turn_off:
      - lambda: |-
          extern void enable_eq(bool enable);
          enable_eq(false);

sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
  - platform: internal_temperature
    name: "CPU Temperature"
  - platform: debug
    loop_time:
      name: "Loop Time"

# 18-band EQ: 40, 50, 60, 70, 80, 90, 100, 110, 120, 130, 140, 200, 315, 500, 800, 1250, 2000, 5000 Hz
number:
  - platform: template
    name: "EQ 0040Hz"
    id: eq_40hz
    min_value: -15
    max_value: 15
    step: 1
    initial_value: 0
    unit_of_measurement: "dB"
    restore_value: true
    on_value:
      - lambda: |-
          extern void set_eq_band(int band, float gain_db);
          set_eq_band(0, x);
  # ... repeat for all 18 bands with indices 0-17

select:
  - platform: template
    name: "EQ Preset"
    options:
      - "Flat (Bypass)"
      - "Flat (Full Range)"
      - "Subwoofer"
      - "Bookshelf"
      - "Floor Standing"
      - "Near Field"
      - "Small/Portable"
      - "Custom"
    initial_option: "Flat (Full Range)"
    on_value:
      - lambda: |-
          extern void apply_eq_preset(const char* preset);
          apply_eq_preset(x.c_str());
```

### Louder ESP32-S3 with TAS5805M Hardware EQ (15 bands)

For ESP32-S3 with TAS5805M professional DAC and hardware DSP (0% CPU, 20Hz-16kHz):

```yaml
substitutions:
  name: louder-esp32-s3
  friendly_name: Louder ESP32-S3
  task_stack_in_psram: "false"

esphome:
  name: ${name}
  friendly_name: ${friendly_name}
  min_version: 2025.7.5
  name_add_mac_suffix: false
  platformio_options:
    board_build.flash_mode: dio

esp32:
  board: esp32-s3-devkitc-1
  variant: ESP32S3
  flash_size: 16MB
  cpu_frequency: 240MHz
  framework:
    type: esp-idf
    version: recommended
    sdkconfig_options:
      CONFIG_ESP32S3_DATA_CACHE_64KB: "y"
      CONFIG_ESP32_S3_BOX_BOARD: "y"
      CONFIG_BT_ALLOCATION_FROM_SPIRAM_FIRST: "y"
      CONFIG_MBEDTLS_SSL_PROTO_TLS1_3: "y"

logger:
  level: DEBUG

api:
  encryption:
    key: !secret api_key

ota:
  - platform: esphome
    password: !secret ota_password

ethernet:
  type: W5500
  clk_pin: GPIO12
  mosi_pin: GPIO11
  miso_pin: GPIO13
  cs_pin: GPIO10
  interrupt_pin: GPIO06
  reset_pin: GPIO05
  clock_speed: 20MHz

external_components:
  - source: github://mrtoy-me/esphome-tas5805m@main
    components: [ tas5805m ]
    refresh: 0s
  - source: github://c-MM/esphome-snapclient@main
    components: [ snapclient ]
    refresh: 0s

psram:
  mode: octal
  speed: 80MHz

i2c:
  sda: GPIO8
  scl: GPIO9
  frequency: 400kHz

audio_dac:
  - platform: tas5805m
    id: tas5805m_dac
    enable_pin: GPIO17
    analog_gain: -5db
    refresh_eq: BY_SWITCH
    dac_mode: BTL
    mixer_mode: STEREO
    volume_max: -1dB
    volume_min: -60db

i2s_audio:
  i2s_lrclk_pin: GPIO15
  i2s_bclk_pin: GPIO14

snapclient:
  hostname: !secret snapserver_ip
  port: 1704
  name: ${friendly_name}
  audio_dac: tas5805m_dac
  i2s_dout_pin: GPIO16

switch:
  - platform: tas5805m
    enable_dac:
      name: "Enable DAC"
      restore_mode: ALWAYS_ON
    enable_eq:
      name: "Enable EQ"
      restore_mode: ALWAYS_OFF

# Hardware EQ: 20, 31.5, 50, 80, 125, 200, 315, 500, 800, 1.25k, 2k, 3.15k, 5k, 8k, 16k Hz
number:
  - platform: tas5805m
    eq_gain_band20Hz:
      name: "Gain 20Hz"
    eq_gain_band31.5Hz:
      name: "Gain 31.5Hz"
    eq_gain_band50Hz:
      name: "Gain 50Hz"
    eq_gain_band80Hz:
      name: "Gain 80Hz"
    eq_gain_band125Hz:
      name: "Gain 125Hz"
    eq_gain_band200Hz:
      name: "Gain 200Hz"
    eq_gain_band315Hz:
      name: "Gain 315Hz"
    eq_gain_band500Hz:
      name: "Gain 500Hz"
    eq_gain_band800Hz:
      name: "Gain 800Hz"
    eq_gain_band1250Hz:
      name: "Gain 1.25kHz"
    eq_gain_band2000Hz:
      name: "Gain 2kHz"
    eq_gain_band3150Hz:
      name: "Gain 3.15kHz"
    eq_gain_band5000Hz:
      name: "Gain 5kHz"
    eq_gain_band8000Hz:
      name: "Gain 8kHz"
    eq_gain_band16000Hz:
      name: "Gain 16kHz"

sensor:
  - platform: tas5805m
    faults_cleared:
      name: "Faults Cleared"

binary_sensor:
  - platform: tas5805m
    have_fault:
      name: "Any Faults"
    left_channel_dc_fault:
      name: "Left DC Fault"
    right_channel_dc_fault:
      name: "Right DC Fault"
```

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
