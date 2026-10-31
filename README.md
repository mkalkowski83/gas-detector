# Czujnik gazu

Schemat ideowy prototypu na **płytce uniwersalnej**: moduł **MH-MQ** (element MQ + komparator LM393) i **ESP32 DevKit V1** (30 pinów). Zasilanie wchodzi kablem USB w ESP32. Moduł czujnika bierze **5 V z pinu 5V/VIN**, bo grzałka MQ nie może wisieć na 3,3 V.

Płytki drukowanej nie ma. Footprinty są puste. Lutowanie na płytce uniwersalnej jest osobnym krokiem.

Schemat: [`docs/kicad/gas-detector-mq/gas-detector-mq.kicad_sch`](docs/kicad/gas-detector-mq/gas-detector-mq.kicad_sch)

## Połączenia

| Skąd | Dokąd | Co to jest |
|------|--------|------------|
| USB | złącze USB na DevKit (U1) | zasilanie całości |
| U2 **VCC** | U1 **5V/VIN**, plus C1 i C2 | sieć `+5V` |
| U2 **GND** | masa DevKit i dzielników | wspólna masa |
| U2 **AO** | R1 **10 kΩ** | sygnał analogowy, do 5 V |
| węzeł za R1 (`AO_3V3`) | R2 **20 kΩ** do masy oraz U1 **D34** (GPIO34) | ADC1, około 3,33 V |
| U2 **DO** | R3 **10 kΩ** | próg cyfrowy z LM393, do 5 V |
| węzeł za R3 (`DO_3V3`) | R4 **20 kΩ** do masy oraz U1 **D35** (GPIO35) | wejście cyfrowe, około 3,33 V |
| C1, C2 | między `+5V` a masą, przy module | 100 nF i 47 µF |

Na symbolu ESP32 krzyżyk oznacza pin, którego **nie lutujemy**. **D34** (GPIO34, ADC1_CH6) i **D35** (GPIO35, ADC1_CH7) są tylko wejściami. Oba są w ADC1, więc Wi-Fi ich nie blokuje. ADC2 przy włączonym radiu nie nadaje się do pomiaru.

Kolejność pinów na symbolu MQ, od góry: **VCC, GND, DO, AO**. Na ESP32 (USB na dole): lewy rząd od góry `3V3, EN, GPIO36/VP, GPIO39/VN, GPIO34, GPIO35, GPIO32, …`, prawy rząd od góry `5V/VIN, GND, GPIO23, …`. Przed lutowaniem porównaj to z nadrukiem na swojej płytce.

## Dzielnik

AO i DO modułu MH wychodzą do 5 V (AO z toru analogowego, DO z LM393 podciągniętego na module do VCC). GPIO ESP32 wytrzymuje około 3,6 V. Szeregowo jest **10 kΩ**, do masy **20 kΩ**:

`5 V × 20 / (10 + 20) = 3,33 V`

20 kΩ do masy ściąga pin do zera, gdy przewód odpadnie. D34 i D35 nie mają wewnętrznego podciągania. Najbliższa wartość z szeregu E12 to 22 kΩ (około 3,44 V, nadal poniżej 3,6 V); na schemacie są 20 kΩ. Przy odczycie AO ustaw tłumienie ADC 11 dB — 3,33 V jest przy górze skali.

## Grzałka i USB

Grzałka MQ bierze około **150 mA z 5 V**. Pin **5V/VIN** jest po stronie USB, przed stabilizatorem 3,3 V (na części płytek przez diodę Schottky’ego). Stabilizator 3,3 V tej grzałki nie uciągnie. Port USB musi jeszcze znieść skoki prądu ESP32 przy Wi-Fi, często ponad 400 mA razem z grzałką. Słaby port albo cienki kabel kończy się restartami.

C1 **100 nF** (ceramika) i C2 **47 µF / 50 V** siedzą przy VCC modułu. Plus C2 do `+5V`. 47 µF wygładza zasilanie przy module, a prąd grzałki i tak idzie stale z USB. Na tej szynie trzymaj się okolicy 47 µF. 470 µF i 1000 µF dają udar prądu przy wpięciu kabla, a kondensator na 6,3 V jest za nisko na 5 V.

## Wygrzewanie

Element MQ po podaniu zasilania musi się rozgrzać, zanim AO ma sens. Zwykle kilka do kilkunastu minut. Nowy czujnik często potrzebuje dłuższego wygrzewania (burn-in), nawet około 24–48 h, zanim wskazanie się uspokoi. Dokładny czas zależy od wersji MQ-x, a tej wersji tu nie znamy. Próg DO kręci się potencjometrem na module.

## Elementy

| Ref | Element | Wartość |
|-----|---------|---------|
| U1 | ESP32 DevKit V1, 30 pin (DOIT) | moduł, zasilany z USB |
| U2 | moduł MH-Sensor-Series (MQ + LM393) | MQ-x nieustalony |
| R1, R3 | rezystor | 10 kΩ |
| R2, R4 | rezystor | 20 kΩ |
| C1 | ceramika | 100 nF |
| C2 | elektrolit THT | 47 µF / 50 V, plus do +5 V |

Buzzer i LED nie wchodzą w ten schemat.

## Założenia

- Moduł to typowa płytka MH: cztery piny **VCC, GND, DO, AO**. Część klonów ma inną kolejność — liczy się nadruk.
- DO idzie do 5 V, bo LM393 na tych modułach jest podciągnięty do VCC.
- Płytka to **30-pinowy DOIT DevKit V1**, nie 38-pinowy ESP32-DevKitC. Na DevKitC pin 5 V bywa w innym miejscu.
- AO (A0) → D34 (GPIO34), DO (D0) → D35 (GPIO35). Oba piny są tylko wejściami, co do odczytu DO wystarcza.
- Bulk C2 to elektrolit 47 µF / 50 V, plus do +5 V, obok ceramiki 100 nF.
- Prąd grzałki około 150 mA, według opisu, nie z noty konkretnego MQ-x.

## Zostaje na lutowanie

Ułożyć moduły na płytce uniwersalnej, sprawdzić nadruk VIN/5V i kolejność VCC-GND-DO-AO, zlutować goldpiny, rezystory i kondensatory według tabeli. Dzielniki trzymać krótko przy pinach ESP32. Wspólna masa. Nie podłączać pinów z krzyżykiem, w szczególności GPIO12 (przy starcie musi zostać w stanie niskim) i GPIO0. Projektu PCB nie rysujemy.
