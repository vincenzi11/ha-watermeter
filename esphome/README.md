# ESPHome Configurations (v5)

This folder contains the current ESPHome configurations for reading Diehl IZAR RC 868 water meters via wM-Bus using CC1101 or SX1262/SX1276 radio modules.

All configurations use the **esp-idf** framework and require **ESPHome >= 2026.6.4**.

---

## Devices Overview

| # | Config | Board | Radio | Features |
|---|--------|-------|-------|----------|
| 1 | [esp32-devkit-v4-wasserzahler.yaml](esp32-devkit-v4-wasserzahler.yaml) | ESP32 DevKit V4 | CC1101 | Water meter, full statistics |
| 2 | [atom-watermeter.yaml](atom-watermeter.yaml) | M5Stack ATOM Lite (ESP32-PICO) | CC1101 | Water meter, RGB LED status, full statistics |
| 3 | [heltec-watermeter.yaml](heltec-watermeter.yaml) | Heltec WiFi LoRa 32 V2 | SX1276 | Water meter, OLED display, full statistics |
| 4 | [wm5-heltec-esp32S3-v4.yaml](wm5-heltec-esp32S3-v4.yaml) | Heltec ESP32-S3 LoRa V4 | SX1262 | Water + gas meter, OLED display, DS18B20 temp, pressure sensor |

> **Runtime Meter ID Change:** The [`test/esp32-devkit-v4-wasserzahler.yaml`](test/esp32-devkit-v4-wasserzahler.yaml) variant supports changing the meter ID at runtime without recompiling. The ID is stored in NVS flash and can be set via curl, the web dashboard, or a Home Assistant service call. See [Changing the Meter ID at Runtime](#changing-the-meter-id-at-runtime-no-recompile) below.

---

## 1. ESP32 DevKit V4 + CC1101

**Board:** AZ-Delivery DevKit V4 (ESP32 240MHz, 520KB RAM, 4MB Flash)  
**Radio:** CC1101 868.95 MHz  
**Hostname:** `water-meter-esp`

### Wiring

```
                          ╭―――――――――――――――――――――――╮
                    GPIO6 | [ ] O  | USB |  O [ ] | 5V
                          |        -------        |
                          |   az-delivery-devkit  |
                          |         -v4           |
                          |                       |
  GDO0  ■ <---- GPIO16    | [■]               [ ] |
  GDO2  ■ <---- GPIO17    | [■]               [ ] |
                GPIO05    | [ ]  ___________  [ ] |
  CLK   ■ <---- GPIO18    | [■] |           | [ ] |
  MISO  ■ <---- GPIO19    | [■] |           | [ ] |
                   GND    | [ ] |           | [ ] |
  CSN   ■ <---- GPIO21    | [■] |           | [ ] |
                          |     |           |     |
  MOSI  ■ <---- GPIO23    | [■] |           | [ ] |
  GND   ■ <----    GND    | [■] |___________| [■] | 3.3V ---> ■ VCC
                          ╰―――――――――――――――――――――――╯
```

| CC1101 Pin | ESP32 GPIO | Function |
|------------|-----------|----------|
| CSN | GPIO21 | Chip Select |
| GDO0 | GPIO16 | IRQ (RX Clock) |
| GDO2 | GPIO17 | TX Status |
| MISO (SO) | GPIO19 | SPI Data In |
| CLK (SCK) | GPIO18 | SPI Clock |
| MOSI (SI) | GPIO23 | SPI Data Out |
| GND | GND | Ground |
| VCC | 3.3V | Power |

---

## 2. M5Stack ATOM Lite + CC1101 (EBYTE TI)

**Board:** M5Stack ATOM Lite S3 (ESP32-PICO-D4, 240MHz, 4MB Flash)  
**Radio:** EBYTE TI CC1101 868 MHz  
**Hostname:** `atom-watermeter`

### Wiring

```
                        ╭―――――――――――――――――――――――――――――――――╮
                        │         ╭―――――――――――――――╮       │
                        │         │ ATOM LITE  S3 │       │
                        │         ╰―――――――――――――――╯       │
 VCC  (red)    ■ <-3.3V │ [■]                             │
 CSN  (violet) ■ <-GP22 │ [■]                    SCL  [ ] │
 MOSI (orange) ■ <-GP19 │ [■]                    SDA  [ ] │
 CLK  (brown)  ■ <-GP23 │ [■]  ╭―――――――――――――――╮  5V  [ ] │
 MISO (green)  ■ <-GP33 │ [■]  │               │  GND [■] │ ■ GND (black)
                        │      │     USB       │          │
                        ╰―――――――――――――――――――――――――――――――――╯
                               │    PORT.A     │
                               ╰―――――――――――――――╯
                                  [■] GND
                                    [■] 5V
                                      [■] GPIO26 (GDO0, yellow)
                                        [■] GPIO32 (GDO2, blue)
```

| CC1101 Pin | ESP32 GPIO | Wire Color |
|------------|-----------|------------|
| CSN | GPIO22 | violet |
| CLK (SCK) | GPIO23 | brown |
| MOSI (SI) | GPIO19 | orange |
| MISO (SO) | GPIO33 | green |
| GDO0 (IRQ) | GPIO26 | yellow |
| GDO2 | GPIO32 | blue |
| VCC | 3.3V | red |
| GND | GND | black |

---

## 3. Heltec WiFi LoRa 32 V2 + SX1276

**Board:** Heltec WiFi LoRa 32 V2 (ESP32, 8MB Flash, built-in OLED + SX1276)  
**Radio:** SX1276 (onboard LoRa chip, repurposed for wM-Bus)  
**Hostname:** `wm5-izar-test`

### Wiring

No external radio module needed. The onboard SX1276 LoRa chip is used directly for wM-Bus reception at 868 MHz.

| Function | GPIO | Notes |
|----------|------|-------|
| SPI CLK | GPIO5 | To SX1276 (onboard) |
| SPI MOSI | GPIO27 | To SX1276 (onboard) |
| SPI MISO | GPIO19 | To SX1276 (onboard) |
| CS (NSS) | GPIO18 | SX1276 chip select |
| RESET | GPIO14 | SX1276 reset |
| IRQ (DIO0) | GPIO35 | SX1276 interrupt |
| OLED SDA | GPIO4 | I2C display |
| OLED SCL | GPIO15 | I2C display |
| Status LED | GPIO25 | Blink on frame received |
| Vext Control | GPIO21 | External power control |
| OLED Reset | GPIO16 | Display reset |

---

## 4. Heltec ESP32-S3 LoRa V4 + SX1262

**Board:** Heltec ESP32-S3 LoRa V4 (ESP32-S3, 2MB PSRAM, 16MB Flash, OLED SSD1315, LoRa SX1262)  
**Radio:** SX1262 (onboard, repurposed for wM-Bus)  
**Hostname:** `wm5-heltec-izar`  
**Extra Sensors:** Gas meter (pulse), DS18B20 temperature, analog pressure sensor

### Wiring

The SX1262 radio is onboard. External connections are only needed for the additional sensors:

| Function | GPIO | Notes |
|----------|------|-------|
| SPI CLK | GPIO9 | To SX1262 (onboard) |
| SPI MOSI | GPIO10 | To SX1262 (onboard) |
| SPI MISO | GPIO11 | To SX1262 (onboard) |
| CS (NSS) | GPIO8 | SX1262 chip select |
| RESET | GPIO12 | SX1262 reset |
| IRQ (DIO1) | GPIO14 | SX1262 interrupt |
| OLED SDA | GPIO17 | I2C display (SSD1315) |
| OLED SCL | GPIO18 | I2C display |
| OLED Reset | GPIO21 | Display reset |
| Dallas 1-Wire | GPIO4 | DS18B20 temperature sensor |
| Gas Pulse | GPIO5 | Reed contact / pulse input |
| Pressure ADC | GPIO1 | Analog pressure sensor |
| LoRa TX Enable | GPIO35 | SX1262 TX switch |
| LoRa RX Enable | GPIO36 | SX1262 RX switch |

---

## Sensors Provided

All configurations provide these water meter sensors to Home Assistant:

| Sensor | Unit | Description |
|--------|------|-------------|
| Water Display | m3 | Total meter reading |
| Water Current | L | Current flow since last telegram |
| Water Hour | L | Consumption this hour |
| Water Day | L | Consumption today |
| Water Yesterday | L | Consumption yesterday |
| Water Week | L | Consumption this week |
| Water Month | L | Consumption this month |
| Water Year | L | Consumption this year |
| Battery Life | years | Remaining IZAR battery |
| Update Interval | sec | Time between telegrams |
| RSSI | dBm | Radio signal strength |

The Heltec V4 configuration additionally provides gas meter readings (pulse-based) and water temperature/pressure.

---

## Getting Started

1. Copy the appropriate YAML file for your board
2. Create your `secrets.yaml` from the template:
   ```bash
   cp secrets.yaml.template secrets.yaml
   ```
3. Edit `secrets.yaml` and fill in your values (WiFi, meter ID, passwords)
4. Compile and flash:
   ```bash
   esphome run esp32-devkit-v4-wasserzahler.yaml
   ```
5. To find your meter ID, set `watermeterId` to `0x00000000` (wildcard) and check the logs

### secrets.yaml Reference

See [`secrets.yaml.template`](secrets.yaml.template) for all required keys. The most important ones:

| Key | Description |
|-----|-------------|
| `wifi_ssid` / `wifi_pswd` | Primary WiFi credentials |
| `ssid2_name` / `ssid2_pswd` | Fallback WiFi (optional) |
| `domain` | mDNS domain (usually `.local`) |
| `hotspot_pswd` | Password for fallback AP hotspot |
| `ota_pswd` | OTA update password |
| `hakey` | Home Assistant API encryption key (base64, 32 bytes) |
| `watermeterId` | Your meter's hex ID (e.g. `0x24058612`) |
| `local_sntp` | Local NTP server (optional) |

> **Generate an API key:**
> ```bash
> python3 -c "import secrets, base64; print(base64.b64encode(secrets.token_bytes(32)).decode())"
> ```

---

## Changing the Meter ID at Runtime (No Recompile!)

> **You do NOT need to recompile or reflash the firmware to switch to a different water meter.**
> The meter ID can be changed live via the web interface, curl, or a Home Assistant service call.

### Method 1: curl (recommended)

```bash
# Set new meter ID (example: 24058612)
curl -X POST "http://water-meter-esp.local/text/wasseruhr_meter_id_eingabe/set" --data "value=24058612"

# Verify current ID
curl "http://water-meter-esp.local/text_sensor/wasseruhr_meter_id"
```

The device restarts automatically after 2 seconds and uses the new ID.

### Method 2: Web Dashboard

1. Open `http://<hostname>.local` in your browser
2. Find the input field **"Wasseruhr Meter ID Eingabe"**
3. Enter the 8-digit hex ID (e.g. `24058612`)
4. Press Enter — the ESP restarts with the new ID

### Method 3: Home Assistant Service

```yaml
service: esphome.water_meter_esp_set_meter_id
data:
  new_meter_id: "24058612"
```

### Scan Mode (find all meters in range)

Set the meter ID to `00000000` to receive telegrams from **all** meters in range. Check the logs for `[wmbus_scan]` entries to find your meter's ID.

```bash
curl -X POST "http://water-meter-esp.local/text/wasseruhr_meter_id_eingabe/set" --data "value=00000000"
```

---

## Links

- [SzczepanLeon ESPHome wMBus Components](https://github.com/SzczepanLeon/esphome-components)
- [HenrikBurton wMBus5 Fork (SX1262 support)](https://github.com/HenrikBurton/esphome-components-wmbus5)
- [wmbusmeters Telegram Decoder](https://wmbusmeters.org/)
- [SmartRC-CC1101-Driver-Lib](https://github.com/LSatan/SmartRC-CC1101-Driver-Lib)
