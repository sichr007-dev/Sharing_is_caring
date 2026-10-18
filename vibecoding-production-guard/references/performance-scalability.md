# Výkon a škálovatelnost: provozní úloha, zatížení, mez

Výkon má samostatný výsledek vedle funkční správnosti, bezpečnosti a obnovitelnosti.
Audit smí objevit neúnosné čekání nebo růst práce i tam, kde specifikace mlčí.
Tento postup určuje, co změřit a jak omezit závěr; neurčuje univerzální SLO.
Opory a meze přenosu: [registr zdrojů](performance-sources.md).

## 1. Kdy a jak hluboko

- **Audit projektu / vydání:** vždy vyplň hodnocení I1–I6 níže. Vyber kritické,
  frekventované a objemově rizikové úlohy; napiš, které plochy výběr nepokrývá.
- **Změna:** vyhodnoť dopad na počet dotazů, data, souběh, render, cache, fronty,
  infrastrukturu a závislosti. Při dopadu použij dotčené kontroly a baseline/opravu.
- **Čistě redakční změna bez dopadu:** stručná kontrola stačí. Výslovně objednaný
  širší audit přesto vyhodnoť samostatně v jeho rozsahu.

Rozliš malé diagnostické měření, ověření cílového provozu a hledání mezí. Nevyžaduj
šest druhů zátěžových testů u každého diffu. Nedostupný běh znamená UNKNOWN a konkrétní
navazující krok, ne prázdnou sekci. Zátěž, poruchy a zvětšování dat patří do izolace;
pravidla přístupu do produkce a předání citlivých dat určuje projekt.

## 2. Výkonový kontrakt před měřením — I1

Z existujícího provozu, speců a evidence sestav profil. Doplň pouze chybějící rozhodnutí.

| Povinné pole | Co musí být určeno |
|---|---|
| Úloha a aktér | Od konkrétní akce do použitelného výsledku: načíst práci, vyhledat, uložit, exportovat. U asynchronního toku i čas do skutečného dokončení, ne pouze přijetí HTTP. |
| Výsledek a kritérium | Správnost/úplnost; přijatelná odezva a četnost překročení, chybovost, průchodnost či deadline dávky. Zdroj kritéria, vlastník a status schválené / návrh / chybí. |
| Data a růst | Aktivní i historický objem, velikost objektů, rozložení, horké klíče/velký zákazník, selektivita filtrů, růst do konkrétního horizontu. Odhady odliš od měření. |
| Práce v čase | Počet současně pracujících, příchody úloh, think time, mix čtení/zápisů/exportů, periodické úlohy, retry, sezónní špička a background práce. |
| Prostředí | Commit/build, konfigurace a resource limity, DB/API/browser/tool verze, síť a zařízení, role a auth, cache, indexy/statistiky, vazba na cílové prostředí. |
| Plán běhu | Model zátěže, měřené intervaly, zahřátí, opakování, stop podmínky a úklid; jak poznáme vyčerpání generátoru. |

Chybějící kritéria jsou nález v požadavcích. Navrhni je s důvodem a vlastníkem;
nepřebírej 200 ms, 10× data ani počet VU z cizího příkladu jako smlouvu. Průzkum může
pokračovat s označenými předpoklady. Bez schválené meze nedokládá splnění SLO;
prokazatelné blokování práce či neúplná data lze reportovat i bez číselného SLO.

## 3. Celý uživatelský tok a místo spotřeby — I2

Měř čas do použitelného výsledku a správnost výsledku pod skutečnou testovací rolí.
Použij přiměřený klientský profil a stejné datové podmínky jako pro backendové měření.
U SPA ověř přihlášenou navigaci, filtr, otevření detailu a uložení podle vybraných úloh;
statická úvodní stránka či login je nenahrazuje. Ukončení měření vázej na doménový stav,
ne pouze `load`, HTTP 200 nebo obecné utichnutí sítě s pollingem.

Zachyť počet a velikost requestů na úlohu, jejich závislosti a opakování, přenos,
JS/render a dlouhé blokování interakce. Propoj waterfall/profil s backendem a DB.
Rychlé jednotlivé SQL nevyvrací N+1, sekvenční síťové vlny, nadměrný JSON nebo drahý
render. Rozlišení problému není závazek přidat index, cache či hardware.

Na serveru zkoumej relevantní CPU, paměť/GC, I/O, pool spojení, čekání na zámky,
frontu a závislosti: využití, saturaci/čekání a chyby. U DB porovnej plán a skutečné
řádky/loops/buffers s reprezentativními parametry, pokud je měření dostupné.
`EXPLAIN ANALYZE` dotaz skutečně provádí; není univerzálně bezpečná produkční inspekce.
Zápisy a náročné plány ověř izolovaně, rollback nevrací všechny možné externí efekty.

## 4. Generátor musí odpovídat otázce — I3

**Uzavřený model:** konečný počet uživatelů čeká na výsledek a pak pokračuje.
Je vhodný pro takovou práci; při zpomalení však snižuje příchod nových úloh.
**Otevřený model:** příchody nezávisí na dokončení. Použij jej pro tvrzení o zvládnutí
takového přítoku. **Smíšený model:** například příchody relací otevřené, kroky uvnitř
relace sekvenční. VU, relace, úlohy/s a requesty/s nejsou zaměnitelné jednotky. [PS02, PS07]

Zapiš plánované, skutečně odstartované a dokončené úlohy, chyby, timeouty,
přerušené/neodstartované iterace a jejich příčiny. Nevynechávej čekající práci z
kapacitního závěru. Latence od skutečného odeslání sama nezahrnuje zpoždění před startem;
uveď hranice měření. U řízených příchodů sleduj skluz proti plánu nebo jeho dostupnou
evidenci. Tím prověřuješ i coordinated omission; samotný přepínač open-loop jej
nevylučuje ve všech dalších měřených vrstvách.

Současně měř generátor: CPU/paměť, síť, spojení, dostupné VU a dosaženou frekvenci.
`dropped_iterations` může vzniknout konfigurací i zpomalením cíle; nejprve rozliš
příčinu, automaticky nezvyšuj VU ani neobviňuj server. Pokud generátor či sdílený host
omezuje test, kapacita aplikace zůstává v této oblasti UNKNOWN. [PS08, PS14]

## 5. Růst, špička a návrat — I4

Sestav malou zdůvodněnou matici: současná data/práce, očekávaný růst a relevantní
hranice. Měň odděleně objem dat a přítok/souběh; přidej jejich důležitou kombinaci.
Zachovej realistické vazby, skew/hotspot a velikosti záznamů; uniformní kopie nejsou
náhradou zákazníka s dominantním objemem. Warm a cold cache měř odděleně podle použití.

Postupně porovnej dokončenou práci, odezvu, chyby, requesty/úlohu a spotřebu zdrojů.
Křivka má ukázat zlom, rezervu nebo mez, kam sahá důkaz. Na novou infrastrukturu,
jiné rozložení dat či řádově jiný objem z jediného bodu neextrapoluj. Nalezený limit
vždy patří konkrétnímu workloadu a prostředí, nikoli „počtu uživatelů aplikace“ obecně.

Podle rizika vyber cílovou zátěž, špičku, delší běh přes relevantní cyklus a hledání
meze s limitem nákladů. Délku odvoď od front, cache, batchí, obnovy tokenů či
podezření na akumulaci; krátký smoke neprokazuje stabilitu celé směny. Uveď vynechané
režimy a proč. Po odeznění špičky/poruchy sleduj odtok fronty, nové legitimní operace,
paměť/spojení a dobu zotavení. Růst paměti sám nedokazuje leak. [PS03, PS05, PS14]

## 6. Správnost při tlaku a platnost měření — I5

- Otestuj úplnost nad skutečným limitem řádků, filtru/URL a velikosti objektu.
  Kontroluj IDs/počty/agregáty proti nezávislému očekávání. Rychlý useknutý výsledek
  nesmí vyhrát výkonový test; drahý exact count nevynucuj u každé interaktivní stránky.
- Prověř relevantní souběžné zápisy, retry po neznámém výsledku a zpožděnou závislost.
  Měř úspěšnou business práci, nejen transport; stávající integritní gate zůstává.
- Browser měřicí kód zachytí síťová selhání **i** HTTP chyby **i** chybné business
  odpovědi. `requestfailed` a HTTP 4xx/5xx jsou odlišné kanály. Je-li přihlášení jen
  přípravou scénáře, odděl je od jeho měřeného okna. Je-li hodnocenou úlohou nebo
  součástí mixu (např. ranní přihlašovací špička), měř je samostatně nebo zahrň do toku.
  Výsledky ukládej tak, aby přežily restart workeru. [PS11]
- Uveď definici metriky, interval, počet vzorků, zahřátí a opakované běhy/rozptyl.
  P50 a vhodný tail percentil uváděj po toku, vedle chybovosti a throughputu.
  P99 z mála pozorování není důkaz spolehlivého tailu; nejistotu přiznej.
- Neprůměruj percentily skupin. Úspěšné a chybové latence můžeš rozlišit, ale zveřejni
  obě populace a počty neukončených pokusů; timeout není rychlý úspěch ani zmizelý vzorek.
- Baseline a opravu srovnej ve srovnatelném prostředí, bez souběžného cizího testu;
  změny cache, dat, sítě a stroje přiznej. Posuď nejistotu rozdílu a reprodukovatelnost:
  rozptyl jednotlivých měření není nejistota odhadu rozdílu. Malý efekt může být
  průkazný při vhodném experimentu a dostatku nezávislých pozorování; přiznej závislost
  vzorků a nevydávej šum za zlepšení. Uchovej očištěná surová měření. [PS01–PS06, PS23]

## 7. Povinný výstup a rozhodnutí — I6

Každý hodnocený tok má řádek; chybějící měření zůstane viditelné:

| Tok / kritérium a jeho status | Data, práce, síť | Důkaz/verze/běhy | Odezva, dokončení, chyby, zdroje | Mez důkazu | PASS / FAIL / UNKNOWN / N/A; další krok |
|---|---|---|---|---|---|

Přilož matici I1–I6 a záznam vybraných/neprovedených režimů, příčinu doloženého
problému nebo hypotézu odděleně, návrh nápravy a stejný test pro ověření nápravy.
Výkonový PASS platí jen pro splněná kritéria v popsaném provozním rozsahu.
Chybějící kritérium nebo nedostatečný experiment znamená UNKNOWN pro tvrzení o přijetí;
neznamená automaticky vadu kódu. N/A má konkrétní důvod nepoužitelnosti.

Nesplnění kritického výkonového požadavku či neověřená kritická kapacita blokují
příslušnou bránu stejně jako jiné kritické kontroly. Nekritický dluh má vlastníka
a rozhodnutí podle místní metodiky. Přijetí rizika nepřepíše výsledek na PASS.
Celkový verdikt nesmí zamaskovat výkonové UNKNOWN za zelené funkční testy.
Statický/read-only audit může skončit s omezeným verdiktem a proveditelným plánem měření.

## 8. Volba otevřeného nástroje

Nejprve použij dostupné schopnosti. Před konkrétním skriptem ověř dokumentaci
instalované verze; zdejší odkazy nejsou pinem závislostí. Žádný nástroj sám není audit.

| Otázka | Kandidát a důvod | Co tím není prokázáno |
|---|---|---|
| Kde čeká přihlášený uživatel SPA? | Existující Playwright + browser profil/Network; alternativně [sitespeed.io](https://github.com/sitespeedio/sitespeed.io) a Browsertime pro opakované browser scénáře. | HTTP generátor neprovádí render; jeden rychlý browser nedokládá kapacitu backendu. |
| Unese služba konkrétní přítok/mix? | [k6](https://github.com/grafana/k6): skriptované scénáře, příchody, metriky a thresholds; [Locust](https://github.com/locustio/locust) pokud tým/testy stojí na Pythonu. | Zvolit model a ověřit sémantiku; VU s čekáním není automaticky cílové RPS. |
| Regrese stránky a browser metrik? | [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci), Lighthouse user flows podle podpory režimu. | Celkové skóre/login nenahrazuje přihlášený task, backendovou kapacitu ani terénní data. |
| Izolovaný HTTP endpoint a latence při řízeném přítoku? | [wrk2](https://github.com/giltene/wrk2): úzce vymezená diagnostika, explicitní měření plánovaného času. | Nenahrazuje autentizovaný business scénář, browser ani audit datové správnosti. |
| Proč se dotaz/zdroj zpomaluje? | DB plán/statistiky/čekání, systémový profiler a existující telemetry. | Samotný index, počet FK bez indexu či vysoké CPU neurčuje vadu; dolož konkrétní workload. |

U k6 samotný `check` není release gate; pro automatické zamítnutí použij odpovídající
threshold a ověř selhávající kontrolní běh i návratový kód. Kritéria odvoď od kontraktu,
ne od ukázkových defaultů. [PS09] HAR/traces mohou obsahovat tokeny, cookies a payloady;
uchovávej bezpečně a před commitem/sdílením očisti. Zátěž třetích stran musí být
vyňata nebo autorizovaná. Automatický cloud upload ani nový monitoring stack nejsou
součástí volby nástroje. Cizí skilly jsou podklad ke srovnání, ne nové oprávnění.
