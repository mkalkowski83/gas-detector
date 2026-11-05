# Gas detector

[Polska wersja](README.md)

Schematic of a prototype built on a **perfboard**: an **MH-MQ** module (MQ element + LM393 comparator) and an **ESP32 DevKit V1** (30 pins). Power comes in through the USB cable into the ESP32. The sensor module takes **5 V from the 5V/VIN pin**, because the MQ heater cannot run from 3.3 V.

There is no PCB. Footprints are empty. Soldering on the perfboard is a separate step.

Schematic: [`docs/kicad/gas-detector-mq/gas-detector-mq.kicad_sch`](docs/kicad/gas-detector-mq/gas-detector-mq.kicad_sch)

ESPHome configuration: [`esphome/gas-sensor.yaml`](esphome/gas-sensor.yaml). AO on GPIO34, green LED on GPIO23, red LED on GPIO22, buzzer on GPIO25, threshold 0.35 V. Secrets go to `esphome/secrets.yaml`, following [`esphome/secrets.yaml.example`](esphome/secrets.yaml.example). The secrets file is not committed.

## Connections

| From | To | What it is |
|------|----|------------|
| USB | USB connector on the DevKit (U1) | power for everything |
| U2 **VCC** | U1 **5V/VIN**, plus C1 and C2 | `+5V` net |
| U2 **GND** | DevKit ground and divider ground | common ground |
| U2 **AO** | R1 **10 kΩ** | analog signal, up to 5 V |
| node after R1 (`AO_3V3`) | R2 **20 kΩ** to ground and U1 **D34** (GPIO34) | ADC1, about 3.33 V |
| U2 **DO** | R3 **10 kΩ** | digital threshold from LM393, up to 5 V |
| node after R3 (`DO_3V3`) | R4 **20 kΩ** to ground and U1 **D35** (GPIO35) | digital input, about 3.33 V |
| C1, C2 | between `+5V` and ground, next to the module | 100 nF and 10 µF |
| U1 **GPIO23** (D23) | R5 **220 Ω**, then D1 anode | green, no leak |
| D1 cathode | ground | short LED leg |
| U1 **GPIO22** (D22) | R6 **220 Ω**, then D2 anode | red, leak |
| D2 cathode | ground | short LED leg |
| U1 **GPIO25** | R7 **1 kΩ**, Q1 base | buzzer control |
| Q1 emitter | ground | BC547, E leg |
| Q1 collector | BZ1 minus and D3 anode | buzzer node |
| BZ1 plus and D3 cathode | `+5V` | TMB12A05, 1N4002 stripe towards +5 V |

On the ESP32 symbol, a cross marks a pin that is **not soldered**. **D34** (GPIO34, ADC1_CH6) and **D35** (GPIO35, ADC1_CH7) are input-only. Both are on ADC1, so Wi-Fi does not block them. ADC2 cannot be used for measurements while the radio is on.

MQ symbol pin order, top to bottom: **VCC, GND, DO, AO**. On the ESP32 (USB at the bottom): left row from the top `3V3, EN, GPIO36/VP, GPIO39/VN, GPIO34, GPIO35, GPIO32, …`, right row from the top `5V/VIN, GND, GPIO23, …`. Compare this with the silkscreen on your board before soldering.

## Divider

AO and DO of the MH module go up to 5 V (AO from the analog path, DO from the LM393 pulled up to VCC on the module). ESP32 GPIOs tolerate about 3.6 V. In series there is **10 kΩ**, to ground **20 kΩ**:

`5 V × 20 / (10 + 20) = 3.33 V`

The 20 kΩ to ground pulls the pin to zero if the wire comes loose. D34 and D35 have no internal pull-ups. The nearest E12 value is 22 kΩ (about 3.44 V, still below 3.6 V); the schematic uses 20 kΩ. When reading AO, set the ADC attenuation to the maximum (`12db` in ESPHome, formerly called 11 dB) — 3.33 V is near the top of the range.

## Heater and USB

The MQ heater draws about **150 mA at 5 V**. The **5V/VIN** pin is on the USB side, before the 3.3 V regulator (on some boards through a Schottky diode). The 3.3 V regulator cannot supply this heater. The USB port also has to handle ESP32 current spikes during Wi-Fi, often over 400 mA together with the heater. A weak port or a thin cable ends in reboots.

C1 **100 nF** (ceramic) and C2 **10 µF / 50 V** sit next to the module's VCC. C2 plus goes to `+5V`. 10 µF is enough as a local reservoir next to the 100 nF: the heater draws current continuously from USB, not from the capacitor. Do not use 1000 µF / 6.3 V here — the voltage is just above 5 V, and a large capacitance on the USB line causes an inrush current when the cable is plugged in.

## Warm-up

After power-up, the MQ element has to heat up before AO makes sense. Usually a few to a dozen or so minutes. A new sensor often needs a longer burn-in, even around 24–48 h, before the reading settles. The exact time depends on the MQ-x variant, which is unknown here. The DO threshold is set with the potentiometer on the module.

## Parts

| Ref | Part | Value |
|-----|------|-------|
| U1 | ESP32 DevKit V1, 30 pin (DOIT) | module, powered from USB |
| U2 | MH-Sensor-Series module (MQ + LM393) | MQ-x variant unknown |
| R1, R3 | resistor | 10 kΩ |
| R2, R4 | resistor | 20 kΩ |
| C1 | ceramic | 100 nF |
| C2 | electrolytic | 10 µF / 50 V, plus to +5 V |
| R5, R6 | resistor | 220 Ω |
| R7 | resistor | 1 kΩ |
| D1 | green LED | anode from GPIO23 via R5, cathode to ground |
| D2 | red LED | anode from GPIO22 via R6, cathode to ground |
| Q1 | BC547 NPN | base via R7, emitter to ground, collector to BZ1 |
| BZ1 | TMB12A05 | 5 V active buzzer, plus to +5 V |
| D3 | 1N4002 | in parallel with BZ1, stripe towards +5 V |

## Alarm

The threshold is **0.35 V** on GPIO34. Clean air after a day of heating sits at about 0.16 V. Gas from a lighter raised the reading to 0.41 V and above.

| State | D1 green, GPIO23 | D2 red, GPIO22 | BZ1, GPIO25 |
|-------|------------------|----------------|-------------|
| below 0.35 V | on | off | silent |
| 0.35 V and above | off | on | beeping |

D1 and D2 are driven from the 3.3 V pins, about 6 mA through 220 Ω. Cathode (short leg) to ground.

The TMB12A05 is a **5 V** active buzzer, up to **30 mA**. An ESP pin cannot drive that, so Q1 (BC547) switches it on the ground side. BZ1 plus to `+5V`, minus to the collector. D3 (1N4002) in parallel with the buzzer, stripe towards `+5V`. GPIO25 high turns the sound on. For the BC547, flat side facing you and legs down, the order is **C B E**.

## Assumptions

- The module is a typical MH board: four pins **VCC, GND, DO, AO**. Some clones use a different order — the silkscreen is what counts.
- DO goes up to 5 V, because the LM393 on these modules is pulled up to VCC.
- The board is a **30-pin DOIT DevKit V1**, not a 38-pin ESP32-DevKitC. On the DevKitC the 5 V pin may be in a different place.
- AO (A0) → D34 (GPIO34), DO (D0) → D35 (GPIO35). Both pins are input-only, which is enough for reading DO.
- The bulk capacitor is 10 µF / 50 V (instead of the previously assumed 47 µF). Do not use 1000 µF / 6.3 V on this rail.
- Heater current of about 150 mA comes from the module description, not from the datasheet of a specific MQ-x.
- Alarm at **0.35 V** on GPIO34. D1 on GPIO23 (D23), D2 on GPIO22 (D22), BZ1 via Q1 from GPIO25. GPIO32 and GPIO33 stay free. The TMB12A05 is an active buzzer: a steady 5 V turns the sound on, no PWM.

## Left for soldering

Lay out the modules on the perfboard, check the VIN/5V silkscreen and the VCC-GND-DO-AO order, solder the pin headers, resistors, diodes, transistor and buzzer according to the table. Keep the dividers short, close to the ESP32 pins. Common ground. TMB12A05 plus to `+5V`. Do not connect the pins marked with a cross, in particular GPIO12 (must stay low at boot) and GPIO0. No PCB design is planned.
