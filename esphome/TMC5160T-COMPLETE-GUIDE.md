# TMC5160T Pro - Vollständige Implementierung für ESP32-S3

## Übersicht

Diese Implementierung bietet die **VOLLSTÄNDIGE Palette** an TMC5160T Features über SPI, alle konfigurierbar über Home Assistant - KEINE Custom Components, alles inline in der YAML!

### Hardware
- **Board**: ESP32-S3 ProS3 (16MB Flash + 8MB PSRAM @ 120MHz)
- **Display**: ILI9341 2.4" (240x320) via SPI
- **Motor Driver**: TMC5160T Pro via SPI (bis 3A, 8-60V)
- **Voice**: INMP441 I2S Mikrofon + Micro Wake Word

## 🎯 Vollständig Implementierte Features

### ✅ StallGuard2 (Sensorless Homing)
- Lastmessung in Echtzeit
- Konfigurierbare Schwelle (-64 bis +63)
- DIAG0 Pin Unterstützung
- Stall-Event Zähler
- Homing ohne mechanische Endschalter

### ✅ CoolStep (Automatische Stromregelung)
- Dynamische Stromanpassung basierend auf Last
- 4 Parameter: SEMIN, SEMAX, SEDN, SEUP
- Bis zu 75% Energie-Einsparung
- Load Value Sensor (% der maximalen Last)

### ✅ DcStep (Hochgeschwindigkeits-Optimierung)
- Automatische Velocity-Anpassung bei Last
- Verlustfreier Betrieb an Leistungsgrenze
- Konfigurier bare DC_TIME und DC_SG

### ✅ StealthChop2 vs SpreadCycle
- Umschaltbar zwischen leise (StealthChop) und kraftvoll (SpreadCycle)
- Threshold-basierte automatische Umschaltung
- PWM-Autoscale und Gradient-Konfiguration

### ✅ Volle Chopper-Konfiguration
- TOFF, HSTRT, HEND, TBL
- 1-256 Mikroschritte konfigurierbar
- MicroPlyer Interpolation

### ✅ Rampen-Generator (SixPoint)
- Alle Parameter: VSTART, V1, VMAX, A1, AMAX, D1, DMAX, VSTOP
- Hardware-basiert (nicht Software!)
- Positionierungs- und Geschwindigkeitsmodi

### ✅ Endschalter & Homing
- Hardware-Endschalter (GPIO 17 & 10)
- StallGuard-basiertes Homing
- Automatisches Position-Latching

### ✅ Encoder-Unterstützung
- Inkremental-Encoder Eingänge
- Konfigurierbare Encoder-Konstante
- Position-Verifizierung

### ✅ Vollständige Diagnostik
- Echtzeit-Temperatur Überwachung (OTPW, OT)
- Short-to-Ground Erkennung (Coil A/B)
- Open-Load Erkennung
- Überstrom-Warnungen
- GSTAT Error-Flags
- Zähler für alle Error-Typen

### ✅ PWM-Konfiguration (StealthChop)
- PWM Amplitude (0-255)
- PWM Gradient (0-255)
- PWM Frequenz (0-3)
- PWM Autoscale aktivierbar

## 📊 Sensoren (alle Live in Home Assistant)

### Position & Bewegung
- **TMC Current Position** - Aktuelle Position in Steps
- **TMC Current Velocity** - Geschwindigkeit in Steps/s
- **TMC TSTEP** - Step-Timing

### StallGuard2
- **TMC StallGuard2 Result** - SG Wert (0-1023)
- **TMC Stall Count** - Anzahl Stall-Events

### CoolStep
- **TMC Load Value** - Motorlast in %
- **TMC Actual Current Scale** - Aktueller Stromwert (0-31)

### Diagnostik
- **TMC Driver Temperature** - Temperatur (normal/120°C/150°C)
- **TMC Driver Status** - DRV_STATUS Register
- **TMC Driver Errors** - Lesbare Fehler-Flags
- **TMC Active Mode** - StealthChop/SpreadCycle/DcStep
- **TMC Short to Ground Count** - Kurzschluss-Zähler
- **TMC PWM Scale** - PWM Skalierungsfaktor

### Status
- **TMC Microsteps** - Aktuelle Mikroschritte
- **TMC Homing Status** - Not Homed/Homing/Homed

## 🎛️ Konfiguration (Home Assistant Numbers)

### Basis-Parameter
- **TMC Max Speed** (1k-200k steps/s)
- **TMC Acceleration** (100-10k steps/s²)
- **TMC Motor Current RUN** (0-31, ~0-3A)
- **TMC Motor Current HOLD** (0-31)

### StallGuard2
- **TMC StallGuard2 Threshold** (-64 bis +63)
  - Höher = weniger empfindlich
  - Niedriger = empfindlicher
- **TMC StallGuard2 Filter** (0/1)

### CoolStep (alle 0-15)
- **TMC CoolStep SEMIN** - Minimum StallGuard Wert
- **TMC CoolStep SEMAX** - Maximum Stromreduzierung
- **TMC CoolStep SEDN** - Strom-Reduzierung Steps
- **TMC CoolStep SEUP** - Strom-Erhöhung Steps
- **TMC CoolStep SEIMIN** - Min. Stromskala (0=1/2, 1=1/4)

### PWM (StealthChop)
- **TMC PWM Amplitude** (0-255)
- **TMC PWM Gradient** (0-255)
- **TMC PWM Frequency** (0-3: 1/1024, 1/683, 1/512, 1/410 fCLK)
- **TMC PWM Autoscale** (0/1)

### Chopper
- **TMC Chopper TOFF** (2-15) - Off-Time
- **TMC Chopper HSTRT** (0-7) - Hysteresis Start
- **TMC Chopper HEND** (0-15) - Hysteresis End
- **TMC Chopper TBL** (0-3) - Blanking Time

### Schwellwerte (Threshold)
- **TMC StealthChop Threshold** (0-1048575 Hz)
- **TMC CoolStep Threshold** (0-1048575 Hz)
- **TMC DcStep Threshold (THIGH)** (0-1048575 Hz)

### DcStep
- **TMC DcStep Time (DC_TIME)** (0-1023)
- **TMC DcStep SGT (DC_SG)** (0-255)

### Rampen-Parameter (Erweitert)
- **TMC V1** (0-500k steps/s) - Velocity Punkt 1
- **TMC A1** (0-60k steps/s²) - Acceleration bis V1
- **TMC D1** (0-60k steps/s²) - Deceleration von V1
- **TMC VSTART** (0-262143 steps/s) - Start-Velocity
- **TMC VSTOP** (1-262143 steps/s) - Stop-Velocity

### Encoder
- **TMC Encoder Constant** - Encoder Auflösung

## 🔘 Switches (Home Assistant)

### Modi
- **TMC5160T Motor Enable** - Motor aktivieren/deaktivieren
- **TMC StallGuard2 Enable** - StallGuard aktivieren
- **TMC CoolStep Enable** - CoolStep aktivieren
- **TMC DcStep Enable** - DcStep aktivieren
- **TMC SpreadCycle Mode** - SpreadCycle ON / StealthChop OFF
- **TMC Encoder Enable** - Encoder aktivieren
- **TMC Endstops Enable** - Hardware-Endschalter aktivieren
- **TMC Shaft Direction Reverse** - Drehrichtung umkehren

## 🛠️ Home Assistant Services

### Basis-Bewegung
```yaml
# Absolute Position anfahren
service: esphome.jarvis_lite_s3_tmc5160t_tmc_move_to
data:
  position: 10000

# Relative Bewegung
service: esphome.jarvis_lite_s3_tmc5160t_tmc_move_relative
data:
  steps: 5000  # Kann negativ sein

# Velocity-Modus (kontinuierlich)
service: esphome.jarvis_lite_s3_tmc5160t_tmc_set_velocity
data:
  velocity: 30000  # Negativ für Rückwärts

# Not-Stopp
service: esphome.jarvis_lite_s3_tmc5160t_tmc_stop

# Motor aktivieren/deaktivieren
service: esphome.jarvis_lite_s3_tmc5160t_tmc_enable
data:
  enable: true
```

### Erweiterte Services
```yaml
# Homing mit StallGuard/Endstop
service: esphome.jarvis_lite_s3_tmc5160t_tmc_home_to_endstop
data:
  direction: -1  # -1 = links, 1 = rechts

# Aktuelle Position setzen (ohne Bewegung)
service: esphome.jarvis_lite_s3_tmc5160t_tmc_set_position
data:
  position: 0  # Z.B. als Home markieren

# Fehler löschen
service: esphome.jarvis_lite_s3_tmc5160t_tmc_clear_errors

# Mikroschritte ändern (1-256)
service: esphome.jarvis_lite_s3_tmc5160t_tmc_set_microsteps
data:
  microsteps: 128

# Register direkt lesen (Debug)
service: esphome.jarvis_lite_s3_tmc5160t_tmc_read_register_service
data:
  address: 0x6F  # DRV_STATUS

# Register direkt schreiben (Advanced)
service: esphome.jarvis_lite_s3_tmc5160t_tmc_write_register_service
data:
  address: 0x6C
  value: 0x10410153

# StallGuard Tuning starten
service: esphome.jarvis_lite_s3_tmc5160t_tmc_tune_stallguard
data:
  velocity: 30000  # Test-Geschwindigkeit
```

## 🔌 GPIO Pin-Belegung

### Display & Touch (SPI Bus 1)
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

### TMC5160T (SPI Bus 2)
```
CLK (SCK):  GPIO12
MOSI (SDI): GPIO13
MISO (SDO): GPIO14
CS:         GPIO11
DIAG0:      GPIO16 (StallGuard)
```

### Endschalter (Optional)
```
Left:  GPIO17 (active low, pullup)
Right: GPIO10 (active low, pullup)
```

### I2S Mikrofon
```
WS:  GPIO25
SCK: GPIO26
SD:  GPIO33
```

### Sonstige
```
Buzzer: GPIO21
```

## 📐 TMC5160T Verdrahtung

### SPI-Verbindung (erforderlich)
```
ESP32-S3      TMC5160T Pro
---------     -------------
GPIO12   -->  CLK (SCK)
GPIO13   -->  SDI (MOSI)
GPIO14   <--  SDO (MISO)
GPIO11   -->  CSN (CS)
GND      ---  GND
3.3V     -->  VCC_IO
```

### Stromversorgung Motor
```
PSU           TMC5160T Pro
----          -------------
+12-48V  -->  VS
GND      ---  GND
```

### Bipolar Stepper Motor
```
Motor         TMC5160T Pro
-----         -------------
Coil A+  -->  OA1
Coil A-  -->  OA2
Coil B+  -->  OB1
Coil B-  -->  OB2
```

### Optional: StallGuard DIAG Pin
```
GPIO16   <--  DIAG0 (mit 1kΩ Pullup zu 3.3V)
```

### Optional: Hardware-Endschalter
```
GPIO17   <--  Left Endstop (NO, active low)
GPIO10   <--  Right Endstop (NO, active low)
```

## 🎓 Feature-Anleitungen

### StallGuard2 Tuning (Sensorless Homing)

1. **Motor testen**:
   ```yaml
   service: esphome.jarvis_lite_s3_tmc5160t_tmc_tune_stallguard
   data:
     velocity: 30000
   ```

2. **SG_RESULT Sensor beobachten** in Home Assistant

3. **SGT Threshold anpassen**:
   - Wert beobachten während freier Lauf (z.B. 100-300)
   - Wert beobachten bei Blockierung (z.B. 0-50)
   - SGT Threshold auf Mittelwert setzen (z.B. 5-15)

4. **StallGuard aktivieren**:
   - Schalter "TMC StallGuard2 Enable" ON

5. **Homing testen**:
   ```yaml
   service: esphome.jarvis_lite_s3_tmc5160t_tmc_home_to_endstop
   data:
     direction: -1  # Zur linken Seite
   ```

### CoolStep Aktivierung

1. **CoolStep aktivieren**:
   - Schalter "TMC CoolStep Enable" ON

2. **Parameter einstellen**:
   - **SEMIN**: 5 (Minimum StallGuard für Strom-Reduzierung)
   - **SEMAX**: 2 (Maximum Strom-Reduzierung: 2 = 1/4)
   - **SEUP**: 1 (Strom schnell erhöhen bei Last: 1 = 2 steps)
   - **SEDN**: 1 (Strom langsam reduzieren: 1 = 32 steps)

3. **CoolStep Threshold setzen**:
   - Wert > 0 (z.B. 10000) aktiviert CoolStep ab dieser Geschwindigkeit

4. **Load Value Sensor überwachen**:
   - Zeigt aktuelle Motorlast in %
   - Strom wird automatisch angepasst

### DcStep für Hochgeschwindigkeit

1. **DcStep aktivieren**:
   - Schalter "TMC DcStep Enable" ON

2. **THIGH Threshold setzen**:
   - z.B. 100000 (DcStep aktiviert ab dieser Geschwindigkeit)

3. **DC_TIME und DC_SG tunen**:
   - DC_TIME: 400 (Standard)
   - DC_SG: 0 (erhöhen bei zu frühem Velocity-Drop)

4. **Motor beschleunigt automatisch bis zur Lastgrenze!**

### SpreadCycle vs StealthChop

**StealthChop** (leise, Standard):
- Für Geschwindigkeiten < TPWMTHRS (z.B. 500 Hz)
- Nahezu geräuschlos
- Ideal für Positionierung

**SpreadCycle** (kraftvoll):
- Schalter "TMC SpreadCycle Mode" ON
- Höhere Präzision
- Mehr Kraft bei niedrigen Geschwindigkeiten
- Hörbar (sinusartiges Geräusch)

**Hybrid (automatisch)**:
- TPWMTHRS = 500: StealthChop bis 500 Hz, dann SpreadCycle
- Beste aus beiden Welten!

## 🔧 Fehlerbehandlung

### TMC Driver Errors Sensor zeigt Fehler

**"Reset"** - Chip wurde zurückgesetzt
- Service `tmc_clear_errors` aufrufen

**"OT"** / **"OTPW"** - Übertemperatur
- Motor-Ström (IRUN) reduzieren
- Kühlung verbessern
- Duty Cycle reduzieren

**"S2GA"** / **"S2GB"** - Short to Ground
- Motorverkabelung prüfen
- Isolierung prüfen
- `short_count` Sensor überwachen

**"S2VSA"** / **"S2VSB"** - Short to VS
- Motorverkabelung prüfen
- Keine Verbindung zu VS!

**"OLA"** / **"OLB"** - Open Load
- Motoranschluss prüfen (locker?)
- Motor defekt?

**"StallGuard"** - Motor blockiert
- Normal bei Endanschlag
- Bei Bewegung: Last zu hoch oder SGT zu niedrig

### Motor bewegt sich nicht

1. **Motor Enable prüfen**: Schalter ON?
2. **Stromeinstellung prüfen**: IRUN mindestens 10
3. **GSTAT prüfen**: Fehler-Flags vorhanden?
4. **Verkabelung prüfen**: Alle SPI-Pins korrekt?
5. **Logs prüfen**: "TMC5160T initialized successfully"?

### SPI Kommunikationsfehler

1. **VCC_IO**: Muss 3.3V sein!
2. **SPI Pins**: Keine doppelte Verwendung
3. **CS Pin**: Muss exklusiv GPIO11 sein
4. **SPI Bus**: Keine anderen Geräte auf spi_tmc

### Position verloren nach Reset

- TMC5160T hat keine Position-Speicherung bei Stromausfall
- **Lösung**: Homing-Routine nach jedem Boot:
  ```yaml
  esphome:
    on_boot:
      - priority: -100
      - lambda: |-
          delay(5000);  // Warten auf Init
          id(tmc_home_with_stallguard).execute(-1);
  ```

## 📚 TMC5160 Register-Referenz (Auswahl)

| Register | Adresse | Beschreibung |
|----------|---------|--------------|
| **GCONF** | 0x00 | Global Configuration |
| **GSTAT** | 0x01 | Global Status Flags |
| **IHOLD_IRUN** | 0x10 | Motor-Ströme |
| **TPOWERDOWN** | 0x11 | Delay nach Stillstand |
| **TSTEP** | 0x12 | Actual Step Time (read) |
| **TPWMTHRS** | 0x13 | StealthChop Threshold |
| **TCOOLTHRS** | 0x14 | CoolStep Threshold |
| **THIGH** | 0x15 | DcStep Threshold |
| **RAMPMODE** | 0x20 | 0=Position, 1=Vel+, 2=Vel-, 3=Hold |
| **XACTUAL** | 0x21 | Aktuelle Position (R/W) |
| **VACTUAL** | 0x22 | Aktuelle Velocity (read) |
| **VSTART** | 0x23 | Start Velocity |
| **A1** | 0x24 | First Acceleration |
| **V1** | 0x25 | First Velocity |
| **AMAX** | 0x26 | Max Acceleration |
| **VMAX** | 0x27 | Max Velocity |
| **DMAX** | 0x28 | Max Deceleration |
| **D1** | 0x2A | First Deceleration |
| **VSTOP** | 0x2B | Stop Velocity |
| **XTARGET** | 0x2D | Target Position |
| **SW_MODE** | 0x34 | Endstop Configuration |
| **RAMP_STAT** | 0x35 | Ramp Status Flags |
| **ENC_MODE** | 0x38 | Encoder Mode |
| **ENC_CONST** | 0x3A | Encoder Constant |
| **CHOPCONF** | 0x6C | Chopper Configuration |
| **COOLCONF** | 0x6D | CoolStep & StallGuard Config |
| **DCCTRL** | 0x6E | DcStep Configuration |
| **DRV_STATUS** | 0x6F | Driver Status (read) |
| **PWMCONF** | 0x70 | PWM Configuration |
| **PWM_SCALE** | 0x71 | PWM Scale (read) |

## 📖 Beispiel: Vollständige Homing-Automation

```yaml
automation:
  - alias: "TMC5160T Complete Homing Sequence"
    trigger:
      - platform: homeassistant
        event: start
    action:
      # 1. Warten auf TMC Init
      - delay: "00:00:05"

      # 2. Motor aktivieren
      - service: switch.turn_on
        target:
          entity_id: switch.tmc5160t_motor_enable

      # 3. StallGuard aktivieren für Homing
      - service: switch.turn_on
        target:
          entity_id: switch.tmc_stallguard2_enable

      # 4. Homing zur linken Seite
      - service: esphome.jarvis_lite_s3_tmc5160t_tmc_home_to_endstop
        data:
          direction: -1

      # 5. Warten auf Homing
      - wait_template: "{{ is_state('text_sensor.tmc_homing_status', 'Homed') }}"
        timeout: "00:00:30"

      # 6. Position auf 0 setzen
      - service: esphome.jarvis_lite_s3_tmc5160t_tmc_set_position
        data:
          position: 0

      # 7. Zur Mitte fahren (z.B. 50000 steps)
      - service: esphome.jarvis_lite_s3_tmc5160t_tmc_move_to
        data:
          position: 50000

      # 8. StallGuard optional deaktivieren für normalen Betrieb
      - service: switch.turn_off
        target:
          entity_id: switch.tmc_stallguard2_enable

      # 9. CoolStep aktivieren für Energieeinsparung
      - service: switch.turn_on
        target:
          entity_id: switch.tmc_coolstep_enable
```

## 🎯 Optimierungs-Tipps

### Maximum Performance
1. SpreadCycle Mode aktivieren
2. IRUN auf 31 (max)
3. VMAX auf 200000
4. AMAX auf 10000
5. DcStep aktivieren (THIGH=100000)

### Maximum Ruhe
1. StealthChop (SpreadCycle OFF)
2. TPWMTHRS = 0 (immer StealthChop)
3. PWM Autoscale = 1
4. PWM Amplitude = 128

### Maximum Effizienz
1. CoolStep aktivieren
2. SEMIN = 5, SEMAX = 5 (1/32 min. Strom)
3. StealthChop für niedrige Geschwindigkeiten
4. TCOOLTHRS = 10000

### Maximum Präzision
1. 256 Mikroschritte
2. SpreadCycle Mode
3. Niedrige Geschwindigkeit (VMAX=50000)
4. CHOPCONF: HSTRT=5, HEND=3, TOFF=5

## 📦 Changelog

### Version 2.0.0 (2024-12-23) - COMPLETE EDITION
- ✅ **ALLE** TMC5160T Features implementiert!
- ✅ StallGuard2 mit vollständiger Konfiguration
- ✅ CoolStep automatische Stromregelung
- ✅ DcStep Hochgeschwindigkeits-Optimierung
- ✅ Vollständige Chopper-Konfiguration (TOFF, HSTRT, HEND, TBL)
- ✅ PWM-Konfiguration für StealthChop
- ✅ Endschalter-Unterstützung (Hardware + Software)
- ✅ Encoder-Eingänge
- ✅ Vollständige Diagnostik (Temperatur, Fehler-Flags, Zähler)
- ✅ 30+ konfigurierbare Parameter
- ✅ 15+ Sensoren
- ✅ 11 Services
- ✅ 9 Switches für Modi
- ✅ Homing mit StallGuard
- ✅ SixPoint Ramp Generator (alle Parameter)
- ✅ Register-Level Zugriff für Advanced Users
- ✅ Vollständige inline YAML Implementierung (keine Custom Components!)

### Version 1.0.0 (Initial)
- Basis TMC5160T SPI Implementierung

## 🔗 Quellen & Links

- **TMC5160 Datasheet**: [Analog Devices](https://www.analog.com/en/products/tmc5160.html)
- **TMC5160 Datasheet PDF Rev. 1.17**: [Direct Link](https://www.analog.com/media/en/technical-documentation/data-sheets/tmc5160a_datasheet_rev1.17.pdf)
- **BIGTREETECH TMC5160 Wiki**: [GitHub](https://github.com/bigtreetech/BIGTREETECH-TMC5160-V1.0)
- **ESPHome**: https://esphome.io

---

**Entwickelt für**: ESP32-S3 ProS3 + TMC5160T Pro
**Implementierung**: Vollständig inline in ESPHome YAML
**Lizenz**: Open Source
**Version**: 2.0.0 COMPLETE
**Datum**: 2024-12-23
