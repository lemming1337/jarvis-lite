# ESP32 DevKit C - Pinout Referenz für Jarvis Lite

## ESP32 DevKit C (30 Pin) - Pinbelegung Übersicht

```
                         ┌─────────────┐
                         │   USB       │
                         └─────────────┘
                         ┌─────────────┐
                    3V3 │  1     30  │ GND
           Touch IRQ 27 │  2     29  │ 5V
   I2S WS (Mikro)    25 │  3     28  │ GPIO 35 (Input only)
   I2S SCK (Mikro)   26 │  4     27  │ GPIO 32     TFT LED (Backlight)
  I2S SD (Mikro)     33 │  5     26  │ GPIO 33     [siehe links]
                     13 │  6     25  │ GPIO 25     [siehe links]
                     12 │  7     24  │ GPIO 26     [siehe links]
     Buzzer         21 │  8     23  │ GPIO 27     [siehe links]
                      - │  9     22  │ TOUCH CS 14
      SPI CLK       18 │ 10     21  │ GPIO 12
      SPI MISO      19 │ 11     20  │ GPIO 13
 (Status LED) TFT_DC 2 │ 12     19  │ MISO 19     [siehe links]
      SPI MOSI      23 │ 13     18  │ CLK 18      [siehe links]
     TFT RST         4 │ 14     17  │ MOSI 23     [siehe links]
     TFT CS         15 │ 15     16  │ RX0
                        └─────────────┘
```

## Pin-Zuordnung Übersicht

### 🔵 Display & Touch (AZ-Touch MOD)

| GPIO | Funktion | Richtung | Beschreibung |
|------|----------|----------|--------------|
| 2 | TFT_DC | Output | Data/Command Select für Display |
| 4 | TFT_RST | Output | Display Reset |
| 15 | TFT_CS | Output | Display Chip Select (Low = aktiv) |
| 18 | SPI_CLK | Output | SPI Clock (geteilt) |
| 19 | SPI_MISO | Input | SPI Master In Slave Out |
| 23 | SPI_MOSI | Output | SPI Master Out Slave In |
| 14 | TOUCH_CS | Output | Touch Chip Select |
| 27 | TOUCH_IRQ | Input | Touch Interrupt (Low = Touch erkannt) |
| 32 | TFT_LED | Output | Backlight PWM (0-100%) |

### 🎤 Mikrofon (INMP441 I2S)

| GPIO | Funktion | Richtung | Beschreibung |
|------|----------|----------|--------------|
| 25 | I2S_WS | Output | Word Select / Left-Right Clock |
| 26 | I2S_SCK | Output | Serial Clock / Bit Clock |
| 33 | I2S_SD | Input | Serial Data (Mikrofondaten) |

### 🔊 Buzzer

| GPIO | Funktion | Richtung | Beschreibung |
|------|----------|----------|--------------|
| 21 | BUZZER | Output | Piezo Buzzer (PWM, 4kHz Resonanz) |

## Pin-Charakteristiken

### Input-Only Pins
Diese Pins können **NICHT** als Output verwendet werden:
- GPIO 34, 35, 36, 39 (nur ADC)

### Strapping Pins (mit Vorsicht verwenden)
Diese Pins beeinflussen den Boot-Modus:
- **GPIO 0**: Boot-Modus (LOW = Flash-Modus)
- **GPIO 2**: Boot-Modus (muss floating/HIGH sein beim Boot)
- **GPIO 5**: VSPI CS0
- **GPIO 12**: Flash Spannung (LOW = 3.3V, HIGH = 1.8V)
- **GPIO 15**: Debug-Output beim Boot

### Empfohlene freie Pins für Erweiterungen

| GPIO | Verwendung | Hinweise |
|------|------------|----------|
| 5 | GPIO | Gut für Output |
| 13 | GPIO | Gut für Output |
| 16 | GPIO/UART2 RX | Gut für Output |
| 17 | GPIO/UART2 TX | Gut für Output |
| 22 | GPIO/I2C SCL | **Empfohlen für I2S Speaker (MAX98357A)** |
| 34 | Input only | ADC1_CH6 |
| 35 | Input only | ADC1_CH7 |

## Externe Verbindungen

### INMP441 Mikrofon Pinout

```
   ┌─────────────┐
   │   INMP441   │
   ├─────────────┤
   │ VDD    3.3V │ ──→ ESP32 3.3V
   │ GND    GND  │ ──→ ESP32 GND
   │ SD     GPIO │ ──→ GPIO 33
   │ WS     GPIO │ ──→ GPIO 25
   │ SCK    GPIO │ ──→ GPIO 26
   │ L/R    SEL  │ ──→ GND (Left) oder 3.3V (Right)
   └─────────────┘
```

### Potentieller Speaker (MAX98357A)

Wenn später ein Lautsprecher hinzugefügt wird:

```
   ┌─────────────┐
   │  MAX98357A  │
   ├─────────────┤
   │ VIN    5V   │ ──→ ESP32 VIN (5V)
   │ GND    GND  │ ──→ ESP32 GND
   │ DIN    GPIO │ ──→ GPIO 22 (empfohlen)
   │ BCLK   GPIO │ ──→ GPIO 26 (geteilt mit INMP441)
   │ LRC    GPIO │ ──→ GPIO 25 (geteilt mit INMP441)
   │ GAIN   -    │ ──→ Optional (GND/3.3V/Float für 3/6/9dB)
   │ SD     -    │ ──→ Optional (HIGH = On, LOW = Shutdown)
   └─────────────┘
```

## Bus-Übersicht

### SPI Bus (HSPI - Hardware SPI)
```
CLK:  GPIO 18 ┐
MISO: GPIO 19 ├─ Geteilt zwischen Display und Touch
MOSI: GPIO 23 ┘
CS1:  GPIO 15 ── Display Chip Select
CS2:  GPIO 14 ── Touch Chip Select
```

### I2S Bus (Audio)
```
WS (LRCLK): GPIO 25 ┐
SCK (BCLK): GPIO 26 ├─ Geteilt zwischen Mikro und Speaker
SD_IN:      GPIO 33 ── Mikrofon Input
SD_OUT:     GPIO 22 ── Speaker Output (optional)
```

## Stromverbrauch

| Komponente | Typisch | Maximum | Hinweise |
|------------|---------|---------|----------|
| ESP32 | 80 mA | 240 mA | Bei WiFi/BT aktiv |
| Display Backlight | 50 mA | 100 mA | Abhängig von Helligkeit |
| INMP441 | 1.4 mA | 1.5 mA | Sehr niedrig |
| Touch | 1 mA | 2 mA | Nur bei Touch aktiv |
| Buzzer | 10 mA | 30 mA | Nur bei Ton |
| **Gesamt** | ~140 mA | ~370 mA | USB 5V / min. 500mA empfohlen |

## Spannungen

- **ESP32**: 3.3V Logik-Level (5V tolerant an einigen Pins)
- **INMP441**: **NUR 3.3V!** (Wichtig!)
- **Display**: 3.3V Logik, aber über Level-Shifter auf PCB
- **Touch**: 3.3V Logik
- **Stromversorgung**: USB 5V oder VIN 5V

## Wichtige Hinweise

⚠️ **KRITISCH**:
1. INMP441 niemals mit 5V versorgen!
2. GPIO 0 und GPIO 2 nicht LOW beim Boot
3. GPIO 12 beeinflusst Flash-Spannung
4. GPIO 34-39 sind nur Input

✅ **Empfehlungen**:
1. Verwende Hardware-SPI (GPIO 18, 19, 23) für beste Performance
2. Kurze Kabel für I2S Verbindungen (<15cm)
3. Gemeinsame Ground-Verbindung für alle Komponenten
4. USB-Kabel mit guter Qualität (min. 500mA)

## Nächste Schritte

1. Verkabelung gemäß diesem Plan durchführen
2. Mit Multimeter Verbindungen prüfen
3. ESPHome flashen (siehe [README.md](../README.md))
4. In Home Assistant einbinden
