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

## ESPHome Version SzczepanLeon/esphome-components v5 


The current version is based on **ESPHome** with **ESP32 boards** and the **SX1262 868 MHz transceiver**. It offers low resource consumption, stable radio reception, and easy OTA updates.

### Supported Boards (v5)

| Board | Configuration |
|-------|---------------|

| **Heltec ESP32-S3 V4 FR Full** (recommended) | [`esphome/wm5-heltec-esp32s3-v4-FR-full.yaml`](esphome/wm5-heltec-esp32s3-v4-FR-full.yaml) |

> **Recommendation:** The **Heltec boards** (WiFi LoRa 32 V2 and ESP32-S3 V4) are the easiest option. They have the radio chip (SX1262) already built-in on the board, 


- Home Assistant



## Version History

| Version | Period | Technology | Status |
|---------|--------|-----------|--------|
| **v5** | 2025+ | ESPHome + ESP32/S3 + SX1262 (esp-idf) | **Current** |


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
│ l
│   ├── wm5-heltec-esp32S3-v4.yaml
│   
├── docs/                   ← Documentation, images, datasheets
│   └── homeassistant/      ← HA dashboard & sensor configs

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
