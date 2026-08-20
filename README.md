# Watermeter - Home Assistant Integration

[![License][license-shield]][license]
[![ESP32 Release](https://img.shields.io/github/v/release/zibous/ha-watermeter.svg?style=flat-square)](https://github.com/zibous/ha-watermeter/releases)
[![ESPHome release][esphome-release-shield]][esphome-release]
[![Support author][donate-me-shield]][donate-me]

[license-shield]: https://img.shields.io/static/v1?label=License&message=MIT&color=orange&logo=license
[license]: https://opensource.org/licenses/MIT

[esphome-release-shield]: https://img.shields.io/static/v1?label=ESPHome&message=2026.6.4&color=green&logo=esphome
[esphome-release]: https://GitHub.com/esphome/esphome/releases/

[donate-me-shield]: https://img.shields.io/static/v1?label=+&color=orange&message=Buy+me+a+coffee
[donate-me]: https://www.buymeacoff.ee/zibous

---

## About

Reading water meters equipped with **IZAR modules** (Diehl IZAR RC 868 I R4 PL) via wM-Bus 868 MHz and integrating them into Home Assistant using ESPHome.

![diehl_metering](./docs/diehl_metering.jpg)

---

## Current Version: v5 (ESPHome + ESP32 + CC1101)

The current version is based on **ESPHome** with **ESP32 boards** and the **CC1101 868 MHz transceiver**. It offers low resource consumption, stable radio reception, and easy OTA updates.

### Supported Boards (v5)

| Board | Configuration |
|-------|---------------|
| **ESP32 DevKit V4** + CC1101 | [`esphome/esp32-devkit-v4-wasserzahler.yaml`](esphome/esp32-devkit-v4-wasserzahler.yaml) |
| **M5Stack ATOM Lite** (ESP32-PICO) + CC1101 | [`esphome/atom-watermeter.yaml`](esphome/atom-watermeter.yaml) |
| **Heltec WiFi LoRa 32** + CC1101 | [`esphome/heltec-watermeter.yaml`](esphome/heltec-watermeter.yaml) |
| **Heltec ESP32-S3 V4** (LoRa, OLED) + CC1101 | [`esphome/wm5-heltec-esp32S3-v4.yaml`](esphome/wm5-heltec-esp32S3-v4.yaml) |

### Requirements

- Water meter with IZAR module (Diehl IZAR RC 868 I R4 PL)
- ESP32 board (see table above)
- CC1101 868 MHz transceiver module (e.g. [Fayme CC1101](https://amzn.eu/d/i5YwBkR) or [EBYTE TI CC1101](https://amzn.eu/d/7GPqsng))
- ESPHome >= 2026.6.4
- Home Assistant

### Quick Start

1. Install ESPHome (Docker or HA Add-on)
2. Create `secrets.yaml` with your WiFi credentials and meter ID
3. Flash the appropriate YAML configuration:
   ```bash
   esphome run esphome/esp32-devkit-v4-wasserzahler.yaml
   ```
4. The device will be automatically discovered in Home Assistant

### Test Configurations

The [`esphome/test/`](esphome/test/) folder contains:
- Minimal test configs to discover meter IDs in range
- Pre-compiled firmware binaries for quick USB flashing

---

## Version History

| Version | Period | Technology | Status |
|---------|--------|-----------|--------|
| **v5** | 2025+ | ESPHome + ESP32/S3 + CC1101 (esp-idf) | **Current** |
| v4 | 2023-2024 | ESPHome + ESP32 + CC1101 (Arduino) | Archived |
| v3 | 2022 | ESPHome + ESP8266/ESP32 + CC1101 | Archived |
| v2 | 2021 | Python App + NanoCUL 868 MHz | Archived |
| v1 | 2020 | Python App + RTL-SDR DVB-T Stick | Archived |

### What's New in v5?

- Switched from Arduino framework to **esp-idf** for better stability
- Support for **ESP32-S3** boards (Heltec V4 with LoRa + OLED display)
- Dynamic meter ID change at runtime (no reflash needed)
- Optimized memory usage
- Pre-built firmware binaries for easy initial USB flashing

---

## Project Structure

```
ha-watermeter/
├── README.md               ← You are here
├── LICENSE
├── esphome/                ← v5 configurations (current)
│   ├── atom-watermeter.yaml
│   ├── esp32-devkit-v4-wasserzahler.yaml
│   ├── heltec-watermeter.yaml
│   ├── wm5-heltec-esp32S3-v4.yaml
│   └── test/               ← Test configs & firmware binaries
├── docs/                   ← Documentation, images, datasheets
│   └── homeassistant/      ← HA dashboard & sensor configs
└── archive/                ← Older versions (v1-v4)
    ├── esphome-v4/         ← ESPHome v4 configurations
    ├── python-app/         ← Python MQTT bridge (v1/v2)
    ├── nanocul/            ← NanoCUL 868 documentation
    └── rtl-sdr/            ← RTL-SDR DVB-T documentation
```

---

## Wiring CC1101 to ESP32

```
                                                           o 1 (3.3V)
                                                           |
 ╭――x――x――x――x――x――x――x――x――x――x――x――x――x――x――x――x――x――x――x――o―╮
 |                                                             |
 | 5v               az-delivery-devkit-v4                      | -- ANT
 |                                                             |
 |                          16 17 5  18 19               23    |
 ╰――x――x――x――x――x――x――x――x――o――x――o――o――o――o――o――o――o――o――o――o―╯
                            |  |  |  |  |                 |   |
                            o  |  |  o  |                 |   ╰-o - 2 (GND)
                            7  o  |  4  o                 o
                          GDO0 6  | CLK 5                 3
                             GD02 o    MISO              MOSI
                                  8
                                 CSN
```

| CC1101 Pin | ESP32 GPIO | Function |
|------------|-----------|----------|
| MOSI | GPIO23 | SPI Data Out |
| MISO | GPIO19 | SPI Data In |
| SCK | GPIO18 | SPI Clock |
| CSN | GPIO05 | Chip Select |
| GDO0 | GPIO16 | RX Clock |
| GDO2 | GPIO17 | TX Status |
| VCC | 3.3V | Power |
| GND | GND | Ground |

---

## Links & Resources

- [SzczepanLeon ESPHome wMBus Components](https://github.com/SzczepanLeon/esphome-components)
- [wmbusmeters](https://github.com/weetmuts/wmbusmeters)
- [wmbusmeters Telegram Decoder](https://wmbusmeters.org/)
- [SmartRC-CC1101-Driver-Lib](https://github.com/LSatan/SmartRC-CC1101-Driver-Lib)
- [Diehl Metering IZAR](https://www.diehl.com/metering/en/)

---

## License

This project is licensed under the [MIT License](LICENSE).
