# ESP32-Devkit-V4 Watermeter

**Device:** ESP32 DevKit V4 + CC1101  
**Protocol:** wM-Bus (Wireless M-Bus) 868.95 MHz  
**Meter:** Diehl IZAR RC 868 I R4 PL  
**Hostname:** `water-meter-esp.local`  
**IP:** `10.1.1.71`

---

## WiFi Configuration

### Adding a New WiFi Network

Add a new entry in `secrets.yaml`:
```yaml
ssid_new_name: "MyNetwork"
ssid_new_pswd: "MyPassword"
```

Then add it under `wifi: networks:` in `esp32-devkit-v4-wasserzahler.yaml`:
```yaml
- ssid: !secret ssid_new_name
  password: !secret ssid_new_pswd
  priority: 20
```

Flash via OTA:
```bash
esphome run esp32-devkit-v4-wasserzahler.yaml
```

---

## No WiFi Available — Hotspot Mode

If no known WiFi network is reachable, the device automatically starts its own hotspot after approximately **60 seconds**:

| | |
|---|---|
| **SSID** | `water-meter-esp_AP` |
| **Password** | `055787478310` |
| **IP** | `192.168.4.1` |

### Connecting
1. Connect your phone or laptop to `water-meter-esp_AP`
2. A captive portal should open automatically  
   — or navigate manually to `http://192.168.4.1`
3. The webserver dashboard will be displayed

---

## Changing the Meter ID

### When Is This Needed?
- When the water utility replaces your meter
- When moving the device to a different meter

### Finding the Meter ID
The meter ID is printed on the meter (8-digit hex number, e.g. `24058612`).  
Alternatively, receive all meters in range:

```bash
# Set wildcard — all meters will appear in the log
curl -X POST "http://water-meter-esp.local/text/wasseruhr_meter_id_eingabe/set" --data "value=00000000"
```
Then look for `[wmbus_scan]` lines in the log:
```
Frame: RSSI=-78dBm Mode=T1 Data=1944a511 78071286 0524...
                                          ^^^^^^^^
                                          This is the Meter ID (bytes 4-7, little-endian)
```

### Method 1: curl (recommended, no HA required)

```bash
# Set ID (example: 24058612)
curl -X POST "http://water-meter-esp.local/text/wasseruhr_meter_id_eingabe/set" --data "value=24058612"

# Check current ID
curl "http://water-meter-esp.local/text_sensor/wasseruhr_meter_id"
```

The ESP restarts automatically after 2 seconds and uses the new ID.

### Method 2: Webserver Dashboard

1. Open `http://water-meter-esp.local` in your browser
2. Find the field **"Wasseruhr Meter ID Eingabe"**
3. Enter the 8-digit hex ID (e.g. `24058612`)
4. Press Enter — the ESP will restart

### Method 3: Home Assistant Service

```yaml
service: esphome.water_meter_esp_set_meter_id
data:
  new_meter_id: "24058612"
```

### In Hotspot Mode (no WiFi)

```bash
curl -X POST "http://192.168.4.1/text/wasseruhr_meter_id_eingabe/set" --data "value=24058612"
```

---

## Known Meter IDs

| ID | Description |
|---|---|
| `24058612` | Current meter (since 2024) |
| `43430778` | Previous meter |
| `86120778` | Older meter |
| `00000000` | Wildcard — receive all meters |

---

## Querying Current Values (curl)

```bash
# Total water reading
curl "http://water-meter-esp.local/sensor/wasseruhr_anzeige"

# Current meter ID
curl "http://water-meter-esp.local/text_sensor/wasseruhr_meter_id"

# Individual sensors (JSON)
curl "http://water-meter-esp.local/sensor/wasser_tag"
curl "http://water-meter-esp.local/sensor/wasser_stunde"
curl "http://water-meter-esp.local/sensor/wasser_gestern"
curl "http://water-meter-esp.local/sensor/wasser_monat"
curl "http://water-meter-esp.local/sensor/wasser_jahr"
```

---

## OTA Update

```bash
# Compile and flash firmware
esphome run esp32-devkit-v4-wasserzahler.yaml

# Compile only
esphome compile esp32-devkit-v4-wasserzahler.yaml
```

---

## Webserver

| URL | Description |
|---|---|
| `http://water-meter-esp.local` | Dashboard |
| `http://water-meter-esp.local/logs` | Live Log |
| `http://10.1.1.71` | Direct via IP |
