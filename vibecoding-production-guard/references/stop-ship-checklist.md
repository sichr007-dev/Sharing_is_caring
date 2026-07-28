# Stop-ship checklist — binární, s důkazní povinností

Každá položka se zodpovídá **ano/ne + důkaz** (soubor:řádek, výstup dotazu, log, screenshot).
„Myslím, že ano" = ne. Položky označené 🔴 blokují nasazení vždy; 🟡 blokují, pokud pro ně
neexistuje zapsané vědomé rozhodnutí (ADR nebo dokument invariantů).

Formulováno pro typický stack „klientská SPA + managed Postgres/BaaS" (React + Supabase apod.),
principy platí obecně — přelož si názvy na svůj stack.

## A. Secrets a klíče

- 🔴 Žádný service/secret klíč v klientském kódu, repu ani logu.
  Důkaz: grep bundlu/`src/` na klíčové prefixy; kontrola build-time env proměnných
  (vše s prefixem typu `VITE_*`/`NEXT_PUBLIC_*` končí v bundlu — smí tam být jen
  anon/publishable klíče a veřejné URL).
- 🔴 Žádné hardcodované credentials, tokeny, hesla v kódu ani migracích.
  Pozn.: credential lifecycle (generování + založení účtu + zápis) nepatří do jednoho
  shell skriptu — AV heuristiky to flagují jako malware a na Windows soubor rovnou smažou.

## B. Přístup (autentizace ≠ autorizace)

- 🔴 Každá tabulka s daty má RLS zapnutou a policy, která skutečně filtruje.
  Důkaz: dotaz na `pg_policies` + `pg_tables.rowsecurity`; každá `using (true)` policy má
  zapsané rozhodnutí, proč je OK.
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

## D. Vstupy a výstupy

- 🔴 Žádné skládání SQL/HTML/URL/path z uživatelského vstupu bez parametrizace/sanitizace;
  `dangerouslySetInnerHTML` (a ekvivalenty) jen se sanitizovaným obsahem.
- 🔴 Runtime validace vstupů na hranici (statické typy za běhu neexistují; žádné slepé
  `update({ ...body })`).
- 🟡 Produkční chybové stavy neprozrazují interní detaily (stack trace, SQL chyby) uživateli.
- 🟡 Vstupy přijímající URL nebo soubory (import, webhook, upload, náhled externí stránky)
  mají omezení: typ/velikost souboru, allowlist cílů, žádný přístup do interní sítě (SSRF).
- 🟡 Neautentizované vstupní body (feed tokeny, webhooky) mají rate limiting nebo aspoň
  doložené rozhodnutí, proč ho nepotřebují; tokeny jsou revokovatelné.
- 🟡 Bezpečnostní HTTP hlavičky (CSP, X-Frame-Options…) nastavené, nebo zapsané rozhodnutí,
  že u interní aplikace za loginem zatím ne.

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

## F. Provoz a obnova

- 🟡 Vlastnosti zálohy známé (co, jak často, retence) a obnova alespoň jednou vyzkoušená;
  bez toho je „platforma dělá backup" jen domněnka.
- 🟡 Kritické operace zanechávají auditní stopu (kdo, kdy, co); aplikační chyby jsou někde
  vidět, ne jen v konzoli prohlížeče.
- 🟡 Závislosti bez známých kritických zranitelností (lockfile + občasný audit; „funguje"
  ≠ „může zůstat navždy").
- 🟡 Nové závislosti ověřené proti halucinaci/typosquattingu (balíček reálně existuje,
  má historii a údržbu — AI občas navrhne neexistující nebo podvržený název).

## G. Manuální testy ownera (co agent nezvládne za tebe)

Inspirováno vrstvou „manual-checklist" z benavlabs/vibe-check: některé kontroly vyžadují
člověka s prohlížečem a druhým účtem. Periodicky (a po každé větší změně přístupů):

- Druhý účet: přihlas se jako jiný uživatel/role a zkus přes URL/ID otevřít cizí záznamy
  (IDOR test v reálném prohlížeči, ne jen v SQL).
- Odhlášený stav: zkus přímé URL na chráněné routy a přímé volání API s anon klíčem.
- Restore drill: jednou skutečně obnov zálohu do izolovaného prostředí a ověř data.
- Chybové stavy: vyvolej chybu (neplatný vstup, neexistující route) a sleduj, co uživatel
  reálně vidí.

## Jak checklist používat

1. Před nasazením projdi relevantní sekce (minimálně A, B, E u každého deploye; C, D podle
   povahy změny; F periodicky a při auditu).
2. Každé „ne" u 🔴 = stop. Každé „ne" u 🟡 bez zapsaného rozhodnutí = stop, dokud rozhodnutí
   nevznikne (stačí řádek v dokumentu invariantů, nebo ADR, pokud je to load-bearing).
3. Nálezy, které nejsou blokující, ale zaslouží fix → backlog s vlastníkem, ne TODO v kódu.
