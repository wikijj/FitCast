# FitCast

## Aký problém riešim?

FitCast rieši problém každodenného rozhodovania, čo si obliecť, tým, že
používateľovi navrhne vhodný outfit z jeho vlastného šatníka podľa
aktuálneho počasia.

## Kto sú moji používatelia?

Používateľmi sú ľudia, ktorí chcú mať prehľad o svojom oblečení a
rýchlejšie si vybrať vhodný outfit podľa počasia.

## Ako chcem problém vyriešiť -- MVP

Minimálna verzia aplikácie bude obsahovať: - vytvorenie digitálneho
šatníka, - pridanie oblečenia pomocou fotografie, - základnú
kategorizáciu oblečenia (napr. tričko, nohavice, bunda, obuv), -
získanie aktuálnej polohy používateľa, - získanie informácií o počasí
pre danú polohu, - jednoduchý algoritmus, ktorý na základe počasia
vyberie vhodné kúsky z digitálneho šatníka a vytvorí návrh outfitu, -
možnosť vygenerovať ďalší vhodný outfit.

Príklad: Ak je vonku 8 °C a prší, aplikácia uprednostní dlhé nohavice,
teplejší vrch, bundu a obuv vhodnú do dažďa.

## Čo v MVP nebude (non-goals) a prečo?

V prvej verzii aplikácie nebude: - AI rozpoznávanie oblečenia z
fotografie -- používateľ si kategóriu oblečenia vyberie manuálne, aby
bola implementácia jednoduchšia, - automatické hodnotenie farebných
kombinácií -- nie je potrebné na overenie hlavnej funkcionality, -
sociálna sieť ani zdieľanie outfitov s ostatnými používateľmi --
nesúvisí priamo s hlavným problémom aplikácie, - nakupovanie alebo
odporúčanie nového oblečenia -- cieľom je využívať oblečenie, ktoré už
používateľ vlastní, - pokročilá personalizácia pomocou AI -- MVP najprv
overí, či funguje základný výber outfitu podľa počasia.

Tieto funkcie môžu byť doplnené v ďalších verziách aplikácie.

## Aké technológie potrebujem?

-   mobilný framework / technológia na vývoj mobilnej aplikácie,
-   fotoaparát telefónu -- na pridávanie fotografií oblečenia,
-   GPS -- na získanie polohy používateľa,
-   Weather API -- na získanie aktuálneho počasia podľa polohy,
-   lokálna databáza -- na uloženie jednotlivých kúskov oblečenia a ich
    kategórií,
-   internetové pripojenie -- na komunikáciu s Weather API,
-   Git a GitHub -- na verzovanie zdrojového kódu a správu repozitára.
