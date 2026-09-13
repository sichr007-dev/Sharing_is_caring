# Defektní třídy AI-asistovaného vývoje

Tři třídy pocházejí z výzkumu vibe-coded aplikací — Deng, Fan, Meng: *Understanding the
(In)Security of Vibe-Coded Applications*, arXiv:2606.23130 (v3, 09/2026); korpus 10 517
aplikací, hloubkový audit 200 nasazených: **91,0 % obsahovalo aspoň jednu zranitelnost,
65,77 % nálezů Critical/High**. Studie je odvozuje ze tří systémových limitů agentů: *memory
defects* (třída 1), *objective defects* (třída 2), *knowledge defects* (třída 3). Čtvrtá třída,
tiché selhání, vychází z auditní praxe a z literatury o produkčních selháních (viz
[sources.md](sources.md)). Každá třída má lovecké heuristiky — použij je při review i auditu.

## 1. Neúplné šíření změny (incomplete change propagation)

**Mechanismus:** Agent pracuje s lokálním kontextem změny. Pravidlo, které platí na N místech,
rozšíří jen tam, kde ho zrovna vidí. Aplikace *vypadá* zabezpečeně/konzistentně, protože nové
místo pravidlo má — stará místa ne.

**Ilustrativní scénáře (takhle třída vypadá v praxi):**

- **Tabulka bez grantů** — v DB s hardeningem (default privileges vypnuté) startují nové
  tabulky BEZ base grantů; RLS policy bez grantu = feature tiše rozbitá — třeba service
  účet bez SELECT a celá admin sekce vrací 403. Kanonické pravidlo: **každý CREATE TABLE
  explicitně grantuje + privilege assert do smoke testu**.
- **Whitelist drift u stavových přechodů** — nové patchovatelné pole entity musí do whitelistu
  transition RPC, jinak padá každé uložení s přechodem. Kanonické pravidlo: **nové pole ⇒
  aktualizace RPC + rollback smoke**.
- **Guard došířený jen do části kopií** — kontrolní tok existuje ve čtyřech kopiích (per
  entita), oprava dorazí do dvou. Zbylé dvě zůstanou otevřené a nikdo o tom neví, protože
  „to je přece opravené". Čím starší systém a čím víc kopií per entita, tím jistější.

**Lovecké heuristiky:**

- Najdi všechna místa, kde je pravidlo implementováno (grep na název pole/tabulky/policy),
  a ověř, že změna dorazila do všech. Grep hledá **kandidáty**, ne důkaz — každý kandidát se
  potvrzuje čtením (viz verification-failure-modes.md §2).
- **Tabulka kopie × vlastnost**: pro každou kopii funkce/toku sloupce „má guard", „má grant",
  „má validaci", „má ochranu proti souběhu". Rozdíly ve sloupcích jsou nález. Normalizované
  srovnání kopií (sjednotit názvy entit, porovnat příkaz po příkazu) dá číslo místo dojmu.
- Stejný mismatch na ≥2 místech = neexistuje kanonický kontrakt. Nález není „oprav obě místa",
  ale „vytvoř jeden zdroj pravdy" (constraint, sdílená funkce, generovaný kód, smoke assert).
- Ptej se: „co se stane s *příštím* CREATE TABLE / novým polem / novým endpointem?" Pokud
  odpověď zní „musí si vzpomenout", je to latentní defekt této třídy → navrhni mechanickou
  pojistku (smoke assert, lint, template).

## 2. Fix oslabením obranné vrstvy (function-fix side effects)

**Mechanismus:** Viditelná funkčnost dostane přednost před dlouhodobou správností. Chyba
autentizace/CORS/přístupu se „vyřeší" oslabením nebo obejitím kontroly. Aplikace zase funguje —
obranná vrstva fakticky přestala existovat. Extrémní doložený případ z výzkumu: bypass endpoint
s hardcodovanými credentials jako „oprava" loginu.

**Typické podoby:**

- CORS chyba → povolit vše
- RLS blokuje dotaz → přepnout na privilegovaný service účet / policy `using (true)`
- validace odmítá request → validaci odstranit
- test neprojde → mocknout kritický subsystém / test smazat
- chyba se špatně ladí → obalit `try/catch` a mlčky spolknout (tím vzniká třída 4)
- Strict Mode / linter hlásí problém → vypnout detektor (odstranění detektoru, ne oprava)
- limit API ořezává výsledek → zvednout strop místo stránkování (limit je ochrana payloadu)

**Lovecké heuristiky:**

- V diffu hledej **odstraněné nebo změkčené řádky** kolem auth/validace/policy — mazání je
  podezřelejší než přidávání.
- Grep na `using (true)`, service-role klienty, `--no-verify`, `@ts-ignore`, `eslint-disable`,
  prázdné `catch`, `TODO.*auth|security|validat` — každý nález chce zdůvodnění nebo lístek.
- „Dočasné" řešení bez data expirace a vlastníka je trvalé. `TODO: add real auth` se nasadí.
- Owner-gaty a schvalovací kroky v pipeline jsou **záměrné obranné vrstvy** — návrh „pojďme
  to allowlistovat/obejít, ať to jede" je přesně tato defektní třída.

## 3. Nevyslovený invariant (defekty znalostí)

**Mechanismus:** Pravidlo nebylo v promptu a nikdo (uživatel ani agent) ho explicitně
neformuloval. Nejčastější zdroj problémů v citovaném výzkumu — protože se netestuje to,
co nikdo nevyslovil.

**Klasické nevyslovené požadavky:** hesla se hashují; oprávnění se ověřují na serveru; cizí
HTML se sanitizuje; vícenásobný zápis je transakční; tajný klíč nejde do browseru; workflow
pořadí vynucuje server, ne pořadí obrazovek; neúspěšné čtení se uživateli hlásí.

**Typické projektové vzory (kandidáti do `docs/INVARIANTS.md` každého projektu):**

- **Datový klíč ≠ UI label může být záměr** — enum/klíč zůstává stabilní, zobrazovaný název
  se mění; „oprava" tohoto rozporu je regrese. Záměr patří do ADR.
- **Stavové přechody jdou výhradně přes atomické serverové operace**, nikdy přímým UPDATE.
- **Technická časová pole ≠ doménová osa** — `created_at` může nést datum importu/reseedu;
  KPI a řady se bucketují po doménových datech.
- **Reseed/refresh sdíleného prostředí ⇒ QA baseline neplatná ⇒ povinný smoke re-run.**
- **Evidence migrací může lhát** — aplikovanost změny se ověřuje dotazem do cílového systému
  (`pg_proc`, `pg_policies`), ne z evidenční tabulky.
- **Chybějící rozhodovací kritérium** — systém řeší strukturálně tutéž situaci dvěma vzory
  (jednou samostatná entita, podruhé rozlišovací atribut) a nikde není zapsáno, kdy platí co.
  Není to špatný model, je to nekonzistentní model; nejlevnější náprava je krátký zápis
  kritéria, ne refaktor.

**Lovecké heuristiky:**

- Když si při analýze říkáš „to je přece jasné" — právě jsi našel kandidáta na nevyslovený
  invariant. Zapiš ho.
- U každé mutace: kdo garantuje, že ji smí provést *tento uživatel* v *tomto stavu* nad
  *tímto záznamem*? Pokud odpověď je jen „UI to nenabídne", invariant není vynucen.
- Po dokončení změny: která pravidla jsem dodržel jen proto, že je znám z kontextu session?
  Ta do dokumentu invariantů, jinak je příští session (nebo jiný agent) poruší.

## 4. Tiché selhání (silent failure)

**Mechanismus:** Agent generuje šťastnou cestu a k ní „defenzivní" fallback — `?? []`, prázdný
`catch`, výchozí hodnotu. Kód vypadá robustně, ale selhání se převede na prázdný nebo částečný
výsledek bez signálu. Aplikace vznikající „obrazovku po obrazovce" má chybové stavy jen tam,
kde si je někdo výslovně vyžádal (typicky ukládání); čtení ploch žádný chybový stav nemá.
Ticho je horší než pád: pád se eskaluje jako incident, ticho se projeví až jako špatné
rozhodnutí člověka, který netušil, že mu chybí data — a příčinu už nikdo nedohledá.

**Proč to není okrajové:** Yuan et al. (OSDI 2014) analyzovali 198 produkčních selhání
distribuovaných systémů — **92 % katastrofických selhání bylo důsledkem špatného ošetření
nefatální, explicitně signalizované chyby**, třetina z nich triviálně (prázdný handler, TODO,
přehnaně široký catch). OWASP Top 10 2025 tuto třídu poprvé zařadil jako samostatnou kategorii
A10 *Mishandling of Exceptional Conditions* (24 CWE, mj. CWE-636 *Failing Open*). Auditní praxe
nad vibe-coded aplikacemi ukazuje totéž: vlny auditu, které se ptají „je to napsané správně",
vycházejí dobře, a teprve otázka „co se stane, když se něco pokazí" vynese nálezy s nejvyšší
závažností — typicky většina poruchových scénářů skončí bez jakéhokoli signálu uživateli.

**Typické podoby (Supabase-style stack, přelož si na svůj):**

- **Prázdný výsledek místo chyby** — `const { data } = await q; setRows(data ?? [])`;
  `error` se nikdy nečte. Uživatel vidí „nemám práci", ne „nepodařilo se načíst".
- **Chybové hlášení jen pro akce** — ošetření chyb existuje výhradně u ukládání; seznamy
  a detaily žádný chybový stav nemají.
- **Tichý ořez na stropu API** — PostgREST `db-max-rows` (na Supabase typicky 1000) je tvrdý
  strop pro tabulky, views **i RPC**; odpověď je 200 bez chyby, jen kratší. Součty, exporty
  a KPI se počítají z části dat. Projeví se až při růstu historie.
- **Tvrdý limit infrastruktury bez odpovědi** — filtr `.in('id', ids)` přes GET narůstá
  s výběrem uživatele; při určité délce URL ho brána odmítne (414) dřív, než dorazí
  k aplikaci, a to bez CORS hlaviček. V prohlížeči je to síťová chyba bez odpovědi — nevidí
  ji uživatel, běžný monitoring ani sonda sledující jen kanál odpovědí.
- **Ztracený zápis mimo chráněnou cestu** — dva lidé nad týmž záznamem: poslední zápis vyhraje
  a nikde nevznikne stopa, že ke konfliktu došlo. Vedle chráněné cesty (transition RPC)
  existuje paralelní vchod přímým `update().eq('id', …)` bez podmínky na verzi.
- **Pravidlo existuje a neplatí** — pravidlo je napsané, ale přebije ho jiné se stejnou vahou,
  jen zapsané později (permisivní policies se skládají přes OR; kaskádové styly; konfigurační
  vrstvy). Přítomnost pravidla nic nedokazuje.

**Lovecké heuristiky:**

- Grep kandidátů: `?? []`, `|| []`, `catch {}` / `catch (e) { console.`, `.select(` bez
  `.range(`/`.limit(`/`.single(` a bez `count`, `.in(` s neomezeným polem, `update(` +
  `.eq('id'` bez podmínky na `version`/`updated_at`. Kandidát ≠ nález: potvrď čtením nebo testem.
- **Poměr chybových stavů k čtecím dotazům.** Spočítej chybové stavy v UI a čtecí dotazy.
  Řádový nepoměr je nález sám o sobě.
- **Matice porucha × plocha.** Pro každou plochu tři režimy poruchy (chyba serveru, přerušené
  spojení, mrtvý socket/timeout) a jediná otázka: uvidí uživatel chybu, nebo prázdnou plochu?
  Poměr „nouzové prázdno" k ošetřeným chybám je metrika, která se přenáší mezi projekty.
- **Desetinásobek dat.** Porovnej počet řádků v DB s počtem doručeným klientovi. Na malých
  datech tato třída neexistuje; audit bez škálovaného datasetu ji z principu nenajde.
- **Bisekce limitu.** Půlením najdi hranu (délka URL, velikost těla), kde infrastruktura
  požadavek odmítne, a přepočti ji na provozní veličinu („kolik položek ve filtru to vydrží").
  Přenáší se metoda, ne číslo — hrana je vlastnost konkrétní infrastruktury.
- **Test ztraceného zápisu.** Dvě sezení nad týmž záznamem: A otevře, B uloží, A uloží starou
  kopii. Nedestruktivní (hodnoty se vrátí). Před závěrem ověř, že zápis A vůbec doletěl
  (viz verification-failure-modes.md §3).
- **Invariantní test výsledku, ne přítomnosti.** U pravidla, které „přece platí", změř
  výsledek; při podezření vlož identické pravidlo na konec pořadí a změř rozdíl.
- **Ráčna:** čistý výsledek jednorázové kontroly převeď na trvalé pravidlo — lint „select bez
  range/limit/single", CI gate „žádný request nesmí selhat bez viditelného chybového stavu",
  smoke „count v DB = count doručený".

## Proč čtyři třídy, ne checklist zranitelností

OWASP kategorie (Broken Access Control, Injection, Mishandling of Exceptional Conditions…)
říkají **co** je rozbité. Tyto čtyři třídy říkají **jak k tomu při AI-asistované práci
dochází** — a proto fungují preventivně: dají se aplikovat na změnu, která ještě žádnou
zranitelnost nemá. Checklist zranitelností patří do auditu (stop-ship reference); optiky
patří do každodenního review.
