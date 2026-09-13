# Analytický gate a self-review — plné znění

Dva šablonové průchody: **před změnou** (gate) a **po implementaci** (self-review diffu).
Smysl: nahradit impulzivní „prompt patching" (symptom → úprava → obrazovka funguje → dál)
analýzou, která drží systémový kontext. Nejde o délku — u malé změny je průchod otázka
dvou minut; u změny na gate povrchu (auth/policies/migrace/stavy/chybové cesty) je to jediná
pojistka proti čtyřem defektním třídám.

## Gate před změnou (zatím neměň kód)

1. **Skutečná příčina** — popiš příčinu, ne symptom. Ověř ji důkazem (log, dotaz, repro),
   ne dojmem. Zvlášť u „RLS/validace/auth blokuje X": nejdřív rozhodni, jestli blokuje
   *správně*. Zvlášť u „vrací to jen N řádků" / „filtr někdy nic nevrátí": nejdřív zjisti,
   jestli nejde o strop API nebo limit brány (defect-classes.md §4).
2. **Celý tok** — zmapuj cestu: klientská komponenta → hook/query vrstva → API dotaz nebo
   RPC → brána/proxy → policy/grant → úložiště. Vyjmenuj všechny dotčené soubory, endpointy,
   tabulky, policies, RPC, secrets, externí služby.
3. **Spec-first** — přečti relevantní projektový spec, příslušné ADR a dokument invariantů
   (existují-li). Nerekonstruuj pravidla z paměti.
4. **Invarianty** — vyjmenuj, které systémové a bezpečnostní invarianty se změny týkají
   a jak zůstanou zachovány. Nenapsaný invariant, na který ses právě spolehl → zapiš jako
   kandidáta do dokumentu invariantů.
5. **Kontrola oslabení** — zkontroluj explicitně: neoslabuje návrh autentizaci, autorizaci,
   RLS/policies, granty, validaci, CORS, TLS, transakčnost, auditování nebo limity payloadu?
   Pokud ano, je to rozhodnutí pro ownera, ne implementační detail.
6. **Blast radius a propagace** — která další místa implementují stejné pravidlo
   (whitelisty, grants, UI guard + serverový guard, duplikované konstanty, kopie funkce per
   entita)? Změna jde do všech, nebo má zapsaný důvod, proč ne. Zvaž mechanickou pojistku
   (smoke assert, constraint), aby si příští změna „nemusela vzpomenout".
7. **Souběh a regrese** — co se stane při dvojkliku, retry, dvou uživatelích nad stejným
   záznamem (ztracený zápis: má UPDATE podmínku na verzi?), starém klientu proti nové DB
   (oddělené nasazení DB a klienta!)?
8. **Chování při selhání** — pro každou novou nebo změněnou čtecí i zapisovací cestu: co
   uvidí uživatel, když dotaz selže, vrátí jen část (strop řádků), nebo ho brána odmítne
   (délka URL, velikost těla)? Prázdný výsledek musí být odlišitelný od chyby; částečný
   výsledek musí být detekovatelný (count vs. doručeno, explicitní stránkování).
9. **Testy předem** — navrhni test/smoke, který před opravou selže a po ní projde.
   U DB změny: migrace + rollback-tx smoke blok do repa. U chybové cesty: negativní test,
   který poruchu skutečně vyvolá (ne jen ověří, že handler existuje).
10. **Minimální změna** — teprve teď navrhni diff. Žádné nesouvisející refaktory; ty jdou
    do samostatného commitu, pokud vůbec. Pokud jednořádková oprava vymění jednu vadu za
    jinou (např. „nevidím obsah" za „nedostanu se jinam"), rozsah opravy je rozhodnutí
    ownera — předlož obě varianty s cenou, nenasazuj tu levnější mlčky.

Krátká cesta: u změny, která se gate povrchů dotýká jen okrajově, stačí body 1, 2, 6, 8 —
ale rozhodnutí „jdu krátkou cestou" napiš nahlas do odpovědi, ať je auditovatelné.

## Self-review diffu (po implementaci, před commitem)

Projdi vlastní diff jako nezávislý reviewer. Nevylepšuj kód — nejdřív jen sepiš nálezy
(soubor, závažnost, důkaz):

- porušení invariantů z bodu 4 gate
- chybějící serverová autorizace (nové/změněné dotazy a RPC — kdo smí, v jakém stavu,
  nad čím)
- neošetřené vstupy: skládání SQL/HTML/URL, chybějící runtime validace, `...body` spready
- únik secrets: nové env proměnné (build-time prefixy typu `VITE_*`/`NEXT_PUBLIC_*`!),
  klíče v kódu, tokeny v logu, source mapy a interní soubory v publikovaném artefaktu
- chybějící transakce u vícekrokových zápisů; přímé UPDATE stavů mimo transition operace
- race conditions a duplicitní mutace (double-click, retry); editace záznamu bez ochrany
  proti ztracenému zápisu
- **neúplné šíření změny**: grep na název pole/pravidla — dorazila změna do whitelistů,
  grantů, smoke assertů, typů, všech kopií funkce?
- **tiché selhání**: nové čtecí cesty bez chybového stavu odlišitelného od prázdna; dotazy
  bez explicitního `range`/`limit`/`single` nad tabulkou, která poroste; filtry ze seznamu
  ID přes GET; `?? []` a prázdné `catch` u nového kódu
- odstraněné/změkčené obranné řádky (mazání kolem auth/validace je podezřelejší než
  přidávání); nové `@ts-ignore`, `eslint-disable`, prázdné `catch`, vypnuté kontroly,
  zvednuté limity
- regresní dopad na jiné komponenty/endpointy sdílející tok
- chybějící error handling a auditní stopa

Nálezy vyřeš, nebo je explicitně předlož ownerovi jako známé riziko. Mlčky přejít nález
= defekt třídy 2 (oslabení obrany), jen v měkčí podobě.

## Vztah k projektové disciplíně

Gate nenahrazuje, jen zapíná ve správný moment to, co v projektu už platí: spec-first,
schvalovací kroky na DDL/nasazení, migrace + smoke bloky v repu, decision log či commit
trailery (má-li je projekt), ADR pro load-bearing rozhodnutí. Pokud gate odhalí, že se rozhoduje
něco strukturálního (oslabení vrstvy, nový kanonický kontrakt), výstupem je ADR návrh,
ne tichý fix.
