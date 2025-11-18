# Jarvis Lite - ESP32 Voice Assistant

Ein Voice Assistant für Home Assistant mit **On-Device Wake Word Detection** basierend auf:
- **AZ-Touch MOD 2.8"** Display-Gehäuse (240x320 Portrait)
- **ESP32 DevKit C** Mikrocontroller
- **INMP441** I2S Mikrofon
- **micro_wake_word** für lokale Wake Word Erkennung

## ✨ Features

- 🎤 **Wake Word Detection**: "Okay Nabu" oder "Hey Jarvis" (On-Device)
- 📱 **Touchscreen Display**: 2.8" mit animierten Status-Anzeigen
- ⏲️ **Timer-Verwaltung**: Visuelles Feedback mit Timeline
- 🔇 **Mute-Funktion**: Per Touch oder Home Assistant
- 🎵 **Audio-Feedback**: Piezo-Buzzer für alle Events
- 🌐 **Zwei Wake Word Modes**: On-Device oder In Home Assistant
- 📊 **Status-Pages**: Idle, Listening, Thinking, Replying, Error

## 🎨 UI-Varianten

Dieses Projekt bietet **mehrere Implementierungen** für verschiedene Hardware:

### Standard ESP32 Varianten

1. **Standard-Variante** (`voice-assistant.yaml`)
   - Verwendet ILI9xxx Display Pages mit PNG-Bildern
   - Bewährte Casita-Illustrationen
   - Einfach zu verstehen und anzupassen
   - 4MB Flash

2. **LVGL-Variante** (`voice-assistant-lvgl.yaml`) ⭐ **EMPFOHLEN**
   - Modernes Widget-basiertes UI ohne große Bilddateien
   - ~500KB weniger Flash-Speicher benötigt
   - Flüssigere Animationen und bessere Performance
   - 4MB Flash
   - **📖 [Zur LVGL-Dokumentation](docs/lvgl-variant.md)**

### ESP32-S3 Variante

3. **S3 LVGL-Variante** (`voice-assistant-lvgl-s3.yaml`) 🚀 **NEU**
   - Optimiert für **Unexpected Maker ProS3** (ESP32-S3)
   - 16MB Flash + 8MB PSRAM
   - 2x größerer LVGL Buffer (50% statt 25%)
   - 2x schnellere Display-Kommunikation (40MHz statt 20MHz)
   - Native USB-C Unterstützung
   - Platz für zukünftige Erweiterungen
   - **📖 [Zur S3-Dokumentation](docs/esp32-s3-variant.md)**

Alle Varianten unterstützen die gleichen Features (Wake Word, Timer, Mute, etc.).

## 📋 Inhaltsverzeichnis

- [Features](#-features)
- [UI-Varianten](#-ui-varianten)
- [Hardware-Setup](#-hardware-setup)
- [Software-Installation](#-software-installation)
- [Wake Word Detection](#-wake-word-detection)
- [Konfiguration](#-konfiguration)
- [Erste Inbetriebnahme](#-erste-inbetriebnahme)
- [Display Views](#-display-views)
- [Troubleshooting](#-troubleshooting)

## 🔧 Hardware-Setup

Eine detaillierte Dokumentation mit allen Pin-Belegungen findest du hier:
**[docs/hardware-setup.md](docs/hardware-setup.md)**

### Benötigte Komponenten

- AZ-Touch MOD mit 2.8" Display (ILI9341 + XPT2046)
- ESP32 DevKit C (30 Pin Version)
- INMP441 I2S MEMS Mikrofon
- Jumper-Kabel für INMP441 Verbindung
- USB-Kabel für ESP32

### Kurzübersicht Verkabelung

```
INMP441 → ESP32
-----------------
VDD  → 3.3V
GND  → GND
SD   → GPIO 33
WS   → GPIO 25
SCK  → GPIO 26
L/R  → GND
```

> ⚠️ **Wichtig**: INMP441 nur mit 3.3V versorgen, NIEMALS 5V!

## 🎤 Wake Word Detection

Jarvis Lite nutzt **micro_wake_word** für lokale Wake Word Erkennung direkt auf dem ESP32:

### Verfügbare Wake Words
- **"Okay Nabu"** (Standard)
- **"Hey Jarvis"**

### Wake Word Modes

**On Device (Standard):**
- ✅ Wake Word läuft lokal auf ESP32
- ✅ Keine Daten werden übertragen bis Wake Word erkannt
- ✅ Geringere Latenz
- ✅ Besserer Datenschutz

**In Home Assistant:**
- Audio-Stream wird kontinuierlich an HA gesendet
- Flexibler (mehr Wake Words verfügbar)
- Höhere Server-Last

**Detaillierte Dokumentation:** [docs/wake-word-setup.md](docs/wake-word-setup.md)

## 💻 Software-Installation

### Voraussetzungen

1. **Home Assistant** (Version 2023.12 oder neuer)
2. **ESPHome** Add-on installiert (min. 2024.11.0)
3. **Home Assistant Voice** Pipeline konfiguriert
4. **USB-Kabel** für ESP32 (Updates nur via USB, kein OTA!)

> ⚠️ **Wichtig**: Diese Konfiguration verwendet eine **Custom Partition ohne OTA-Support** für maximale Firmware-Größe (~3MB). Updates sind **nur via USB** möglich! Siehe [Flash-Optimierung](docs/flash-optimization.md)

### ESPHome Installation

#### Option 1: Via ESPHome Dashboard (empfohlen)

1. Öffne ESPHome Dashboard in Home Assistant
2. Klicke auf "+ NEW DEVICE"
3. Wähle "ESP32"
4. Lade die Datei `esphome/voice-assistant.yaml` hoch
5. Erstelle `secrets.yaml` (siehe unten)

#### Option 2: Via ESPHome CLI

```bash
# ESPHome installieren
pip install esphome

# Konfiguration validieren
esphome config esphome/voice-assistant.yaml

# Kompilieren
esphome compile esphome/voice-assistant.yaml

# Flashen (beim ersten Mal via USB)
esphome upload esphome/voice-assistant.yaml
```

## ⚙️ Konfiguration

### 1. Secrets erstellen

Kopiere die Beispiel-Datei und passe sie an:

```bash
cp esphome/secrets.yaml.example esphome/secrets.yaml
```

Bearbeite `esphome/secrets.yaml`:

```yaml
wifi_ssid: "DeinWiFiName"
wifi_password: "DeinWiFiPasswort"
ap_password: "fallback12345"
api_key: "generierter-api-key"
ota_password: "dein-ota-passwort"
```

### 2. API Key generieren

```bash
# In Home Assistant ESPHome Dashboard wird automatisch ein Key generiert
# Oder manuell:
openssl rand -base64 32
```

### 3. Home Assistant Voice Pipeline einrichten

1. Gehe zu **Einstellungen** → **Sprachassistenten**
2. Wähle eine Pipeline (z.B. "Home Assistant")
3. Konfiguriere:
   - **Speech-to-Text**: Whisper oder Cloud
   - **Intent Recognition**: Home Assistant Conversation
   - **Text-to-Speech**: Piper oder Cloud

## 🚀 Erste Inbetriebnahme

### 1. Hardware zusammenbauen

Folge der Anleitung in [docs/hardware-setup.md](docs/hardware-setup.md)

### 2. ESP32 flashen

**Erstes Mal (via USB):**

```bash
esphome run esphome/voice-assistant.yaml
```

Wähle den USB-Port (z.B. `/dev/ttyUSB0` oder COM3)

**Weitere Updates (OTA über WiFi):**

Nach dem ersten Flash kannst du Updates drahtlos einspielen.

### 3. In Home Assistant einbinden

1. ESP32 sollte automatisch erkannt werden
2. Gehe zu **Einstellungen** → **Geräte & Dienste**
3. Klicke auf "Entdeckte Geräte"
4. Wähle "Jarvis Voice Assistant"
5. Gib den API Key aus `secrets.yaml` ein

### 4. Testen

**Wake Word aktivieren:**
- Sage "Okay Nabu" oder "Hey Jarvis"
- Display wechselt zu "Listening"
- Buzzer piept kurz
- Sprich deinen Befehl (z.B. "Turn on the lights")

**Alternativ: Mute Button**
- Berühre den unteren Display-Bereich
- Mikrofon wird gemuted/unmuted

## 📱 Display Views

Das Display zeigt verschiedene Ansichten je nach Status (alle im 240x320 Portrait Format):

### Idle (Bereit)
- "Casita" Idle-Animation
- Timer-Widget (wenn Timer läuft)
- Timer-Timeline am unteren Rand

### Listening (Hört zu)
- "Casita" Listening-Animation
- Buzzer: 1x kurzer Piep
- Wartet auf deinen Befehl

### Thinking (Verarbeitet)
- "Casita" Thinking-Animation
- Text-Box oben: Zeigt deine Anfrage
- STT wird verarbeitet

### Replying (Antwortet)
- "Casita" Replying-Animation
- Text-Box oben: Deine Anfrage
- Text-Box unten: Antwort des Assistenten

### Timer Finished
- "Casita" Timer-Animation
- Buzzer klingelt bis du den Timer stoppst
- Berühre Display zum Stoppen

### Muted
- Schwarzer Bildschirm mit "MUTED"
- Timer-Anzeige weiterhin aktiv
- Reagiert nicht auf Wake Words

### Error / No Connection
- Error-Illustration
- Zeigt WiFi- oder HA-Verbindungsprobleme

## 🐛 Troubleshooting

### Display bleibt schwarz

- **Prüfe Backlight**: GPIO 32 Verbindung
- **Prüfe SPI**: GPIO 18, 19, 23 Verbindungen
- **Reset**: GPIO 4 sollte HIGH sein

### Touch reagiert nicht

- **Kalibrierung**: Passe `calibration` Werte in YAML an
- **Interrupt**: Prüfe GPIO 27 Verbindung
- **Schwellwert**: Erhöhe `threshold` Wert

### Mikrofon nimmt nichts auf

- **Spannung**: Nur 3.3V, NICHT 5V!
- **L/R Pin**: Muss auf GND (Left Channel)
- **I2S Pins**: GPIO 25, 26, 33 prüfen
- **Log**: Schaue in ESPHome Logs nach Audio-Daten

### WiFi verbindet nicht

- **SSID/Password**: Prüfe `secrets.yaml`
- **2.4 GHz**: ESP32 unterstützt nur 2.4 GHz WiFi
- **Fallback AP**: Nach 1 Min wird Fallback-Hotspot aktiviert
  - SSID: "Jarvis Voice Assistant Fallback"
  - Password: siehe `secrets.yaml`

### Voice Assistant startet nicht

- **Home Assistant Version**: Mind. 2023.12
- **ESPHome Version**: Mind. 2024.11.0
- **Pipeline**: Voice Pipeline muss eingerichtet sein
- **API Key**: Muss in HA übereinstimmen

### Wake Word wird nicht erkannt

- **INMP441 Spannung**: Nur 3.3V, niemals 5V!
- **L/R Pin**: Muss auf GND (Left Channel)
- **Mute Status**: Display darf nicht "MUTED" zeigen
- **Engine Location**: Prüfe "On device" ist ausgewählt
- **Logs prüfen**: ESPHome → Jarvis → Logs (nach "micro_wake_word")

### Display zeigt nur "INITIALIZING"

- **Wartezeit**: 30 Sekunden nach Boot abwarten
- **WiFi**: Verbindung prüfen
- **HA API**: Home Assistant erreichbar?

## 📚 Weitere Ressourcen

### Projekt-Dokumentation
- **[ESP32-S3 ProS3 Variante](docs/esp32-s3-variant.md)** 🚀 - Optimiert für 16MB Flash + 8MB PSRAM
- **[LVGL-Variante](docs/lvgl-variant.md)** ⭐ - Modernes Widget-basiertes UI
- **[Flash-Optimierung](docs/flash-optimization.md)** - Custom Partitions & No-OTA Setup ⚡
- **[Wake Word Setup Guide](docs/wake-word-setup.md)** - Detaillierte Wake Word Dokumentation
- **[Hardware Setup](docs/hardware-setup.md)** - Kompletter Anschlussplan
- **[Pinout Reference](docs/pinout-reference.md)** - GPIO Pin-Belegung

### Externe Ressourcen
- [ESPHome Voice Assistant](https://esphome.io/components/voice_assistant.html)
- [ESPHome micro_wake_word](https://esphome.io/components/micro_wake_word/)
- [ESPHome LVGL Component](https://esphome.io/components/lvgl.html)
- [Home Assistant Voice](https://www.home-assistant.io/voice_control/)
- [S3-BOX Reference Config](https://github.com/esphome/wake-word-voice-assistants)
- [AZ-Touch MOD Hardware](https://www.hwhardsoft.de/english/projects/arduitouch-esp/)

## 🔄 Updates und Erweiterungen

### Firmware Updates (nur USB!)

Da OTA deaktiviert ist, musst du den ESP32 per USB verbinden:

```bash
# ESP32 per USB-Kabel verbinden
esphome run esphome/voice-assistant.yaml

# Wähle USB Port (z.B. /dev/ttyUSB0 oder COM3)
```

**Alternative: ESPHome Web Flasher**
1. Kompiliere Firmware in ESPHome Dashboard
2. Download .bin Datei
3. Öffne https://web.esphome.io/
4. Upload & Flash via Browser

**Warum kein OTA?** Die Firmware mit Wake Word + Display ist zu groß (~2.4MB) für Standard-Partitionierung (max. 1.6MB). Custom Partition ermöglicht bis zu 3MB. Details: [Flash-Optimierung](docs/flash-optimization.md)

### Lautsprecher hinzufügen (MAX98357A)

Für TTS-Ausgabe kannst du einen I2S Verstärker hinzufügen.

**Hardware:**
- MAX98357A I2S Verstärker
- 4-8Ω Lautsprecher (3-5W)

**Verkabelung:** Siehe [docs/hardware-setup.md](docs/hardware-setup.md)

**ESPHome Config:**
```yaml
speaker:
  - platform: i2s_audio
    id: external_speaker
    dac_type: external
    i2s_dout_pin: GPIO22
    i2s_audio_id: i2s_audio_bus
    sample_rate: 48000
    bits_per_sample: 16bit
    channel: left

voice_assistant:
  speaker: external_speaker  # Hinzufügen
```

### Eigene Wake Words trainieren

Du kannst eigene Wake Words erstellen:
- [microWakeWord Trainer](https://www.kevinahrendt.com/micro-wake-word)
- [Training Guide](https://github.com/kahrendt/microWakeWord)

## 📝 Lizenz

Dieses Projekt ist Open Source und frei verwendbar.

## 🤝 Beitragen

Verbesserungen und Bug-Fixes sind willkommen!
