# Wake Word Detection - Jarvis Lite Voice Assistant

## Übersicht

Dein Jarvis Lite Voice Assistant unterstützt jetzt **On-Device Wake Word Detection** mit dem `micro_wake_word` Component von ESPHome. Das bedeutet, dass die Wake Word Erkennung direkt auf dem ESP32 läuft, ohne Cloud-Verbindung.

## Verfügbare Wake Words

Die Konfiguration enthält zwei Wake Words:

1. **"Okay Nabu"** (Standard)
2. **"Hey Jarvis"**

## Wake Word Engine Locations

Du kannst wählen, wo die Wake Word Erkennung stattfindet:

### Option 1: On Device (Standard)
- Wake Word Detection läuft **lokal auf dem ESP32**
- Keine Daten werden gesendet, bis das Wake Word erkannt wird
- Geringere Latenz
- Funktioniert auch bei langsamer Internet-Verbindung
- **Empfohlen für Datenschutz**

### Option 2: In Home Assistant
- Wake Word Detection läuft auf dem **Home Assistant Server**
- Audio-Stream wird kontinuierlich an Home Assistant gesendet
- Höhere Server-Last
- Flexibler (mehr Wake Words verfügbar)

## Konfiguration in Home Assistant

### Wake Word Engine wechseln

1. Gehe zu **Einstellungen** → **Geräte & Dienste** → **ESPHome**
2. Wähle dein "Jarvis Voice Assistant" Gerät
3. Unter **Konfiguration** findest du "Wake word engine location"
4. Wähle "On device" oder "In Home Assistant"

### Mute Funktion

Du kannst das Mikrofon jederzeit muten:

1. In Home Assistant: Toggle den "Mute" Switch
2. Am Display: Touchscreen-Button unten in der Mitte (80-160px, 280-310px)

Wenn gemuted, zeigt das Display "MUTED" an und reagiert nicht auf Wake Words.

## Display Views

Die Konfiguration enthält folgende Display-Ansichten (angepasst für 240x320):

### Idle Page (Bereit)
- Zeigt "Casita" Idle-Animation
- Timer-Widget (wenn Timer aktiv)
- Timer-Timeline am unteren Rand

### Listening Page (Hört zu)
- "Casita" hört zu Animation
- Buzzer spielt kurzen Ton
- Timer-Timeline sichtbar

### Thinking Page (Verarbeitet)
- "Casita" denkt Animation
- Text-Box oben zeigt deine Anfrage
- Timer-Timeline sichtbar

### Replying Page (Antwortet)
- "Casita" antwortet Animation
- Text-Box oben: Deine Anfrage
- Text-Box unten: Antwort von Assist
- Timer-Timeline sichtbar

### Error Page (Fehler)
- "Casita" Error-Animation
- Buzzer spielt Error-Ton (3x Piep)
- Nach 1 Sekunde zurück zu Idle

### Timer Finished Page
- "Casita" Timer-Finished-Animation
- Buzzer spielt bis du den Timer stoppst

### Muted Page
- Schwarzer Bildschirm mit "MUTED" Text
- Timer-Widget und Timeline sichtbar

### No WiFi / No HA Page
- Error-Illustration
- Zeigt Verbindungsprobleme an

## Timer-Funktionalität

Der Voice Assistant unterstützt vollständige Timer-Verwaltung:

### Timer setzen
Sage: *"Hey Jarvis, set a timer for 5 minutes"*

### Timer-Anzeige
- **Timeline**: Horizontaler Balken am unteren Rand (grün = aktiv, blau = pausiert)
- **Widget**: Timer-Countdown im Idle/Muted Screen (nur bei aktivem Timer)

### Timer-Management
- Mehrere Timer gleichzeitig möglich
- Anzeige des nächsten ablaufenden Timers
- Unterscheidung zwischen aktiven und pausierten Timern

## Buzzer-Feedback

Der Piezo-Buzzer gibt akustisches Feedback:

| Ereignis | Ton |
|----------|-----|
| Listening | 1x kurzer Piep (E6) |
| Thinking | 2x kurzer Piep (E6) |
| Error | 3x tiefer Piep (A4) |
| Timer Finished | Wiederholend bis gestoppt |

## Display-Layout Details (240x320)

### Portrait Mode Anpassungen

Gegenüber dem S3-BOX (320x240 Landscape) wurden folgende Anpassungen vorgenommen:

```
Original S3-BOX:  320 x 240 (Landscape)
AZ-Touch MOD:     240 x 320 (Portrait)
```

#### Text-Boxen
- **Breite**: 220px (statt 280px)
- **Position X**: 10px (statt 20px)
- **Request Box Y**: 20px (oben)
- **Response Box Y**: 270px (unten)

#### Timer-Timeline
- **Breite**: 240px (statt 320px)
- **Position Y**: 305px (unten)
- **Höhe**: 15px

#### Timer-Widget
- **Breite**: 140px (statt 160px)
- **Position**: Zentriert (50px, 60px)
- **Höhe**: 40px (statt 50px)

#### Bilder
- Alle Illustrationen werden auf **240x320** skaliert
- Transparenz wird beibehalten (alpha_channel)

## Erweiterte Einstellungen

### Noise Suppression
```yaml
noise_suppression_level: 2
```
Level 0-4, Standard: 2 (reduziert Hintergrundgeräusche)

### Auto Gain
```yaml
auto_gain: 31dBFS
```
Automatische Verstärkungsregelung für gleichbleibende Lautstärke

### Volume Multiplier
```yaml
volume_multiplier: 2.0
```
Verstärkung für TTS-Ausgabe (wenn Lautsprecher hinzugefügt wird)

## Troubleshooting

### Wake Word wird nicht erkannt

1. **Prüfe INMP441 Verkabelung**
   - VDD → 3.3V (NICHT 5V!)
   - L/R → GND (Left Channel)
   - Alle I2S Pins korrekt

2. **Prüfe ESPHome Logs**
   ```bash
   # In Home Assistant
   ESPHome → Jarvis Voice Assistant → Logs
   ```
   Suche nach: "micro_wake_word" Meldungen

3. **Mute Status prüfen**
   - Display darf nicht "MUTED" zeigen
   - Mute Switch in HA muss OFF sein

4. **Wake Word Engine Location**
   - Stelle sicher, dass "On device" gewählt ist
   - Bei "In Home Assistant": Voice Pipeline muss konfiguriert sein

### Display zeigt nur "INITIALIZING"

- Warte 30 Sekunden nach dem Boot
- Prüfe WiFi-Verbindung
- Prüfe Home Assistant API-Verbindung

### Buzzer funktioniert nicht

- Prüfe GPIO 21 Verbindung zum Buzzer
- Buzzer hat Resonanz bei 4kHz (optimal für RTTTL)

### Timer-Timeline zeigt nichts

- Timer muss in Home Assistant Voice Assistant gesetzt werden
- Sage: "Hey Jarvis, set a timer for 1 minute"
- Timeline erscheint automatisch

## Wake Word Model Details

Die Models werden automatisch von ESPHome heruntergeladen:

```yaml
micro_wake_word:
  models:
    - okay_nabu     # https://github.com/esphome/micro-wake-word-models
    - hey_jarvis    # Lokal auf ESP32 gespeichert
```

## Performance

### Speicherverbrauch
- ESP32 RAM: ~200KB für Wake Word Models
- Flash: ~100KB pro Model

### CPU-Last
- Idle (Wake Word aktiv): ~15-20%
- Listening/Processing: ~40-60%
- Display Updates: ~5-10%

### Latenz
- Wake Word Detection: ~300ms
- Speech-to-Text: ~1-2s (abhängig von HA)
- Text-to-Speech: ~1-2s (abhängig von HA)

## Nächste Schritte

### Lautsprecher hinzufügen

Um TTS-Ausgabe zu hören, füge einen MAX98357A I2S Verstärker hinzu:

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
```

Siehe [hardware-setup.md](hardware-setup.md) für Verkabelung.

### Eigene Wake Words trainieren

Du kannst auch eigene Wake Words trainieren:
- https://github.com/kahrendt/microWakeWord
- Tutorial: https://www.kevinahrendt.com/micro-wake-word

## Support & Links

- **ESPHome micro_wake_word**: https://esphome.io/components/micro_wake_word/
- **Home Assistant Voice**: https://www.home-assistant.io/voice_control/
- **Wake Word Models**: https://github.com/esphome/micro-wake-word-models
- **S3-BOX Reference**: https://github.com/esphome/wake-word-voice-assistants
