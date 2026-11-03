# KiCad — czujnik gazu (DevKit + moduł MH-MQ)

Schemat ideowy pod **płytkę uniwersalną**. Bez PCB.

```
docs/kicad/gas-detector-mq/gas-detector-mq.kicad_pro
```

Otwórz `.kicad_sch`. Pliku `.kicad_pcb` nie ma celowo. Opis połączeń, dzielnika, grzałki i wygrzewania: [README.md](../../../README.md) w katalogu głównym.

Biblioteka `symbols/gas-detector.kicad_sym`:

| Symbol | Piny |
|--------|------|
| `ESP32_DEVKIT` | złącze 30 pin + USB; lewy rząd od 3V3, prawy od 5V/VIN |
| `MH_MQ_SENSOR` | VCC, GND, DO, AO |

Rezystory i kondensatory: `Device:R`, `Device:C`, `Device:C_Polarized`. LED, dioda, NPN i brzęczyk są w schemacie jako `gas-detector:LED`, `gas-detector:DIODE`, `gas-detector:NPN`, `gas-detector:BUZZER`.

## Elementy

| Ref | Część | Wartość |
|-----|--------|---------|
| U1 | ESP32 DevKit V1 30 pin | USB → płytka, 5V/VIN → moduł |
| U2 | MH-MQ-Sensor | VCC z VIN, nie z 3V3 |
| R1, R3 | szeregowo z AO i DO | 10 kΩ |
| R2, R4 | z węzła do masy | 20 kΩ |
| C1 | ceramika przy VCC | 100 nF |
| C2 | elektrolit, plus do +5V | 10 µF / 50 V |
| R5, R6 | szeregowo z LED | 220 Ω |
| R7 | baza Q1 | 1 kΩ |
| D1 | LED zielona, brak wycieku | GPIO32, katoda do masy |
| D2 | LED czerwona, wyciek | GPIO33, katoda do masy |
| Q1 | BC547 | GPIO25 przez R7 |
| BZ1 | TMB12A05, plus do +5V | kolektor Q1 |
| D3 | 1N4148, pasek do +5V | równolegle do BZ1 |

AO / A0 (`AO_3V3`) → **D34** (GPIO34, ADC1). DO / D0 (`DO_3V3`) → **D35** (GPIO35, też ADC1, tylko wejście). Alarm przy 0,35 V na GPIO34: zielona GPIO32, czerwona GPIO33, brzęczyk GPIO25. Krzyżyk = pin niepodłączony.
