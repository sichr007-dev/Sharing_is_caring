# Tři defektní třídy AI-asistovaného vývoje

Mechanismy vzniku chyb identifikované výzkumem vibe-coded aplikací — Deng, Fan, Meng:
*Understanding the (In)Security of Vibe-Coded Applications*, arXiv:2606.23130 (06/2026);
korpus 10 517 aplikací (VibeApps), hloubkový audit 200 nasazených: **90 % obsahovalo aspoň
jednu zranitelnost, 76,7 % nálezů Critical/High**. Každá třída má lovecké heuristiky —
použij je při review i auditu.

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

**Lovecké heuristiky:**

- Najdi všechna místa, kde je pravidlo implementováno (grep na název pole/tabulky/policy),
  a ověř, že změna dorazila do všech.
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
- chyba se špatně ladí → obalit `try/catch` a mlčky spolknout
- Strict Mode / linter hlásí problém → vypnout detektor (odstranění detektoru, ne oprava)

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
pořadí vynucuje server, ne pořadí obrazovek.

**Typické projektové vzory (kandidáti do `docs/INVARIANTS.md` každého projektu):**

- **Datový klíč ≠ UI label může být záměr** — enum/klíč zůstává stabilní, zobrazovaný název
  se mění; „oprava" tohoto rozporu je regrese. Záměr patří do ADR.
- **Stavové přechody jdou výhradně přes atomické serverové operace**, nikdy přímým UPDATE.
- **Technická časová pole ≠ doménová osa** — `created_at` může nést datum importu/reseedu;
  KPI a řady se bucketují po doménových datech.
- **Reseed/refresh sdíleného prostředí ⇒ QA baseline neplatná ⇒ povinný smoke re-run.**
- **Evidence migrací může lhát** — aplikovanost změny se ověřuje dotazem do cílového systému
  (`pg_proc`, `pg_policies`), ne z evidenční tabulky.

**Lovecké heuristiky:**

- Když si při analýze říkáš „to je přece jasné" — právě jsi našel kandidáta na nevyslovený
  invariant. Zapiš ho.
- U každé mutace: kdo garantuje, že ji smí provést *tento uživatel* v *tomto stavu* nad
  *tímto záznamem*? Pokud odpověď je jen „UI to nenabídne", invariant není vynucen.
- Po dokončení změny: která pravidla jsem dodržel jen proto, že je znám z kontextu session?
  Ta do dokumentu invariantů, jinak je příští session (nebo jiný agent) poruší.

## Proč tři třídy, ne checklist zranitelností

OWASP kategorie (Broken Access Control, Injection…) říkají **co** je rozbité. Tyto tři třídy
říkají **jak k tomu při AI-asistované práci dochází** — a proto fungují preventivně: dají se
aplikovat na změnu, která ještě žádnou zranitelnost nemá. Checklist zranitelností patří do
auditu (stop-ship reference); optiky patří do každodenního review.
