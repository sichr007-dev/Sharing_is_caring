# Výstup a mandát auditu

Audit bez pevného tvaru výstupu sklouzne k seznamu obvinění: nesouhlas s architektonickým
rozhodnutím se prodává jako vada, nulový nález vypadá jako opomenutí a položka čekající na
vlastníka jako nesplněný úkol auditora. Tahle reference je protokol, který tomu brání.
Nestojí nic a je plně přenositelný mezi projekty.

## 1. Třídění — každá položka patří právě do jedné třídy

| Třída | Význam | Jak se s ní nakládá |
|---|---|---|
| **Vada** | Systém dělá něco špatně. | Reprodukce, závažnost, stav. Vyžaduje rozhodnutí o opravě. |
| **Dluh** | Funguje, ale zdražuje každou další změnu. | Vede se s odhadem dopadu a pořadím. Nikdy se neprodává jako vada. |
| **Rozhodnutí** | Vypadá jako vada, ale je to vědomá volba s doloženým zdůvodněním (ADR, invarianty). | Hlásí se jako informace s odkazem na zdroj a s popisem ceny, kterou nese. Otázka ownerovi zní „platí ještě?", ne „opravit". |
| **Provoz** | Netýká se kódu, ale prostředí (včetně prostředí auditu samotného). | Vede se odděleně, jinak se ztratí mezi kódovými nálezy. |
| **Prověřeno bez nálezu** | Otázka byla položena, odpověď je „čisté" — s důkazem práce. | Dostává číslo. Bez toho není vidět rozsah auditu a příští běh to dělá znovu. |

Poznávací znamení špatného třídění: report, kde je všechno „vada". Poznávací znamení
dobrého: kapitola **„co vypadá jako nález a není"** (ratifikované vzory, záměrné rozpory,
věci, které FK mít nemohou, protože identity přijdou později).

## 2. Stavy — čtyři, ne dva

**opraveno · otevřené · blokované rozhodnutím vlastníka · vědomě neděláme**

Poslední dva odlišují report od seznamu přání:

- **Blokované rozhodnutím** není nesplněný úkol auditora — je to úkol pro vlastníka a musí být
  v reportu vidět jako takový (typicky: cvičná obnova zálohy bez zadaných cílů obnovy).
- **Vědomě neděláme** má vlastní kapitolu: co se nemá refaktorovat a s jakou cenou by změna
  přišla. Bez ní klient nerozezná opomenutí od záměru.
- **Otevřené** zůstává otevřené, i když se opravovalo: co bylo vyzkoušeno a proč to nestačilo.
  Nedořešená položka označená jako vyřešená je nejdražší varianta falešného PASS.

## 3. Jedna identifikační řada

- Jedna řada pro celý projekt, průběžně číslovaná, napříč všemi vlnami a všemi reporty.
- Prefixy pro **činnosti** (prověrky, měření) se nikdy nepoužívají pro **nálezy**. Míchání
  obou v jedné řadě způsobuje kolize, kdy totéž označení znamená ve dvou dokumentech dvě věci.
- Přečíslování se nedělá. Když se řada mění, vzniká překladová tabulka starých a nových
  označení — jinak se starší reporty stanou nečitelnými. Totéž platí pro katalogy vzorů.
- Konsolidační dokument je zdroj pravdy pro rozhodování; dílčí reporty zůstávají jako důkazní
  příloha. Nesmí se rozhodovat ze dvou dokumentů zároveň.

## 4. Důkazní břemeno

Každý nález i každá oprava nese úroveň doložení z verification-failure-modes.md §7
(změřeno / reprodukováno / doloženo experimentem). Oprava se dokládá diferenčním testem
proti ponechanému orákulu. Odhad je v reportu označen jako odhad. Prahy naměřené na tomto
projektu se nepřenášejí na jiný — přenáší se metoda.

## 5. Hranice mandátu

Audit, který si sám rozšiřuje rozsah, přestává být auditem. Čtyři hranice:

| Hranice | Formulace | Proč |
|---|---|---|
| **Produkce** | Vůči produkčnímu prostředí je audit výhradně čtecí. Vše ostatní běží nad stínovým prostředím postaveným z migrací. | Chrání ownera i auditora. Důsledek: projekt bez přehratelných migrací nelze auditovat v plném rozsahu — říct předem, ne v půlce. |
| **Nasazení** | Auditor opravy naprogramuje a otestuje, ale nenasazuje. Nasazení je krok vlastníka. | Odděluje odpovědnost; report smí poctivě říct „opraveno a doloženo, nenasazeno". |
| **Nová schopnost** | Nová funkčnost se v auditu neprogramuje. Vzniknou z ní otázky a specifikace. | Jednořádková oprava často vymění jednu vadu za jinou („nevidím obsah" → „nedostanu se jinam"). Rozsah opravy je rozhodnutí vlastníka. |
| **Refaktoring páteře** | Refaktoring páteřní zápisové cesty se nedělá bez regresní sítě — i když je navržený a odsouhlasený. | Průchod je předpokladem refaktoringu, ne jeho výsledkem. Vědomě neudělaný refaktor se zapíše i s důvodem; to je správný výstup, ne selhání. |

## 6. Prostředí auditu jako nový povrch

Stínové prostředí, které si auditor postaví, je nová plocha útoku na infrastruktuře klienta
i vlastní: testovací databáze s výchozím heslem dosažitelná z internetu, protože kontejnerová
vrstva obešla host firewall; ochranná pravidla, která nepřežila restart. Data bývají
syntetická, otevřený server ne.

Pravidlo: kontrola expozice je **první i poslední krok** každého běhu a opakuje se po každém
startu prostředí. Výsledek se zapisuje do registru jako provozní položka i tehdy, když je čistý.
Celý auditní pracovní prostor (harness, sondy, surová data, snímky) žije **mimo** verzovaný
repozitář klienta; do repozitáře vstupují jen opravy, na oddělené větvi, kterou nasazuje owner.

## 7. Předpoklady na straně projektu — domluvit předem

Část prověrek má tvrdé předpoklady. Rozsah auditu se domlouvá podle toho, co projekt má,
ne podle toho, co by měl mít:

| Projekt má | Co jde spustit |
|---|---|
| přístup ke zdrojovému kódu | statická analýza, hygiena závislostí, injekční povrch |
| migrace přehratelné od nuly | replay, parita ochran, plány dotazů pod skutečnou rolí |
| deterministický generátor dat / anonymizovaný vzorek | invarianty pod souběhem, škálování |
| klient přesměrovatelný na stínové prostředí | vstřikování poruch, průchod plochami — prověrky s nejvyšší výtěžností |
| ADR, specifikace, dokument invariantů | odlišení vady od rozhodnutí; bez toho auditor buď mlčí, nebo hádá |

Rozdíl mezi druhým a čtvrtým řádkem je rozdíl mezi „našli jsme dluh v kódu" a „našli jsme,
že vám mizí data z obrazovky, aniž by to kdokoli poznal". Pokud projekt na čtvrtý řádek
nedosáhne, je poctivé rozsah zúžit v nabídce, ne dodat polovinu.

## 8. Delta po každém běhu

Auditní postup je produkt, ne dokumentace: má verzi, vlastníka a backlog. Po každém běhu se
kontroluje **delta** — co přibylo (nástroj, vzor, metodické pravidlo, zaznamenaná past).
Prázdná delta je signál: buď byl projekt triviální, nebo se postup přestal učit. Nejvýnosnější
prověrka bývá ta, která v předchozí verzi chyběla — pokud postup někam neumí položit otázku,
projekt v té oblasti nevypadá zdravě, jen se v ní neměřilo.
