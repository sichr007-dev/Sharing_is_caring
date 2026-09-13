# Stop-ship checklist — binární, s důkazní povinností

Každá položka se zodpovídá **ano/ne + důkaz** (soubor:řádek, výstup dotazu, log, screenshot).
„Myslím, že ano" = ne. Položky označené 🔴 blokují nasazení vždy; 🟡 blokují, pokud pro ně
neexistuje zapsané vědomé rozhodnutí (ADR nebo dokument invariantů).

Formulováno pro typický stack „klientská SPA + managed Postgres/BaaS" (React + Supabase apod.),
principy platí obecně — přelož si názvy na svůj stack.

## A. Secrets, klíče a artefakt

- 🔴 Žádný service/secret klíč v klientském kódu, repu ani logu.
  Důkaz: grep bundlu/`src/` na klíčové prefixy; kontrola build-time env proměnných
  (vše s prefixem typu `VITE_*`/`NEXT_PUBLIC_*` končí v bundlu — smí tam být jen
  anon/publishable klíče a veřejné URL).
- 🔴 Žádné hardcodované credentials, tokeny, hesla v kódu ani migracích.
  Pozn.: credential lifecycle (generování + založení účtu + zápis) nepatří do jednoho
  shell skriptu — AV heuristiky to flagují jako malware a na Windows soubor rovnou smažou.
- 🟡 Publikovaný artefakt (bundle, balíček, image) neobsahuje source mapy, `.env`, interní
  dokumentaci ani testovací fixtury. Důkaz: výpis obsahu artefaktu, ne packaging konfigurace
  — únik zdrojáků přes špatně nakonfigurované balení je doložená třída incidentu, kterou
  logický review nevidí (viz sources.md §F).

## B. Přístup (autentizace ≠ autorizace)

- 🔴 Každá tabulka s daty má RLS zapnutou a policy, která skutečně filtruje.
  Důkaz: dotaz na `pg_policies` + `pg_tables.rowsecurity`; každá `using (true)` policy má
  zapsané rozhodnutí, proč je OK. Pozor: permisivní policies se skládají přes OR — jedna
  široká policy tiše přebije všechny úzké (defect-classes.md §4 „pravidlo existuje a neplatí").
- 🔴 Autorizace se vynucuje v DB/na serveru, ne skrytím prvků v klientu (viditelnost
  v navigaci je UX, ne ochrana).
- 🔴 Uživatel nemůže změnou ID v requestu číst/měnit cizí záznam (IDOR/BOLA test).
- 🟡 Grants odpovídají hardening režimu DB: nová tabulka má explicitní GRANT +
  privilege assert ve smoke (klasika třídy „neúplné šíření změny", viz defect-classes.md §1).
- 🟡 Sensitive RPC nejsou volatelné s anon klíčem; SECURITY DEFINER funkce mají prověřený
  `search_path` a oprávnění — a guardy uvnitř nich pamatují, že `current_user` je vlastník,
  ne volající (viz verification-failure-modes.md §3).

## C. Integrita dat a souběh

- 🔴 Business invarianty vynucuje DB (NOT NULL, UNIQUE, FK, CHECK), ne jen formulář.
- 🔴 Vícekrokové business operace jsou transakční; stavové přechody jdou přes atomické
  serverové operace, žádný přímý UPDATE stavu z klienta.
- 🟡 Kritické mutace přežijí double-click/retry (idempotence: unikátní constraint,
  idempotency key, nebo doložené proč to nevadí).
- 🟡 Workflow pořadí vynucuje server — klient nemůže zavolat „finalize" bez předchozích stavů.
- 🟡 Editace záznamu z UI chrání proti ztracenému zápisu: UPDATE má podmínku na verzi nebo
  `updated_at` (compare-and-set) a nulový počet změněných řádků se hlásí jako konflikt —
  nebo je zapsané rozhodnutí, že last-write-wins je pro tuto entitu přijatelné. Důkaz:
  test dvou sezení (defect-classes.md §4), ne přítomnost sloupce `version`.

## D. Vstupy, výstupy a chování při selhání

- 🔴 Žádné skládání SQL/HTML/URL/path z uživatelského vstupu bez parametrizace/sanitizace;
  `dangerouslySetInnerHTML` (a ekvivalenty) jen se sanitizovaným obsahem.
- 🔴 Runtime validace vstupů na hranici (statické typy za běhu neexistují; žádné slepé
  `update({ ...body })`).
- 🔴 Každá čtecí cesta má chybový stav odlišitelný od prázdného výsledku. Důkaz: vyvolaná
  porucha (chyba serveru, přerušené spojení, timeout) nad plochou a screenshot toho, co
  uživatel vidí — ne přítomnost `error` větve v kódu.
- 🟡 Dotazy nad tabulkami, které porostou, mají explicitní stránkování (`range`/`limit`)
  nebo kontrolu „count v DB = počet doručený"; strop API (`db-max-rows`) je znám a zapsán.
  Souhrny, exporty a KPI se nikdy nepočítají z odpovědi, která mohla být ořezaná.
- 🟡 Filtry skládané ze seznamu ID nebo z výběru uživatele mají hranici velikosti, jdou
  přes POST/RPC, nebo je doložená maximální délka požadavku pod limitem brány. Odmítnutí
  bránou (413/414) nemá CORS hlavičky a v aplikaci vypadá jako „nic".
- 🟡 Produkční chybové stavy neprozrazují interní detaily (stack trace, SQL chyby) uživateli.
- 🟡 Vstupy přijímající URL nebo soubory (import, webhook, upload, náhled externí stránky)
  mají omezení: typ/velikost souboru, allowlist cílů, žádný přístup do interní sítě (SSRF).
- 🟡 Neautentizované vstupní body (feed tokeny, webhooky) mají rate limiting nebo aspoň
  doložené rozhodnutí, proč ho nepotřebují; tokeny jsou revokovatelné.
- 🟡 Bezpečnostní HTTP hlavičky (CSP, X-Frame-Options…) nastavené, nebo zapsané rozhodnutí,
  že u interní aplikace za loginem zatím ne.
- 🟡 Pokud aplikace volá LLM s uživatelským vstupem nebo mu dává nástroje: vstup i načtený
  obsah (dokumenty, e-maily, webové stránky) se považují za nedůvěryhodné instrukce, nástroje
  mají allowlist a oprávnění *volajícího uživatele* (ne service účtu), výstup modelu se
  nespouští ani nevkládá do HTML bez sanitizace (OWASP Agentic ASI01–ASI03, viz sources.md §G).

## E. Migrace a nasazení (oddělené nasazení DB a klienta)

- 🔴 DB změna aplikována PŘED nasazením klienta; nová serverová volání (RPC/endpointy)
  ověřena v cílovém prostředí přímým dotazem (`pg_proc`, health check) — ne přes evidenci
  migrací, ta může lhát.
- 🔴 `git log origin/main..HEAD` zkontrolován před pushem — push bere celou větev včetně
  cizích lokálních commitů; commit čekající na „push až po DB apply" snadno odjede
  s cizím pushem.
- 🔴 Schéma se mění jen verzovanou migrací v repu + smoke blok; DDL/hromadné DML nad
  sdíleným prostředím = schvalovaný krok, nikdy tichý zásah.
- 🟡 Rollback známý: jak se změna vrací (migrace zpět / redeploy předchozí verze klienta)
  a kdo to umí provést.
- 🟡 Po reseedu/refreshi sdíleného prostředí proběhl smoke re-run (QA baseline je neplatná).
- 🟡 Mechanické pojistky nasazení (piny počtu migrací, hash manifesty, count asserty)
  bumpnuty vědomě a s doložením provenience — a po zavedení aspoň jednou skutečně spuštěny.
- 🟡 Mezistav „nový klient proti staré DB" (nebo naopak) je buď nemožný, nebo otestovaný;
  dvoukolejné nasazení už vyrábělo částečně nefunkční mezistavy.

## F. Provoz, obnova a auditovatelnost

- 🟡 Vlastnosti zálohy známé (co, jak často, retence) a obnova alespoň jednou vyzkoušená;
  bez toho je „platforma dělá backup" jen domněnka. Chybí-li od vlastníka cíle obnovy
  (přípustný výpadek, přípustná ztráta dat), vede se položka jako **blokovaná rozhodnutím**,
  ne jako nesplněná.
- 🟡 Kritické operace zanechávají auditní stopu (kdo, kdy, co); aplikační chyby jsou někde
  vidět, ne jen v konzoli prohlížeče — včetně požadavků, které selžou **bez odpovědi**
  (monitoring sledující jen kanál odpovědí je k nim slepý).
- 🟡 Závislosti bez známých kritických zranitelností (lockfile + občasný audit; „funguje"
  ≠ „může zůstat navždy"). Instalace v CI jde z lockfile (`npm ci`), nové verze mají
  minimální stáří / cooldown, postinstall skripty jen se zdůvodněním — samošířící se npm
  červi z roku 2026 cílí i na konfigurace AI agentů (hooks, tasks.json) a klíče v `~/.claude`,
  `~/.codex` apod. (viz sources.md §F).
- 🟡 Nové závislosti ověřené proti halucinaci/typosquattingu (balíček reálně existuje,
  má historii a údržbu — AI občas navrhne neexistující nebo podvržený název).
- 🟡 Projekt je auditovatelný: migrační sada přehratelná od nuly do prázdné DB, deterministický
  generátor dat (nebo bezpečně anonymizovaný vzorek), klient přesměrovatelný na stínové
  prostředí, ADR a dokument invariantů existují. Náklad je nulový, pokud se na to myslí při
  vzniku migrací a seedu; při auditu je nesplatitelný — bez toho nelze spustit polovinu
  prověrek, včetně těch s nejvyšší výtěžností (chování při selhání).
- 🟡 Stínové/testovací prostředí není dosažitelné zvenčí. Kontrola po **každém** startu
  prostředí, ne jednou — ochranná pravidla nepřežijí restart a kontejnerová vrstva umí obejít
  host firewall (publikace portu na všechna rozhraní). Data bývají syntetická, vstupní bod
  na server ne.

## G. Manuální testy ownera (co agent nezvládne za tebe)

Inspirováno vrstvou „manual-checklist" z benavlabs/vibe-check: některé kontroly vyžadují
člověka s prohlížečem a druhým účtem. Periodicky (a po každé větší změně přístupů):

- Druhý účet: přihlas se jako jiný uživatel/role a zkus přes URL/ID otevřít cizí záznamy
  (IDOR test v reálném prohlížeči, ne jen v SQL).
- Odhlášený stav: zkus přímé URL na chráněné routy a přímé volání API s anon klíčem.
- Restore drill: jednou skutečně obnov zálohu do izolovaného prostředí a ověř data.
- Chybové stavy: vyvolej chybu (neplatný vstup, neexistující route) a sleduj, co uživatel
  reálně vidí.
- Porucha uprostřed načtení: vypni síť (DevTools offline) nebo zablokuj API po otevření
  plochy a sleduj — uvidíš chybu, nebo prázdnou plochu, která vypadá jako „žádná data"?
- Dva prohlížeče nad jedním záznamem: oba otevřou, oba uloží. Přežila změna prvního,
  nebo tiše zmizela?

## Jak checklist používat

1. Před nasazením projdi relevantní sekce (minimálně A, B, E u každého deploye; C, D podle
   povahy změny; F periodicky a při auditu).
2. Každé „ne" u 🔴 = stop. Každé „ne" u 🟡 bez zapsaného rozhodnutí = stop, dokud rozhodnutí
   nevznikne (stačí řádek v dokumentu invariantů, nebo ADR, pokud je to load-bearing).
3. Nálezy, které nejsou blokující, ale zaslouží fix → backlog s vlastníkem, ne TODO v kódu.
4. Čistý výsledek jednorázové kontroly převeď na trvalé pravidlo (lint, CI assert, smoke).
   Jinak se příští rok zhorší a nikdo si nevšimne — dobrý výsledek je důvod pravidlo
   zafixovat, ne kontrolu zahodit.
5. Prahy naměřené na jednom projektu (délka URL, strop řádků, šířka rozpadu layoutu) se
   nepřenášejí; přenáší se metoda, kterou se hrana najde.
