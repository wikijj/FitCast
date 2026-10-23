# FitCast – Technický návrh riešenia (DESIGN.md)

**Stav:** návrh na schválenie  
**Východisko:** `SPEC.md` – špecifikácia požiadaviek FitCast  
**Cieľ:** poskytnúť jednoznačný technický podklad na implementáciu MVP, vrátane pravidiel celodenného vrstvenia.

## 1. Cieľ a rozsah

FitCast je mobilná aplikácia pre iOS, ktorá umožní vytvoriť lokálny digitálny šatník a odporučiť kompletný outfit na základe **hodinovej predpovede počasia na zostávajúcu časť dnešného dňa**. Pri zmenách teploty má preferovať vrstvenie: základný outfit vhodný na teplejšiu časť dňa a samostatne odnímateľné vrstvy na chladnejšie obdobie. Návrh musí vychádzať výlučne z oblečenia používateľa.

V MVP nebude registrácia, cloudová synchronizácia, AI rozpoznávanie fotografií, hodnotenie módnych kombinácií, e-shop ani plánovanie na ďalšie dni.

## 2. Technologický stack

| Oblasť | Návrh | Dôvod |
|---|---|---|
| Mobilná aplikácia | React Native + Expo + TypeScript | Vývoj na Windows, testovanie na iPhone cez Expo Go |
| Navigácia | Expo Router | Obrazovky a navigácia podľa súborov |
| UI | React Native komponenty, StyleSheet | Jednoduché, natívne použiteľné rozhranie |
| Výber fotografií | `expo-image-picker` | Fotografie z galérie |
| Práca s obrázkami | `expo-file-system` / kompatibilné API aktuálnej verzie Expo | Trvalé lokálne uloženie kópie obrázka |
| Poloha | `expo-location` | GPS po udelení súhlasu |
| Počasie | Open-Meteo Forecast API | Hodinová teplota, pravdepodobnosť zrážok a kód počasia |
| Vyhľadanie mesta | Open-Meteo Geocoding API | Manuálna lokalita pri zamietnutí GPS |
| Lokálne dáta | `expo-sqlite` | Trvalé uloženie oblečenia a jeho vlastností |
| Vývoj a verzovanie | VS Code, Git, GitHub | Priebežné commity do existujúceho repozitára |

**Poznámka:** Konkrétne verzie knižníc a názvy API pre prácu so súbormi treba overiť podľa aktuálne nainštalovanej verzie Expo SDK. `node_modules`, lokálne databázy a fotografie používateľa sa nesmú commitovať.

## 3. Architektúra

Aplikácia bude fungovať bez vlastného backendu.

```text
Používateľ
   │
   ▼
React Native UI (Expo Router)
   ├── Šatník ──> WardrobeService ──> SQLite + lokálne obrázky
   ├── Lokalita ──> LocationService ──> GPS / ručne zadané mesto
   ├── Počasie ──> WeatherService ──> Open-Meteo API
   └── Outfit ──> OutfitEngine
                     ├── uložené oblečenie
                     └── hodinová predpoveď
                          ▼
                   návrh vrstiev + odporúčania počas dňa
```

**Zodpovednosti modulov:** UI zobrazuje dáta a spracúva akcie používateľa; služby riešia lokálne úložisko, polohu a API; `OutfitEngine` je samostatná testovateľná funkcia bez závislosti od UI a internetu.

## 4. Obrazovky a navigácia

### 4.1 Home – domovská obrazovka
- Názov lokality a možnosť zmeniť mesto.
- Prehľad dnešnej hodinovej predpovede: teplota, pravdepodobnosť zrážok, orientačné podmienky.
- Stručná informácia o teplotnom rozpätí dňa, napr. „Ráno 7 °C, popoludní 21 °C“.
- Primárne tlačidlo **Navrhnúť outfit**.
- Pri chybe API vysvetlenie a tlačidlo **Skúsiť znova**.

### 4.2 My Wardrobe – digitálny šatník
- Zoznam/galéria fotografií oblečenia.
- Filtrovanie podľa kategórie.
- Akcie **Pridať oblečenie** a **Odstrániť** s potvrdením.
- Prázdny stav s výzvou na pridanie prvého kúsku.

### 4.3 Add Clothing – pridanie oblečenia
- Výber fotografie z galérie.
- Povinné polia: kategória, minimálna a maximálna vhodná teplota, vhodnosť do dažďa.
- Pre odnímateľné vrstvy typ vrstvy (napr. sveter, bunda) a informácia, že sa dajú nosiť cez základný vrchný diel.
- Validácia povinných polí a intervalu (`minTempC <= maxTempC`).
- Tlačidlá **Uložiť** a **Zrušiť**.

### 4.4 Outfit Recommendation – návrh outfitu
- Fotografie vybraných kúskov: základný vrchný diel, spodný diel, obuv a voliteľné vrstvy.
- Zrozumiteľné odporúčania podľa času: „Ráno si obleč aj sveter a bundu; popoludní ich môžeš odložiť.“
- Tlačidlo **Navrhnúť iný outfit**.
- Ak vhodná kombinácia neexistuje, zobrazenie chýbajúcich kategórií alebo vrstiev.

### 4.5 Location – výber lokality
- Tlačidlo **Použiť moju polohu** s vysvetlením súhlasu.
- Alternatíva: vyhľadať a potvrdiť mesto.
- Pri viacerých výsledkoch vyhľadávania umožniť vybrať správne mesto/krajinu.

**Navigácia:** spodné karty `Domov` a `Šatník`; `Pridať oblečenie`, `Výber lokality` a `Návrh outfitu` ako vnorené obrazovky.

## 5. Dátový model

### 5.1 ClothingItem

| Pole | Typ | Význam |
|---|---|---|
| `id` | string (UUID) | Identifikátor kúsku |
| `imageUri` | string | Lokálna cesta ku kópii fotografie |
| `category` | enum | `top`, `bottom`, `outer_layer`, `shoes` |
| `name` | string, voliteľné | Používateľský názov |
| `minTempC` | number | Dolná hranica vhodnej teploty |
| `maxTempC` | number | Horná hranica vhodnej teploty |
| `rainSuitable` | boolean | Vhodné do dažďa |
| `layerType` | enum/null | `sweater`, `jacket`, `coat` alebo null |
| `createdAt` | ISO datetime | Dátum pridania |

**Pravidlo:** `layerType` sa používa len pre `outer_layer`. V MVP sa vrstvy kombinujú v poradí základný vrchný diel → sveter → bunda/kabát. Priamo zadané teplotné intervaly sú orientačné, nie zdravotné odporúčania.

### 5.2 WeatherHour

| Pole | Typ | Význam |
|---|---|---|
| `time` | ISO datetime | Hodina v časovej zóne lokality |
| `temperatureC` | number | Predpovedaná teplota |
| `precipitationProbability` | number (0–100) | Pravdepodobnosť zrážok |
| `weatherCode` | number | Kód poveternostných podmienok |

### 5.3 LocationSelection

`latitude`, `longitude`, `displayName`, `timezone`, `source` (`gps` alebo `manual`). Uloží sa lokálne, aby používateľ nemusel zadávať mesto pri každom spustení.

### 5.4 OutfitSuggestion

`baseTopId`, `bottomId`, `shoesId`, `layerIds[]`, `instructions[]`, `weatherSummary`, `signature`. Tento objekt sa vypočíta z uložených kúskov a predpovede; v MVP sa nemusí ukladať do databázy.

## 6. Integrácia s počasím

1. Aplikácia načíta uloženú lokalitu alebo požiada o GPS. Ak používateľ GPS odmietne, ponúkne ručné zadanie mesta.
2. Pre vybranú polohu zavolá Open-Meteo Forecast API s hodinovými údajmi `temperature_2m`, `precipitation_probability`, `weather_code` a správnou časovou zónou.
3. Hodiny sa filtrujú podľa **lokálneho dátumu vybranej lokality**; pri otvorení počas dňa sa berú zostávajúce hodiny dneška.
4. Odpoveď sa validuje (existencia polí, rovnaká dĺžka časových radov, číselné hodnoty).
5. Pri nedostupnom internete/API sa zobrazí chyba a opakovanie načítania; staré dáta sa nesmú označiť za aktuálnu predpoveď.

**Technická poznámka:** V návrhu sa nepredpokladá vlastný server. Pred implementáciou overiť podmienky používania a limity zvoleného weather API.

## 7. Algoritmus výberu outfitu a vrstvenia

### 7.1 Základný princíp

Algoritmus musí vytvoriť **základný outfit vhodný na najteplejšiu časť zostávajúceho dňa** a doplniť ho o **odnímateľné vrstvy pre chladnejšie hodiny**. Základný outfit musí obsahovať `top`, `bottom`, `shoes`; vrstvy pochádzajú z `outer_layer`. Algoritmus nepoužíva AI ani internet, iba pripravené dáta.

### 7.2 Predspracovanie predpovede

- Z hodinových údajov urč `minTemp`, `maxTemp`, čas minima a maxima a hodiny s významnou pravdepodobnosťou dažďa.
- Pre MVP je navrhnutá hranica **teplotného rozdielu aspoň 6 °C** ako signál na osobitné posúdenie vrstvenia. Ide o konfigurovateľný návrh, ktorý treba potvrdiť testami.
- Počasie sa vyhodnocuje za všetky zostávajúce hodiny dneška; ak je deň už takmer na konci, algoritmus pracuje iba s dostupným zvyškom.

### 7.3 Výber oblečenia

1. Vyfiltruj kusy podľa kategórií.
2. Pre základný vrchný diel, spodný diel a obuv vyber kandidátov vhodných na `maxTemp`.
3. Skontroluj, či sú spodný diel a obuv použiteľné aj pri `minTemp`; ak nie, uprednostni kúsky s najväčším pokrytím teplotného rozsahu. Nevyhovujúcu kombináciu neprezentuj ako plne vhodnú.
4. Pre chladnejšie hodiny hľadaj odnímateľné vrstvy. Preferuj jednu vrstvu; ak nestačí a dostupné údaje to umožňujú, kombináciu svetra a bundy.
5. Pri riziku dažďa uprednostni vhodnú obuv a vrchnú vrstvu označenú `rainSuitable`. Ak taká neexistuje, upozorni na obmedzenie.
6. Vygeneruj pokyny podľa časových úsekov (ráno/popoludní/večer) podľa toho, ktoré vrstvy sú potrebné.
7. Ak chýba povinná kategória alebo dostatočné vrstvenie, zobraz konkrétny problém; nevymýšľaj chýbajúce oblečenie.

**Dôležitá implementačná zásada:** Vhodnosť kombinácie vrstiev nemožno spoľahlivo odvodiť len sčítaním teplotných intervalov jednotlivých kúskov. Pre MVP sa použijú transparentné pravidlá pre základný diel a voliteľné vrstvy; pri nejednoznačnosti aplikácia zobrazí orientačné odporúčanie alebo upozornenie. Teplotné hranice treba overiť s používateľom.

### 7.4 Návrh pseudokódu

```text
function generateOutfit(items, forecast, previousSignature):
    hours = remainingHoursOfToday(forecast)
    if hours is empty: return error("Predpoveď nie je dostupná")

    minTemp = min(hours.temperatureC)
    maxTemp = max(hours.temperatureC)
    rainRisk = some(hours.precipitationProbability >= 50)

    bases = findBaseCombinations(items, maxTemp, minTemp, rainRisk)
    if bases is empty: return missingCategoriesOrSuitableItems()

    candidates = []
    for base in bases:
        layers = findCompatibleRemovableLayers(items, base, minTemp, maxTemp, rainRisk)
        candidate = evaluateAcrossHours(base, layers, hours)
        if candidate.meetsMinimumRequirements:
            candidates.append(candidate)

    if candidates is empty: return insufficientLayersOrCoverage()

    ranked = sortByWeatherCoverageAndSimplicity(candidates)
    selected = firstWithDifferentSignature(ranked, previousSignature)
    if selected is null: return noAlternativeAvailable()
    return selected.withInstructionsForHours(hours)
```

Prahová hodnota dažďa **50 %** a teplotný rozdiel **6 °C** sú návrhové parametre, nie definitívne požiadavky zo `SPEC.md`.

### 7.5 Príklad

Hodinová predpoveď: 08:00 – 7 °C; 13:00 – 21 °C; 19:00 – 13 °C. V šatníku sú tričko, sveter, ľahká bunda, rifle a tenisky. FitCast odporučí:

- **Ráno:** tričko + sveter + bunda + rifle + tenisky.
- **Popoludní:** tričko + rifle + tenisky; sveter a bundu odložiť.
- **Večer:** podľa ochladenia znovu pridať sveter alebo bundu.

Ak používateľ nemá vhodnú bundu alebo sveter, aplikácia musí túto skutočnosť oznámiť, nie odporučiť neexistujúci kus.

## 8. Alternatívne outfity

Každý návrh má `signature` vytvorený z usporiadaných ID vybraných kúskov. Pri kliknutí na **Navrhnúť iný outfit** sa vyberie ďalší vhodný kandidát s iným podpisom. Ak existuje iba jeden vhodný návrh, tlačidlo zobrazí informáciu „Ďalší vhodný outfit momentálne nemáš.“ Náhodnosť je voliteľná; prednosť má predvídateľné poradie podľa vhodnosti.

## 9. Lokálne ukladanie a práca s fotografiami

- Pri výbere obrázka sa vytvorí **kópia do trvalého priečinka aplikácie**; do SQLite sa uloží iba lokálna URI.
- Po uložení musí byť obrázok dostupný aj po reštarte aplikácie.
- Pri odstránení oblečenia sa odstráni databázový záznam a príslušný lokálny obrázok, pokiaľ ho nepoužíva iný záznam.
- Pri zlyhaní kopírovania obrázka alebo zápisu do databázy sa používateľovi zobrazí chyba a nesmie vzniknúť neúplný záznam.
- Poloha, fotografie ani šatník sa neodosielajú na vlastný server. Externému weather API sa posielajú iba údaje potrebné na predpoveď pre vybranú polohu.

## 10. Chybové a okrajové stavy

| Situácia | Správanie aplikácie |
|---|---|
| Zamietnuté GPS | Ponúknuť ručné zadanie mesta |
| Mesto sa nenašlo | Upozorniť a umožniť nové vyhľadávanie |
| API neodpovedá | Zobraziť chybu a opakovanie |
| Prázdny šatník | Výzva na pridanie oblečenia |
| Chýba vrchný diel/spodný diel/obuv | Nevygenerovať neúplný outfit; vypísať chýbajúce kategórie |
| Veľký rozdiel teplôt bez vhodných vrstiev | Upozorniť, že šatník nepokrýva celý deň |
| Riziko dažďa bez vhodného oblečenia | Upozorniť na chýbajúcu ochranu pred dažďom |
| Iba jedna kombinácia | Oznámiť, že alternatíva nie je dostupná |
| Chybný teplotný interval | Zablokovať uloženie a vysvetliť chybu |
| Chýbajúca alebo odstránená fotografia | Zobraziť náhradný obrázok a umožniť opravu/odstránenie |

## 11. Navrhovaná štruktúra projektu

```text
FitCast/
├── project.md
├── SPEC.md
├── DESIGN.md
├── app/
│   ├── _layout.tsx
│   ├── (tabs)/
│   │   ├── index.tsx            # Domov
│   │   └── wardrobe.tsx        # Šatník
│   ├── add-clothing.tsx
│   ├── location.tsx
│   └── outfit.tsx
├── src/
│   ├── components/             # Znovupoužiteľné UI prvky
│   ├── models/                 # Typy a dátové modely
│   ├── services/               # Weather, Location, Wardrobe
│   ├── storage/                # SQLite a lokálne fotografie
│   ├── logic/                  # OutfitEngine a pravidlá vrstvenia
│   └── constants/              # Konfigurovateľné hranice
├── assets/
├── __tests__/
├── package.json
└── app.json
```

**Poznámka:** Ide o cieľovú štruktúru, nie o tvrdenie, že súbory už existujú. Ak použitá šablóna Expo vytvorí inú štruktúru, prispôsobiť ju bez zbytočného prepisovania projektu.

## 12. Implementačný postup

1. **Inicializácia:** Expo + TypeScript, navigácia, základný dizajn, prvý commit.
2. **Šatník:** SQLite, výber a trvalé uloženie fotografie, formulár, galéria, odstránenie.
3. **Lokalita a počasie:** GPS, vyhľadávanie mesta, Open-Meteo, validácia hodinovej predpovede.
4. **OutfitEngine:** výber základného outfitu, vrstvenie, dážď, alternatívy, chybové stavy.
5. **Prezentácia odporúčania:** fotografie kúskov a pokyny na pridávanie/odkladanie vrstiev podľa času.
6. **Testovanie a finalizácia:** jednotkové testy algoritmu, manuálne testy na iPhone, opravy a finálne commity.

Odporúčané samostatné commity: `Initialize Expo app`, `Implement local wardrobe`, `Integrate daily weather forecast`, `Implement layered outfit engine`, `Add outfit recommendation screen`, `Add tests and error handling`.

## 13. Testovací plán a väzba na SPEC.md

| Test | Vstup / scenár | Očakávaný výsledok | Požiadavky |
|---|---|---|---|
| T1 | Pridanie platného oblečenia | Zobrazí sa v šatníku aj po reštarte | F1–F4 |
| T2 | Pridanie bez kategórie | Uloženie sa zablokuje | F1–F3 |
| T3 | Odstránenie oblečenia | Zmizne zo šatníka | F5 |
| T4 | GPS zamietnuté | Možnosť zadať mesto | F6 |
| T5 | Hodinová predpoveď | Zobrazia sa hodiny dnešného dňa | F7 |
| T6 | Stabilná teplota celý deň | Kompletný outfit bez zbytočných vrstiev | F8 |
| T7 | Ráno 7 °C, obed 21 °C | Základný outfit + odnímateľné vrstvy a pokyny | F8, F11 |
| T8 | Popoludní dážď | Preferencia vhodnej vrstvy/obuvi alebo upozornenie | F8, F11 |
| T9 | Chýbajú topánky | Informácia o chýbajúcej kategórii | F10 |
| T10 | Chýba vhodná bunda pri chlade | Upozornenie na nedostatočné vrstvenie | F10, F11 |
| T11 | Existujú dva vhodné outfity | Alternatíva sa odlišuje | F9 |
| T12 | Existuje len jeden outfit | Oznámenie o nedostupnej alternatíve | F9 |
| T13 | Výpadok internetu/API | Chybová správa, žiadne falošne aktuálne počasie | F7, F10 |
| T14 | Spustenie večer | Použijú sa iba zostávajúce hodiny dneška | F7, F8 |

**Kritérium dokončenia:** Všetky povinné funkcie F1–F11 podľa `SPEC.md` fungujú, kritické testy prejdú a aplikácia je použiteľná na reálnom iPhone.

## 14. Rozhodnutia na schválenie pred implementáciou

1. **Teplotné intervaly oblečenia:** používateľ zadáva minimálnu a maximálnu teplotu (navrhnuté, ešte nepotvrdené).
2. **Hranica vrstvenia:** rozdiel 6 °C (navrhnuté, treba overiť).
3. **Riziko dažďa:** pravdepodobnosť aspoň 50 % (navrhnuté, treba overiť).
4. **Rozsah dňa:** od aktuálnej hodiny do konca miestneho dňa (podľa `SPEC.md`).
5. **Fotografie:** v MVP len výber z galérie (podľa `SPEC.md` ako návrh).
6. **Algoritmus:** pravidlá bez AI; uprednostnenie úplného pokrytia počasia pred náhodným výberom.

**Pravidlo pre AI agenta:** Pred programovaním nejasných bodov zo sekcie 14 požiadať o potvrdenie. Nevymýšľať nové funkcie mimo `SPEC.md`; pri konflikte medzi dokumentmi má prednosť schválený `SPEC.md` a zmenu treba výslovne zaznamenať.
