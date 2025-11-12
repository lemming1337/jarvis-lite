# Flash-Speicher Optimierung - No OTA Partition

## Problem

Die Voice Assistant Firmware mit Wake Word Detection und Display-Grafiken ist zu groß für die Standard ESP32 Partitionierung mit OTA-Support:

- **Standard mit OTA**: factory (1MB) + ota_0 (1MB) + ota_1 (1MB) = max. ~1.6MB pro App
- **Tatsächliche Firmware-Größe**: ~2-3MB (mit allen Features)

## Lösung

Wir verwenden eine **Custom Partition Table ohne OTA-Support**, was die maximale App-Größe auf **~3MB** erhöht.

## Custom Partition Table

Die Datei `esphome/partitions.csv` definiert:

```csv
# Name,   Type, SubType,  Offset,   Size,     Flags
nvs,      data, nvs,      0x9000,   0x6000,   # 24KB Non-Volatile Storage
phy_init, data, phy,      0xf000,   0x1000,   # 4KB PHY Init Data
factory,  app,  factory,  0x10000,  0x2F0000, # ~3MB App (3.072.000 Bytes)
spiffs,   data, spiffs,   0x300000, 0x100000, # 1MB Filesystem
```

### Partition-Details

| Partition | Größe | Verwendung |
|-----------|-------|------------|
| **nvs** | 24 KB | Non-Volatile Storage (WiFi, Konfiguration) |
| **phy_init** | 4 KB | PHY Initialisierungs-Daten |
| **factory** | 3 MB | Haupt-Applikation (Wake Word Models, Display-Grafiken) |
| **spiffs** | 1 MB | Dateisystem (optional, kann vergrößert werden) |

**Gesamt:** 4 MB Flash (Standard ESP32 DevKit C)

## ESPHome Konfiguration

### 1. ESP-IDF Framework (Wichtig!)

```yaml
esp32:
  board: esp32dev
  framework:
    type: esp-idf          # NICHT arduino!
    version: recommended
    platform_version: recommended
```

**Warum ESP-IDF?**
- Arduino Framework ignoriert custom partition tables
- ESP-IDF unterstützt custom partitions nativ
- Bessere Performance für Voice Assistant

### 2. Custom Partition Table einbinden

```yaml
esphome:
  name: ${name}
  platformio_options:
    board_build.partitions: partitions.csv
    board_upload.flash_size: 4MB
```

### 3. OTA deaktivieren

```yaml
# OTA deaktiviert - Custom Partition für maximale App-Größe
# ota:
#   - platform: esphome
```

**Wichtig:** Ohne OTA sind Updates nur via **USB-Kabel** möglich!

## Vor- und Nachteile

### ✅ Vorteile

- **3x mehr App-Speicher**: ~3MB statt ~1MB
- **Alle Features möglich**: Wake Word + Display + Voice Assistant
- **Schnelleres Flashen**: Nur eine Partition wird geschrieben
- **Einfachere Partition Table**: Weniger Komplexität

### ❌ Nachteile

- **Kein OTA**: Updates nur via USB-Kabel möglich
- **Mehr Aufwand**: Device muss physisch erreichbar sein
- **Entwicklung**: Jedes Update erfordert USB-Verbindung

## Flash-Vorgang

### Erstes Mal (Initial Flash)

```bash
# Via ESPHome Dashboard oder CLI
esphome run esphome/voice-assistant.yaml

# Wähle USB Port (z.B. /dev/ttyUSB0 oder COM3)
```

### Weitere Updates

Da OTA deaktiviert ist, **immer** via USB flashen:

```bash
# ESP32 per USB verbinden
esphome run esphome/voice-assistant.yaml
```

**Alternative: ESPHome Web Flasher**
- https://web.esphome.io/
- Firmware kompilieren in ESPHome
- Download .bin Datei
- Upload via Web Flasher

## Speicherverbrauch Breakdown

Typischer Speicherverbrauch mit allen Features:

| Component | Größe | Beschreibung |
|-----------|-------|--------------|
| **ESPHome Core** | ~500 KB | Basis-Firmware |
| **Wake Word Models** | ~200 KB | okay_nabu + hey_jarvis |
| **Display Driver** | ~100 KB | ILI9341 + Touch |
| **Voice Assistant** | ~300 KB | Audio-Processing |
| **WiFi/Network** | ~200 KB | WiFi + API |
| **Display-Grafiken** | ~800 KB | Casita Illustrationen (240x320) |
| **Fonts** | ~100 KB | Figtree Font Familie |
| **Sonstige** | ~200 KB | Scripts, Globals, etc. |
| **GESAMT** | ~2.4 MB | Tatsächlicher Verbrauch |

Mit Standard-Partitionierung (1.6MB max) → **Passt nicht!**
Mit Custom-Partitionierung (3MB max) → **Passt perfekt!** ✅

## Speicher weiter optimieren (optional)

Falls die Firmware immer noch zu groß ist:

### 1. Display-Grafiken verkleinern

```yaml
image:
  - file: ${idle_illustration_file}
    resize: 200x267  # Kleiner als 240x320
    type: GRAYSCALE  # Statt RGB
```

### 2. Font-Glyphs reduzieren

```yaml
font:
  - id: font_request
    size: 13
    glyphs: " 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"  # Nur benötigte Zeichen
```

### 3. Nur ein Wake Word

```yaml
micro_wake_word:
  models:
    - hey_jarvis  # Nur eines statt zwei
```

### 4. SPIFFS verkleinern

In `partitions.csv`:
```csv
factory,  app,  factory,  0x10000,  0x3E0000,  # 3.9MB statt 3MB
spiffs,   data, spiffs,   0x3F0000, 0x10000,   # 64KB statt 1MB
```

## Framework-Unterschiede: Arduino vs ESP-IDF

| Feature | Arduino | ESP-IDF |
|---------|---------|---------|
| Custom Partitions | ❌ Ignoriert | ✅ Unterstützt |
| Kompilierzeit | Schneller | Langsamer |
| Speicher-Effizienz | Gut | Besser |
| Wake Word Support | ✅ | ✅ |
| I2S Audio | ✅ | ✅ |
| Entwicklung | Einfacher | Komplexer |

**Für dieses Projekt:** ESP-IDF ist **zwingend erforderlich** für Custom Partitions!

## Troubleshooting

### Fehler: "Partition table file not found"

**Lösung:** Stelle sicher, dass `partitions.csv` im gleichen Verzeichnis wie die YAML-Datei liegt:

```
esphome/
├── voice-assistant.yaml
└── partitions.csv
```

### Fehler: "Firmware too large for partition"

Auch mit 3MB zu groß?

1. Prüfe kompilierte Größe:
   ```bash
   esphome compile esphome/voice-assistant.yaml
   # Ausgabe zeigt: "RAM: [====  ]  45.2%  used..."
   #                "Flash: [========]  78.5%  used..."
   ```

2. Optimiere nach Anleitung oben
3. Evtl. SPIFFS weiter verkleinern

### Fehler: "Custom partitions ignored"

**Ursache:** Arduino Framework verwendet

**Lösung:** Wechsel zu ESP-IDF:
```yaml
esp32:
  framework:
    type: esp-idf  # NICHT arduino!
```

### ESP32 bootet nicht mehr

**Ursache:** Ungültige Partition Table

**Lösung:** Neu flashen mit `esptool.py`:
```bash
# Erase Flash komplett
esptool.py --port /dev/ttyUSB0 erase_flash

# Neu flashen
esphome run esphome/voice-assistant.yaml
```

## Migration zurück zu OTA (falls gewünscht)

Falls du später wieder OTA aktivieren möchtest:

1. **partitions.csv löschen** oder umbenennen
2. **OTA wieder aktivieren** in YAML:
   ```yaml
   ota:
     - platform: esphome
       password: !secret ota_password
   ```
3. **Framework optional auf Arduino zurück** (oder bei ESP-IDF bleiben)
4. **Features reduzieren**, damit Firmware < 1.6MB
5. **Via USB flashen** (einmalig)
6. Danach: OTA-Updates möglich

## Empfehlung

Für **Entwicklung/Testing**: Behalte Custom Partition (No OTA)
- Schnelle Iteration via USB
- Alle Features verfügbar
- Keine Größen-Limitierung

Für **Produktion**: Entscheide basierend auf:
- **Wichtig: OTA** → Standard Partition, Features reduzieren
- **Wichtig: Features** → Custom Partition (No OTA), USB-Updates

**Für Jarvis Lite:** Custom Partition ist die beste Wahl, da:
- Wake Word Detection ist Kern-Feature
- Display mit Grafiken ist Kern-Feature
- Device ist meist in Reichweite (Wandmontage)
- USB-Updates sind akzeptabel

## Weitere Ressourcen

- [ESP-IDF Partition Tables](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-guides/partition-tables.html)
- [ESPHome ESP32 Configuration](https://esphome.io/components/esp32.html)
- [ESPHome Custom Partitions (Community)](https://community.home-assistant.io/t/custom-partition-file/629597)
