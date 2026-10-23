# FitCast – Špecifikácia požiadaviek

## Problém
Používatelia sa každý deň rozhodujú, čo si obliecť, bez jednoduchého prehľadu o svojom oblečení a jeho vhodnosti vzhľadom na vývoj počasia počas celého dňa.

## Používatelia a roly
- **Používateľ:** spravuje svoj digitálny šatník, zisťuje počasie a generuje návrhy outfitov.
- **Administrátor:** v MVP nie je potrebný; aplikácia funguje lokálne bez registrácie.

## Funkcie (MVP)
- **F1 – Pridanie oblečenia:** Ako používateľ chcem vybrať fotografiu z galérie a pridať kus oblečenia, aby som si vytvoril digitálny šatník.
- **F2 – Kategorizácia:** Ako používateľ chcem zaradiť oblečenie medzi vrchné diely, spodné diely, vrchné vrstvy a obuv, aby sa dalo použiť pri tvorbe outfitu.
- **F3 – Vlastnosti oblečenia:** Ako používateľ chcem označiť vhodné teploty a vhodnosť do dažďa, aby aplikácia vedela vyberať podľa počasia.
- **F4 – Prehľad šatníka:** Ako používateľ chcem vidieť fotografie a kategórie uložených kúskov, aby som mal prehľad o svojom oblečení.
- **F5 – Odstránenie:** Ako používateľ chcem odstrániť kus oblečenia, ktorý už nevlastním, aby šatník zostal aktuálny.
- **F6 – Poloha:** Ako používateľ chcem povoliť GPS alebo zadať mesto, aby aplikácia vedela získať miestne počasie.
- **F7 – Celodenná predpoveď:** Ako používateľ chcem vidieť hodinovú predpoveď teploty a zrážok na dnešný deň, aby som vedel, ako sa počasie zmení.
- **F8 – Návrh outfitu:** Ako používateľ chcem vygenerovať outfit zo svojho oblečenia podľa celodennej predpovede, aby som sa rýchlejšie rozhodol, čo si obliecť.
- **F9 – Iný outfit:** Ako používateľ chcem požiadať o alternatívnu kombináciu, aby som mal na výber.
- **F10 – Chybové stavy:** Ako používateľ chcem dostať jasné upozornenie, ak chýba vhodné oblečenie alebo počasie nie je dostupné, aby som vedel, ako pokračovať.

- **F11 – Vrstvenie:** Ako používateľ chcem, aby mi aplikácia pri výrazných zmenách teploty odporučila viacero kombinovateľných vrstiev, ktoré môžem počas dňa odložiť alebo znovu obliecť, aby bol outfit vhodný ráno aj popoludní.

## Akceptačné kritériá
- Nové oblečenie možno uložiť len s fotografiou, kategóriou a označením vhodných podmienok.
- Uložené oblečenie zostáva dostupné po reštartovaní aplikácie.
- Aplikácia žiada súhlas s polohou; pri zamietnutí ponúkne manuálne zadanie mesta.
- Pri chybe pripojenia alebo Weather API zobrazí zrozumiteľnú správu.
- Vygenerovaný outfit obsahuje vrchný diel, spodný diel a obuv; podľa chladu aj vrchnú vrstvu.
- Algoritmus vyhodnocuje hodinovú predpoveď na zostávajúcu časť dnešného dňa, najmä ranné, popoludňajšie a večerné teploty a zrážky; používa pravidlá, nie AI.
- Ak sa počas dňa výrazne mení teplota, navrhne kombinovateľné vrstvy: základný outfit vhodný na teplejšiu časť dňa a odnímateľnú vrstvu alebo vrstvy na chladnejšie obdobie.
- Používateľ vidí odporúčanie, ktoré vrstvy si má počas dňa odložiť alebo znovu obliecť.
- Ak nie sú dostupné vhodné vrstvy, aplikácia upozorní na obmedzenie návrhu.
- Ak chýba vhodná kategória oblečenia, aplikácia oznámi, čo chýba, namiesto neúplného návrhu.
- Alternatívny outfit ponúkne len vtedy, keď existuje iná vhodná kombinácia.

## Obmedzenia
- Primárne iOS; vývoj na Windows v React Native, Expo a TypeScript, testovanie na iPhone cez Expo Go.
- Jazyk rozhrania: slovenčina.
- Oblečenie a fotografie sa ukladajú lokálne v telefóne; bez používateľských účtov.
- Počasie sa načítava cez externé API (predbežne Open-Meteo), ktoré vyžaduje internet.
- Na výber fotografií treba prístup do galérie; GPS je voliteľné pri manuálnom zadaní mesta.
- Presnosť návrhov závisí od vlastností oblečenia zadaných používateľom.

## Mimo rozsahu MVP (non-goals)
- **AI rozpoznávanie oblečenia:** manuálna kategorizácia zjednoduší prvú verziu.
- **Hodnotenie farieb a módneho štýlu:** MVP rieši najmä vhodnosť podľa počasia.
- **Sociálna sieť a zdieľanie outfitov:** nesúvisia s hlavnou funkcionalitou.
- **Nakupovanie a e-shop:** cieľom je využívať existujúci šatník.
- **Registrácia a cloudová synchronizácia:** zvýšili by zložitosť bez potreby pre MVP.
- **Plánovanie outfitov na viac dní:** možné rozšírenie v budúcnosti.

## Otvorené otázky
1. **Rozhodnuté:** Výber vychádza z hodinovej predpovede na celý deň (pri spustení počas dňa zo zostávajúcich hodín). Aplikácia musí zohľadniť rozdiel medzi chladným ránom a teplým popoludním a odporučiť odnímateľné vrstvy.
2. Ako sa označí vhodnosť oblečenia? **Návrh:** teplotný interval a príznak „vhodné do dažďa“.
3. Aké teplotné hranice určujú potrebu bundy? **Návrh:** jednoduché pravidlá overené testovaním.
4. Má byť možnosť fotografovať priamo v aplikácii? **Návrh:** v MVP iba galéria.
5. Ako vybrať alternatívny outfit? **Návrh:** iná vhodná kombinácia bez bezprostredného opakovania.

## Kritérium dokončenia MVP
Používateľ dokáže pridať a uchovať oblečenie, prezerať šatník, získať celodennú hodinovú predpoveď podľa polohy alebo mesta a vygenerovať úplný outfit s vhodným vrstvením pri zmenách teploty; aplikácia správne ošetrí chýbajúce oblečenie a nedostupné počasie.
