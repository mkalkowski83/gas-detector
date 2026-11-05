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
| C1, C2 | między `+5V` a masą, przy module | 100 nF i 10 µF |
| U1 **GPIO23** (D23) | R5 **220 Ω**, potem anoda D1 | zielona, brak wycieku |
| katoda D1 | masa | krótka noga diody |
| U1 **GPIO22** (D22) | R6 **220 Ω**, potem anoda D2 | czerwona, wyciek |
| katoda D2 | masa | krótka noga diody |
| U1 **GPIO25** | R7 **1 kΩ**, baza Q1 | sterowanie brzęczyka |
| emiter Q1 | masa | BC547, noga E |
| kolektor Q1 | minus BZ1 i anoda D3 | węzeł brzęczyka |
| plus BZ1 i katoda D3 | `+5V` | TMB12A05, pasek 1N4002 przy +5 V |

Na symbolu ESP32 krzyżyk oznacza pin, którego **nie lutujemy**. **D34** (GPIO34, ADC1_CH6) i **D35** (GPIO35, ADC1_CH7) są tylko wejściami. Oba są w ADC1, więc Wi-Fi ich nie blokuje. ADC2 przy włączonym radiu nie nadaje się do pomiaru.

Kolejność pinów na symbolu MQ, od góry: **VCC, GND, DO, AO**. Na ESP32 (USB na dole): lewy rząd od góry `3V3, EN, GPIO36/VP, GPIO39/VN, GPIO34, GPIO35, GPIO32, …`, prawy rząd od góry `5V/VIN, GND, GPIO23, …`. Przed lutowaniem porównaj to z nadrukiem na swojej płytce.

## Dzielnik

AO i DO modułu MH wychodzą do 5 V (AO z toru analogowego, DO z LM393 podciągniętego na module do VCC). GPIO ESP32 wytrzymuje około 3,6 V. Szeregowo jest **10 kΩ**, do masy **20 kΩ**:

`5 V × 20 / (10 + 20) = 3,33 V`

20 kΩ do masy ściąga pin do zera, gdy przewód odpadnie. D34 i D35 nie mają wewnętrznego podciągania. Najbliższa wartość z szeregu E12 to 22 kΩ (około 3,44 V, nadal poniżej 3,6 V); na schemacie są 20 kΩ. Przy odczycie AO ustaw tłumienie ADC 11 dB — 3,33 V jest przy górze skali.

## Grzałka i USB

Grzałka MQ bierze około **150 mA z 5 V**. Pin **5V/VIN** jest po stronie USB, przed stabilizatorem 3,3 V (na części płytek przez diodę Schottky’ego). Stabilizator 3,3 V tej grzałki nie uciągnie. Port USB musi jeszcze znieść skoki prądu ESP32 przy Wi-Fi, często ponad 400 mA razem z grzałką. Słaby port albo cienki kabel kończy się restartami.

C1 **100 nF** (ceramika) i C2 **10 µF / 50 V** siedzą przy VCC modułu. Plus C2 do `+5V`. 10 µF starcza jako lokalny magazyn obok 100 nF: grzałka bierze prąd stale z USB, a nie z kondensatora. Nie wstawiać tu 1000 µF / 6,3 V — napięcie jest tuż nad 5 V, a duża pojemność na linii USB daje udar prądu przy wpięciu kabla.

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
| C2 | elektrolit | 10 µF / 50 V, plus do +5 V |
| R5, R6 | rezystor | 220 Ω |
| R7 | rezystor | 1 kΩ |
| D1 | LED zielona | anoda od GPIO23 przez R5, katoda do masy |
| D2 | LED czerwona | anoda od GPIO22 przez R6, katoda do masy |
| Q1 | BC547 NPN | baza przez R7, emiter do masy, kolektor do BZ1 |
| BZ1 | TMB12A05 | brzęczyk aktywny 5 V, plus do +5 V |
| D3 | 1N4002 | równolegle do BZ1, pasek do +5 V |

## Alarm

Próg to **0,35 V** na GPIO34. Czyste powietrze po dobie grzania stoi przy około 0,16 V. Gaz z zapalniczki podnosił odczyt do 0,41 V i wyżej.

| Stan | D1 zielona, GPIO23 | D2 czerwona, GPIO22 | BZ1, GPIO25 |
|------|--------------------|---------------------|-------------|
| poniżej 0,35 V | świeci | zgaszona | cicho |
| 0,35 V i więcej | zgaszona | świeci | piszczy |

D1 i D2 biorą prąd z pinów 3,3 V, około 6 mA przez 220 Ω. Katoda (krótka noga) do masy.

TMB12A05 to brzęczyk aktywny na **5 V**, do **30 mA**. Pin ESP tego nie uciągnie, więc Q1 (BC547) włącza go od strony masy. Plus BZ1 do `+5V`, minus do kolektora. D3 (1N4002) równolegle do brzęczyka, pasek od strony `+5V`. GPIO25 wysoki włącza dźwięk. Przy BC547, płaską stroną do siebie i nogami w dół, kolejność to **C B E**.

## Założenia

- Moduł to typowa płytka MH: cztery piny **VCC, GND, DO, AO**. Część klonów ma inną kolejność — liczy się nadruk.
- DO idzie do 5 V, bo LM393 na tych modułach jest podciągnięty do VCC.
- Płytka to **30-pinowy DOIT DevKit V1**, nie 38-pinowy ESP32-DevKitC. Na DevKitC pin 5 V bywa w innym miejscu.
- AO (A0) → D34 (GPIO34), DO (D0) → D35 (GPIO35). Oba piny są tylko wejściami, co do odczytu DO wystarcza.
- Bulk to 10 µF / 50 V (zamiast przyjętych wcześniej 47 µF). 1000 µF / 6,3 V nie używać na tej szynie.
- Prąd grzałki około 150 mA, według opisu, nie z noty konkretnego MQ-x.
- Alarm przy **0,35 V** na GPIO34. D1 na GPIO23 (D23), D2 na GPIO22 (D22), BZ1 przez Q1 z GPIO25. GPIO32 i GPIO33 zostają wolne. TMB12A05 jest brzęczykiem aktywnym: stałe 5 V włącza dźwięk, bez PWM.

## Zostaje na lutowanie

Ułożyć moduły na płytce uniwersalnej, sprawdzić nadruk VIN/5V i kolejność VCC-GND-DO-AO, zlutować goldpiny, rezystory, diody, tranzystor i brzęczyk według tabeli. Dzielniki trzymać krótko przy pinach ESP32. Wspólna masa. Plus TMB12A05 do `+5V`. Nie podłączać pinów z krzyżykiem, w szczególności GPIO12 (przy starcie musi zostać w stanie niskim) i GPIO0. Projektu PCB nie rysujemy.
