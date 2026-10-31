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

Rezystory i kondensatory: `Device:R`, `Device:C`, `Device:C_Polarized`.

## Elementy

| Ref | Część | Wartość |
|-----|--------|---------|
| U1 | ESP32 DevKit V1 30 pin | USB → płytka, 5V/VIN → moduł |
| U2 | MH-MQ-Sensor | VCC z VIN, nie z 3V3 |
| R1, R3 | szeregowo z AO i DO | 10 kΩ |
| R2, R4 | z węzła do masy | 20 kΩ |
| C1 | ceramika przy VCC | 100 nF |
| C2 | elektrolit THT, plus do +5V | 47 µF / 50 V |

AO / A0 (`AO_3V3`) → **D34** (GPIO34, ADC1). DO / D0 (`DO_3V3`) → **D35** (GPIO35, też ADC1, tylko wejście). Krzyżyk = pin niepodłączony. GPIO32 zostaje wolny.
