# Jarvis Lite - ESP32 Voice Assistant

Ein Voice Assistant für Home Assistant basierend auf:
- **AZ-Touch MOD 2.8"** Display-Gehäuse
- **ESP32 DevKit C** Mikrocontroller
- **INMP441** I2S Mikrofon

## 📋 Inhaltsverzeichnis

- [Hardware-Setup](#hardware-setup)
- [Software-Installation](#software-installation)
- [Konfiguration](#konfiguration)
- [Erste Inbetriebnahme](#erste-inbetriebnahme)
- [Troubleshooting](#troubleshooting)

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

## 💻 Software-Installation

### Voraussetzungen

1. **Home Assistant** (Version 2023.12 oder neuer)
2. **ESPHome** Add-on installiert
3. **Home Assistant Voice** Pipeline konfiguriert

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

- Tippe auf den Push-to-Talk Button am Display
- Sprich einen Befehl (z.B. "Turn on the lights")
- Der Buzzer sollte piepen und das Display reagieren

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
- **Pipeline**: Voice Pipeline muss eingerichtet sein
- **API Key**: Muss in HA übereinstimmen

## 📚 Weitere Ressourcen

- [ESPHome Voice Assistant Dokumentation](https://esphome.io/components/voice_assistant.html)
- [Home Assistant Voice](https://www.home-assistant.io/voice_control/)
- [AZ-Touch MOD Infos](https://www.hwhardsoft.de/english/projects/arduitouch-esp/)

## 🔄 Updates und Erweiterungen

### Wake Word hinzufügen

In `esphome/voice-assistant.yaml`:

```yaml
voice_assistant:
  use_wake_word: true
  # Weitere Wake Word Konfiguration
```

### Lautsprecher hinzufügen (MAX98357A)

Siehe [docs/hardware-setup.md](docs/hardware-setup.md) für Pin-Belegung.

```yaml
speaker:
  - platform: i2s_audio
    id: external_speaker
    dac_type: external
    i2s_dout_pin: GPIO22
    i2s_audio_id: i2s_in
    mode: mono
```

## 📝 Lizenz

Dieses Projekt ist Open Source und frei verwendbar.

## 🤝 Beitragen

Verbesserungen und Bug-Fixes sind willkommen!
