# ESPHome Configurations (v5)

This folder contains the current ESPHome configurations for reading Diehl IZAR RC 868 water meters via wM-Bus using  SX1262 radio modules.

All configurations use the **esp-idf** framework and require **ESPHome >= 2026.6.4**.

---

## Devices Overview

| # | Config | Board | Radio | Features |
|---|--------|-------|-------|----------|
| 1 | [wm5-heltec-esp32s3-v4-FR-full.yaml](wm5-heltec-esp32s3-v4-FR-full.yaml) | ESP32 DevKit V4 | SX1262 | Water meter, full statistics |

> **Runtime Meter ID Change:** The [`wm5-heltec-esp32s3-v4-FR-full.yaml`](wm5-heltec-esp32s3-v4-FR-full.yaml) variant supports changing the meter ID at runtime without recompiling. The ID is stored in NVS flash and can be set via curl, the web dashboard, or a Home Assistant service call. See [Changing the Meter ID at Runtime](#changing-the-meter-id-at-runtime-no-recompile) below.



## 1. Heltec WiFi LoRa 32 V2 + SX1276

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
