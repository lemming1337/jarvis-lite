# AZ-Touch MOD 2.8" + ESP32 DevKit C + INMP441 Mikrofon

## Hardware-Übersicht

### Komponenten
- **Display-Gehäuse**: AZ-Touch MOD mit 2.8" TFT (320x240)
- **Display-Controller**: ILI9341 (SPI)
- **Touch-Controller**: XPT2046 (SPI, resistiv)
- **Mikrocontroller**: ESP32 DevKit C (30 Pin Version)
- **Mikrofon**: INMP441 I2S MEMS Mikrophone

## Pinbelegung AZ-Touch MOD → ESP32 DevKit C

### Display (ILI9341) - SPI

| AZ-Touch Signal | ESP32 GPIO | Funktion |
|----------------|------------|----------|
| TFT_CS | GPIO 15 | Chip Select Display |
| TFT_DC | GPIO 2 | Data/Command |
| TFT_RST | GPIO 4 | Reset |
| TFT_MOSI | GPIO 23 | SPI MOSI (Master Out Slave In) |
| TFT_MISO | GPIO 19 | SPI MISO (Master In Slave Out) |
| TFT_CLK | GPIO 18 | SPI Clock |
| TFT_LED | GPIO 32 | Backlight (PWM) |

### Touchscreen (XPT2046) - SPI

| AZ-Touch Signal | ESP32 GPIO | Funktion |
|----------------|------------|----------|
| TOUCH_CS | GPIO 14 | Chip Select Touch |
| TOUCH_IRQ | GPIO 27 | Touch Interrupt |
| TOUCH_MOSI | GPIO 23 | SPI MOSI (geteilt mit Display) |
| TOUCH_MISO | GPIO 19 | SPI MISO (geteilt mit Display) |
| TOUCH_CLK | GPIO 18 | SPI Clock (geteilt mit Display) |

### Buzzer

| AZ-Touch Signal | ESP32 GPIO | Funktion |
|----------------|------------|----------|
| BUZZER | GPIO 21 | Piezo Buzzer (4kHz Resonanz) |

## Pinbelegung INMP441 Mikrofon → ESP32 DevKit C

### I2S Verbindung

| INMP441 Pin | ESP32 GPIO | Funktion | Hinweis |
|-------------|------------|----------|---------|
| VDD | 3.3V | Spannungsversorgung | **Wichtig: 3.3V, NICHT 5V!** |
| GND | GND | Masse | |
| SD | GPIO 33 | Serial Data (I2S Data) | Audiodaten |
| WS | GPIO 25 | Word Select (LRCLK) | Left/Right Clock |
| SCK | GPIO 26 | Serial Clock (BCLK) | Bit Clock |
| L/R | GND | Kanal-Auswahl | GND = Left, 3.3V = Right |

### Wichtige Hinweise zum INMP441
- **Niemals 5V anlegen!** Nur 3.3V verwenden
- L/R Pin auf GND für Left-Channel
- SCK und WS werden vom ESP32 als Master generiert
- Die GPIO-Pins sind frei wählbar, aber die obigen sind empfohlen

## Belegte und freie GPIO Pins

### Bereits belegt (AZ-Touch MOD + INMP441)
```
GPIO 2  - TFT_DC
GPIO 4  - TFT_RST
GPIO 14 - TOUCH_CS
GPIO 15 - TFT_CS
GPIO 18 - SPI_CLK (Display + Touch)
GPIO 19 - SPI_MISO (Display + Touch)
GPIO 21 - BUZZER
GPIO 23 - SPI_MOSI (Display + Touch)
GPIO 25 - I2S_WS (Mikrofon)
GPIO 26 - I2S_SCK (Mikrofon)
GPIO 27 - TOUCH_IRQ
GPIO 32 - TFT_LED (Backlight)
GPIO 33 - I2S_SD (Mikrofon)
```

### Noch frei verfügbar
```
GPIO 5, 12, 13, 16, 17, 22, 34, 35, 36, 39
```
*Hinweis: GPIO 34-39 sind nur Input-fähig (ADC1)*

## Lautsprecher-Anbindung (optional)

Falls du später einen Lautsprecher hinzufügen möchtest (z.B. MAX98357A I2S Verstärker):

| MAX98357A | ESP32 GPIO | Funktion |
|-----------|------------|----------|
| VIN | 5V | Spannungsversorgung |
| GND | GND | Masse |
| DIN | GPIO 22 | I2S Data Out |
| BCLK | GPIO 26 | Bit Clock (geteilt mit INMP441) |
| LRC | GPIO 25 | Left/Right Clock (geteilt mit INMP441) |
| GAIN | - | Optional, siehe Datenblatt |
| SD | - | Shutdown (optional) |

## Stromversorgung

- **ESP32 DevKit C**: USB (5V) oder VIN Pin (5V)
- **AZ-Touch MOD**: Hat eigenen Spannungsregler (9-35V DC)
- **Empfehlung**: ESP32 über USB versorgen während der Entwicklung

## Schaltplan (Textdarstellung)

```
┌─────────────────────────────────────────┐
│         AZ-Touch MOD 2.8" PCB           │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │      ILI9341 Display (SPI)       │  │
│  │      XPT2046 Touch (SPI)         │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │   ESP32 DevKit C (30 Pin)        │  │
│  │                                  │  │
│  │  [GPIO Pins wie oben zugeordnet] │  │
│  └──────────────────────────────────┘  │
│                                         │
│  Buzzer [GPIO 21]                       │
└─────────────────────────────────────────┘

        Externe Verbindung:

┌──────────────────┐
│  INMP441 Mikro   │
├──────────────────┤
│ VDD  → 3.3V      │
│ GND  → GND       │
│ SD   → GPIO 33   │
│ WS   → GPIO 25   │
│ SCK  → GPIO 26   │
│ L/R  → GND       │
└──────────────────┘
```

## Wichtige Hinweise für ESPHome

1. **SPI Bus**: Display und Touch teilen sich denselben SPI Bus (HSPI)
2. **I2S Audio**: Separater I2S Bus für Mikrofon (und optional Lautsprecher)
3. **PWM Backlight**: GPIO 32 für Display-Helligkeit
4. **Interrupt**: GPIO 27 für Touch-Erkennung

## Nächste Schritte

1. Hardware gemäß diesem Plan verbinden
2. ESPHome Konfiguration erstellen (siehe `esphome/voice-assistant.yaml`)
3. Mit ESPHome Dashboard flashen
4. In Home Assistant einbinden

## Quellen und Referenzen

- AZ-Touch MOD: https://www.hwhardsoft.de/english/projects/arduitouch-esp/
- INMP441 Datasheet: Invensense/TDK INMP441
- Home Assistant Voice: https://www.home-assistant.io/voice_control/
- ESPHome Voice Assistant: https://esphome.io/components/voice_assistant.html
