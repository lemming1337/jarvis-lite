# ESP32-S3 LVGL Voice Assistant mit TMC5160T Pro Stepper-Treiber

## Übersicht

Diese Variante erweitert den Jarvis Voice Assistant um vollständige SPI-Steuerung des **TMC5160T Pro** Stepper-Treibers. Der TMC2209 (UART) wurde durch den leistungsstärkeren TMC5160T Pro (SPI) ersetzt.

### Hardware-Konfiguration

**Board**: Unexpected Maker ProS3 (ESP32-S3-MINI-1-N16R8)
- 16MB Flash
- 8MB PSRAM (Octal, 120MHz)
- 240MHz CPU

**Komponenten**:
- Display: ILI9341 2.4" (240x320) über SPI
- Touch: XPT2046 über SPI
- Mikrofon: INMP441 über I2S
- **Stepper-Treiber**: TMC5160T Pro über SPI

## GPIO-Pin-Belegung

### Display & Touch (SPI Bus 1 - spi_main)
```
CLK:  GPIO18
MOSI: GPIO23
MISO: GPIO19
DC:   GPIO4
CS:   GPIO5
RST:  GPIO22
Touch CS: GPIO14
Touch IRQ: GPIO27
Backlight: GPIO15
```

### TMC5160T Pro (SPI Bus 2 - spi_tmc)
```
CLK (SCK):  GPIO12
MOSI (SDI): GPIO13
MISO (SDO): GPIO14
CS:         GPIO11
DIAG0:      GPIO16 (optional, für Stallguard)
```

### Mikrofon (I2S)
```
WS (LRCLK): GPIO25
SCK (BCLK): GPIO26
SD (DIN):   GPIO33
```

### Sonstige
```
Buzzer: GPIO21
```

## TMC5160T Features

### Standard-Konfiguration
- **StealthChop-Modus** aktiviert (leiser Betrieb)
- **Mikroschritte**: 256 (konfigurierbar)
- **Laufstrom (IRUN)**: 20/31 (ca. 1.4A bei 2A Max)
- **Haltestrom (IHOLD)**: 10/31 (ca. 0.7A)
- **Max. Geschwindigkeit**: 50.000 Steps/s
- **Beschleunigung**: 1.000 Steps/s²

### Home Assistant Services

#### 1. Motor zu absoluter Position bewegen
```yaml
service: esphome.jarvis_lite_s3_tmc5160t_tmc_move_to
data:
  position: 10000  # Zielposition in Steps
```

#### 2. Relative Bewegung
```yaml
service: esphome.jarvis_lite_s3_tmc5160t_tmc_move_relative
data:
  steps: 5000  # Positive oder negative Steps
```

#### 3. Geschwindigkeitsmodus (kontinuierliche Bewegung)
```yaml
service: esphome.jarvis_lite_s3_tmc5160t_tmc_set_velocity
data:
  velocity: 20000  # Steps/s (positiv = vorwärts, negativ = rückwärts)
```

#### 4. Not-Stopp
```yaml
service: esphome.jarvis_lite_s3_tmc5160t_tmc_stop
```

#### 5. Motor aktivieren/deaktivieren
```yaml
service: esphome.jarvis_lite_s3_tmc5160t_tmc_enable
data:
  enable: true  # true = Motor aktiviert, false = deaktiviert
```

### Konfigurierbare Parameter (über Home Assistant)

#### Geschwindigkeit & Beschleunigung
- **TMC Max Speed**: 1.000 - 200.000 Steps/s
- **TMC Acceleration**: 100 - 10.000 Steps/s²

#### Motor-Ströme
- **TMC Motor Current (RUN)**: 0-31 (0 = 0A, 31 = max. Strom)
- **TMC Motor Current (HOLD)**: 0-31

*Hinweis: Motorstrom hängt vom Sensewiderstandes ab. Bei 0.075Ω: 31 ≈ 2A, 20 ≈ 1.4A*

### Sensoren

Die folgenden Sensoren werden automatisch in Home Assistant angezeigt:

1. **TMC Current Position** - Aktuelle Motor-Position in Steps
2. **TMC Current Velocity** - Aktuelle Geschwindigkeit in Steps/s
3. **TMC Driver Status** - TMC5160T Statusregister (für Diagnose)

### Schalter

- **TMC5160T Motor Enable** - Motor aktivieren/deaktivieren

## TMC5160T Register-Zugriff

Die Implementierung bietet direkten Zugriff auf TMC5160T-Register über Lambda-Code:

```cpp
// Register schreiben
id(tmc_write_register)(0x00, 0x00000004);  // GCONF

// Register lesen
uint32_t status = id(tmc_read_register)(0x6F);  // DRV_STATUS
```

### Wichtige Register

| Register | Adresse | Beschreibung |
|----------|---------|--------------|
| GCONF | 0x00 | Global Configuration |
| GSTAT | 0x01 | Global Status |
| IHOLD_IRUN | 0x10 | Motor-Ströme |
| RAMPMODE | 0x20 | Bewegungsmodus (0=Position, 1/2=Velocity) |
| XACTUAL | 0x21 | Aktuelle Position (read) |
| VACTUAL | 0x22 | Aktuelle Geschwindigkeit (read) |
| XTARGET | 0x2D | Ziel-Position |
| VMAX | 0x27 | Maximale Geschwindigkeit |
| AMAX | 0x26 | Beschleunigung |
| CHOPCONF | 0x6C | Chopper Configuration |
| DRV_STATUS | 0x6F | Driver Status (Fehler, Warnungen) |
| PWMCONF | 0x70 | StealthChop PWM Config |

## Verdrahtung TMC5160T Pro

### SPI-Verbindung (erforderlich)
```
ESP32-S3      TMC5160T Pro
---------     -------------
GPIO12   -->  CLK (SCK)
GPIO13   -->  SDI (MOSI)
GPIO14   <--  SDO (MISO)
GPIO11   -->  CSN (Chip Select)
GND      ---  GND
3.3V     -->  VCC_IO (Logik-Versorgung)
```

### Power (Motor-Versorgung)
```
Power Supply  TMC5160T Pro
------------  -------------
+12-48V  -->  VS (Motor Supply)
GND      ---  GND
```

### Motor-Anschluss (Bipolar Stepper)
```
Stepper       TMC5160T Pro
-------       -------------
Coil A+  -->  OA1
Coil A-  -->  OA2
Coil B+  -->  OB1
Coil B-  -->  OB2
```

### Optional: DIAG-Pin (Stallguard)
```
GPIO16   <--  DIAG0 (über 1kΩ Pullup zu 3.3V)
```

## Beispiel Home Assistant Automation

### Einfache Position anfahren
```yaml
automation:
  - alias: "Motor zu Startposition"
    trigger:
      - platform: state
        entity_id: input_boolean.motor_home
        to: "on"
    action:
      - service: esphome.jarvis_lite_s3_tmc5160t_tmc_move_to
        data:
          position: 0
```

### Kontinuierliche Bewegung bei Tastendruck
```yaml
automation:
  - alias: "Motor vorwärts"
    trigger:
      - platform: state
        entity_id: input_boolean.motor_forward
        to: "on"
    action:
      - service: esphome.jarvis_lite_s3_tmc5160t_tmc_enable
        data:
          enable: true
      - service: esphome.jarvis_lite_s3_tmc5160t_tmc_set_velocity
        data:
          velocity: 30000

  - alias: "Motor stopp"
    trigger:
      - platform: state
        entity_id: input_boolean.motor_forward
        to: "off"
    action:
      - service: esphome.jarvis_lite_s3_tmc5160t_tmc_stop
```

## Unterschiede zu TMC2209

| Feature | TMC2209 (UART) | TMC5160T Pro (SPI) |
|---------|----------------|---------------------|
| **Kommunikation** | UART (Single-Wire) | SPI (4-Wire) |
| **Max. Strom** | 2A RMS | 3A RMS |
| **Spannung** | 5.5-28V | 8-60V |
| **SPI-Geschwindigkeit** | - | Bis 4 MHz |
| **externe MOSFETs** | Nein | Ja (integriert) |
| **StallGuard4** | Ja | Ja (erweitert) |
| **CoolStep** | Ja | Ja |
| **StealthChop2** | Ja | StealthChop2 |
| **Rampen-Generator** | Software | Hardware (integriert) |
| **Position Tracking** | Software | Hardware |

## Fehlerbehebung

### Motor bewegt sich nicht
1. Überprüfe Motor-Enable: `TMC5160T Motor Enable` auf ON
2. Prüfe Motorstrom-Einstellungen (min. 10 für IRUN)
3. Kontrolliere Verkabelung (SPI + Motor-Anschluss)
4. Überprüfe Logs: `TMC5160T initialized successfully` sollte erscheinen

### SPI-Kommunikationsfehler
1. Prüfe SPI-Pins (CLK, MOSI, MISO, CS)
2. Stelle sicher, dass keine anderen Geräte GPIO12-14 verwenden
3. Prüfe VCC_IO (3.3V) am TMC5160T

### Motor vibriert oder läuft unrund
1. Reduziere Max Speed
2. Erhöhe Acceleration-Werte
3. Prüfe Mikrostep-Einstellung in CHOPCONF
4. Aktiviere StealthChop (sollte standardmäßig aktiv sein)

### Position verloren nach Reset
- Positionen werden im TMC5160T gespeichert, gehen aber bei Stromausfall verloren
- Implementiere Homing-Routine mit Endschaltern

## Erweiterte Konfiguration

### SpreadCycle statt StealthChop
Füge in `init_tmc5160t` hinzu:
```cpp
id(tmc_write_register)(0x00, 0x00000000);  // GCONF: SpreadCycle
```

### StallGuard-Schwellwert anpassen
```cpp
// In COOLCONF Register (0x6D)
id(tmc_write_register)(0x6D, 0x00010000);  // SGT = 1 (0 = sensibler, 63 = weniger sensibel)
```

### Höhere SPI-Geschwindigkeit
Ändere in der SPI-Konfiguration:
```yaml
spi:
  - id: spi_tmc
    clk_pin: GPIO12
    mosi_pin: GPIO13
    miso_pin: GPIO14
    data_rate: 4MHz  # Statt 1MHz
```

## Lizenz & Credits

- ESPHome: https://esphome.io
- TMC5160T Datasheet: Trinamic/Analog Devices
- Basierend auf: jarvis-lite Voice Assistant Project

## Support

Bei Problemen:
1. Aktiviere DEBUG-Level im Logger
2. Überprüfe ESPHome Logs
3. Teste TMC5160T Kommunikation mit `DRV_STATUS` Sensor

---

**Version**: 1.0.0
**Erstellt**: 2024-12-23
**Hardware**: ESP32-S3 ProS3 + TMC5160T Pro
