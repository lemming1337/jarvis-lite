# LVGL Variante - Modernes UI für Voice Assistant

Die LVGL-Variante bietet ein **schlankes, modernes UI** für den Jarvis Voice Assistant ohne die Verwendung großer Bilddateien.

## ✨ Vorteile gegenüber Standard-Variante

### 🎨 Modernes Design
- **Vector-basierte UI-Elemente** statt statischer Bilder
- Weniger Flash-Speicher benötigt (~500KB gespart)
- Flüssigere Animationen
- Responsive Touch-Buttons

### 🚀 Bessere Performance
- Effizienteres Memory-Management
- Hardware-beschleunigte Grafik-Operationen
- Schnellere Screen-Updates
- Weniger CPU-Last

### 💾 Speichereffizienz
- **Keine großen PNG-Dateien** (jeweils ~100-200KB)
- Kompakte Widget-Definitionen
- Optimierte Font-Einbettung

## 📱 UI-Übersicht

### Hauptelemente

```
┌──────────────────────────┐
│      STATUS TEXT         │ ← Status-Label (z.B. "READY", "LISTENING")
├──────────────────────────┤
│                          │
│      ┌────────┐         │
│      │  🎤   │         │ ← Status-Indikator (animierter Kreis)
│      │ ICON  │         │   Farbe ändert sich je nach Phase
│      └────────┘         │
│                          │
│   ┌──────────────┐      │
│   │   00:00      │      │ ← Timer-Widget (nur bei aktivem Timer)
│   └──────────────┘      │
├──────────────────────────┤
│ Request/Response Text    │ ← Text-Feedback
├──────────────────────────┤
│ 🔇  [Mute]        ● OK   │ ← Mute-Button & Connection Status
│ ▓▓▓▓▓▓▓░░░░░░░░░░░      │ ← Timer Progress Bar
└──────────────────────────┘
```

## 🎨 Status-Phasen & Farben

| Status | Farbe | Icon | Beschreibung |
|--------|-------|------|-------------|
| **READY** | 🔵 Blau `0x0080FF` | 💤 | Bereit für Wake Word |
| **LISTENING** | 🟢 Grün `0x00FF00` | 🎤 | Nimmt Befehl auf |
| **THINKING** | 🟠 Orange `0xFFAA00` | 🤔 | Verarbeitet Anfrage |
| **REPLYING** | 🔵 Blau `0x0080FF` | 💬 | Gibt Antwort |
| **ERROR** | 🔴 Rot `0xFF0000` | ❌ | Fehler aufgetreten |
| **MUTED** | ⚫ Grau `0x404040` | 🔇 | Mikrofon stummgeschaltet |
| **TIMER DONE** | 🟣 Magenta `0xFF00FF` | ⏰ | Timer abgelaufen |

## 🔧 Installation

### Voraussetzungen

Alle Voraussetzungen wie bei der Standard-Variante:
- ESP32 DevKit C
- AZ-Touch MOD 2.8" Display
- INMP441 Mikrofon
- Home Assistant mit ESPHome (min. 2024.11.0)

### Flash-Vorgang

```bash
# Erste Installation via USB
esphome run esphome/voice-assistant-lvgl.yaml

# Wähle USB-Port (z.B. /dev/ttyUSB0)
```

### Konfiguration

Die LVGL-Variante verwendet dieselbe `secrets.yaml` wie die Standard-Variante:

```yaml
wifi_ssid: "DeinWiFiName"
wifi_password: "DeinWiFiPasswort"
api_key: "generierter-api-key"
```

## 🎛️ UI-Interaktion

### Mute-Button
- **Position**: Unten links
- **Icon**: 🔇
- **Aktion**: Toggle Mikrofon Mute/Unmute
- **Feedback**: Status-Icon ändert sich, Display zeigt "MUTED"

### Touch-Gesten
- **Tap auf Mute-Button**: Mikrofon stummschalten
- **Tap überall sonst**: LVGL Touch-Events (für zukünftige Features)

## 📊 Timer-Anzeige

### Timer-Widget (Mitte)
- Erscheint automatisch bei aktivem Timer
- Zeigt verbleibende Zeit im Format `HH:MM` oder `MM:SS`
- Grüner Rahmen signalisiert aktiven Timer

### Timer-Fortschrittsbalken (Unten)
- 15px hoher Balken am unteren Bildschirmrand
- Grüne Füllung zeigt verbleibende Zeit visuell
- Animiert bei Fortschritt

## 🔗 Connection Status

Unten rechts zeigt ein kleiner Indikator den Verbindungsstatus:

- **● OK** (Grün): WiFi & Home Assistant verbunden
- **○ WiFi** (Rot): Keine WiFi-Verbindung
- **○ HA** (Orange): WiFi OK, aber Home Assistant nicht erreichbar

## ⚙️ Anpassungen

### Farben ändern

In `voice-assistant-lvgl.yaml` im `update_ui` Script:

```cpp
status_indicator->set_style_bg_color(lv_color_hex(0x00FF00), 0);  // Grün
```

Farbcodes im Hex-Format: `0xRRGGBB`

### Fonts ändern

LVGL nutzt Montserrat-Fonts in verschiedenen Größen:
- `montserrat_32`: Status-Text
- `montserrat_28`: Timer-Anzeige
- `montserrat_20`: Info-Text
- `montserrat_16`: Kleine Labels

### Themes anpassen

Im `lvgl:` Abschnitt:

```yaml
theme:
  obj:
    bg_color: 0x000000      # Schwarz
    text_color: 0xFFFFFF    # Weiß
  btn:
    bg_color: 0x1e1e1e      # Dunkelgrau
    radius: 10              # Abgerundete Ecken
```

## 🐛 Troubleshooting

### LVGL lädt nicht / Display bleibt schwarz

**Lösung 1: Memory-Logs prüfen**
```yaml
logger:
  level: DEBUG
```
Schaue nach "LVGL" Meldungen in den Logs.

**Lösung 2: LVGL Pause/Resume**
Falls das Display "hängt":
```bash
# In Home Assistant ESPHome Logs
# Suche nach "lvgl" Fehlern
```

**Lösung 3: Color Depth reduzieren**
Falls zu wenig RAM:
```yaml
lvgl:
  color_depth: 8  # statt 16
```

### Touch reagiert nicht auf Buttons

**Kalibrierung prüfen:**
```yaml
touchscreen:
  calibration:
    x_min: 220
    x_max: 3900
    y_min: 200
    y_max: 3850
```

**Touch-Debug aktivieren:**
```yaml
touchscreen:
  on_touch:
    - lambda: |-
        ESP_LOGI("touch", "x=%d, y=%d", (int)touch.x, (int)touch.y);
```

### UI-Elemente nicht sichtbar

**Widget-Positionen prüfen:**
```yaml
- label:
    x: 120  # Zentriert bei 240px Breite
    y: 160  # Mitte bei 320px Höhe
```

**Hidden-Flag prüfen:**
```cpp
timer_container->clear_flag(LV_OBJ_FLAG_HIDDEN);  // Anzeigen
timer_container->add_flag(LV_OBJ_FLAG_HIDDEN);    // Verstecken
```

## 🔄 Unterschiede zur Standard-Variante

| Feature | Standard-Variante | LVGL-Variante |
|---------|------------------|---------------|
| **UI-Technologie** | ILI9xxx Display Pages | LVGL Widgets |
| **Grafiken** | PNG-Bilder (~1.5MB) | Vector-Grafik (~50KB) |
| **Animationen** | Bild-Wechsel | Smooth Transitions |
| **Touch-Buttons** | Touch-Areas | LVGL Buttons |
| **Memory** | Höher (Bilder) | Niedriger (Widgets) |
| **Anpassbarkeit** | Mittel | Hoch |
| **Flash-Größe** | ~2.4MB | ~1.9MB |

## 🚀 Erweiterte Features

### Zukünftige Erweiterungen

Mit LVGL sind folgende Features einfach umsetzbar:

- **Swipe-Gesten** für Seiten-Navigation
- **Animierte Übergänge** zwischen Status
- **Slider** für Lautstärke-Kontrolle
- **Mehrere Pages** (z.B. Settings-Page)
- **Charts** für Audio-Level-Anzeige
- **Popup-Dialoge** für Benachrichtigungen

### Beispiel: Volume Slider hinzufügen

```yaml
widgets:
  - slider:
      id: volume_slider
      x: 20
      y: 250
      width: 200
      height: 20
      min_value: 0
      max_value: 100
      on_value:
        - lambda: |-
            // Set voice assistant volume
            id(va).set_volume_multiplier(x / 100.0);
```

## 📚 LVGL Ressourcen

- [LVGL Docs](https://docs.lvgl.io/)
- [ESPHome LVGL Component](https://esphome.io/components/lvgl.html)
- [LVGL Widgets](https://docs.lvgl.io/master/widgets/index.html)
- [LVGL Examples](https://docs.lvgl.io/master/examples.html)

## 🔁 Zurück zur Standard-Variante

Falls du zur Standard-Variante zurück möchtest:

```bash
esphome run esphome/voice-assistant.yaml
```

Beide Varianten können parallel auf verschiedenen ESP32-Boards laufen.
