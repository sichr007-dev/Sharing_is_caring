---
name: vibecoding-production-guard
description: >-
  Produkční gate pro AI-asistované změny v aplikacích s produkční ambicí (libovolný stack).
  Použij VŽDY, když se změna dotýká auth/autorizace,
  RLS/policies/grants, migrací či schématu, validace vstupů, stavových přechodů/workflow,
  secrets, serverless funkcí nebo nasazení — a KDYKOLI hrozí „prompt-patching": oprava
  viditelného symptomu („oprav login", „CORS nefunguje", „RLS blokuje dotaz") bez posouzení
  celého toku. Vynucuje analytický gate před změnou, tři defektní optiky (neúplné šíření
  změny, oslabení obranné vrstvy fixem, nevyslovený invariant) a binární stop-ship checklist
  před nasazením. Použij i při auditu či verifikaci nálezů — kryje chyby detekce samotné:
  falešné PASS, nereprodukovatelné nálezy, test přítomnosti obrany místo chování. Trigger
  i na anglické „fix the login", „make it work". NEpoužívej na čistě kosmetické
  UI změny bez dotyku dat a oprávnění ani na čistý research bez zásahu do kódu.
---

# Vibecoding Production Guard

Aplikace, které „fungují", nejsou totéž jako aplikace konzistentní, zabezpečené a obnovitelné.
AI je výborná v **lokální správnosti** (upravené tlačítko funguje); produkce stojí na **systémové
správnosti** (kontrakty, oprávnění, invarianty drží) a **provozní správnosti** (lze migrovat,
monitorovat, obnovit). Tento skill je gate, který brání tomu, aby lokální fix tiše prodal
systémovou nebo provozní vrstvu.

Nejde o byrokracii: tři incident-třídy, proti kterým skill stojí, jsou doložené výzkumem
vibe-coded aplikací (arXiv:2606.23130 — z 200 auditovaných nasazených aplikací mělo 90 %
aspoň jednu zranitelnost). Detaily a lovecké heuristiky:
[references/defect-classes.md](references/defect-classes.md).

**Scope poznámka:** příklady a checklisty předpokládají dnes nejběžnější vibecoding stack —
klientskou SPA + managed Postgres/BaaS (Supabase-style). Principy platí obecně; názvy
(RLS, granty, `pg_proc`…) si přelož na svůj stack.

## Kdy jsi v gate zóně

Změna spadá pod gate, když se dotýká alespoň jednoho z povrchů:

- **Přístup**: auth, autorizace, row-level security / policies, GRANT/REVOKE, role,
  serverless/edge funkce, privilegované service účty
- **Schéma a data**: migrace, constraint, seed, hromadný datafix
- **Stavy**: stavové přechody entit, workflow endpointy, whitelisty patchovatelných polí,
  stavové guardy v UI
- **Vstupy**: validace, parsování, cokoliv co skládá SQL/HTML/URL z uživatelských dat
- **Secrets**: env proměnné, klíče, tokeny (pozor na build-time prefixy typu `VITE_*`,
  `NEXT_PUBLIC_*` — vše s nimi končí v klientském bundlu)
- **Nasazení**: deploy pipeline, DB apply, změna CI/CD workflow

…**a zvlášť** když zadání zní jako oprava symptomu: „nefunguje X, oprav to". To je moment
nejvyššího rizika — výzkum ukazuje, že právě tady vznikají fixy, které projdou oslabením
obranné vrstvy.

## Krok 1 — Analytický gate (před změnou kódu)

Nejdřív analýza, pak diff. Minimálně:

1. **Příčina, ne symptom** — co je skutečná příčina? (Chyba „RLS/validace blokuje dotaz" je
   často správné chování ochrany, ne bug.)
2. **Celý tok** — zmapuj cestu od klientské komponenty přes query/API vrstvu až do úložiště.
   Vyjmenuj dotčené soubory, endpointy, tabulky, policies, RPC, secrets.
3. **Invarianty** — které systémové a bezpečnostní invarianty musí změna zachovat? Čti
   projektový dokument invariantů a relevantní specy/ADR (existují-li). Nerekonstruuj
   pravidla z paměti.
4. **Blast radius** — která další místa implementují stejné pravidlo? (whitelist, grants,
   guard v UI i na serveru…) Změna se musí rozšířit do všech, nebo explicitně zdůvodnit proč ne.
5. **Minimální diff** — žádné nesouvisející refaktory ve stejném commitu.

Plná verze gate a self-review checklist po implementaci:
[references/analytical-gate.md](references/analytical-gate.md).

## Krok 2 — Tři defektní optiky

Při návrhu i review změny se aktivně ptej na tři mechanismy vzniku chyb (empiricky
nejčastější u AI-asistovaného vývoje):

| Optika | Otázka | Typický příklad |
|---|---|---|
| **Neúplné šíření změny** | Rozšířil jsem pravidlo na *všechna* místa, kde platí? | nové pole bez záznamu ve whitelistu transition RPC; CREATE TABLE bez grantů v post-hardening DB |
| **Fix oslabením obrany** | Neřeším chybu tím, že vypnu/obejdu ochranu? | „RLS blokuje → použiju service účet"; „CORS zlobí → povolím vše"; „validace odmítá → smažu ji" |
| **Nevyslovený invariant** | Neporušuju pravidlo, které nikde není napsané? | datový klíč ≠ UI label jako záměrný rozpor — „oprava" je regrese |

Když při práci narazíš na invariant, který nikde není zapsaný, **zapiš ho jako kandidáta do
projektového dokumentu invariantů** (typicky `docs/INVARIANTS.md`; založ, pokud chybí).
To je ráčna: každý průchod gatem zmenšuje třídu „defektů znalostí" pro příště.

## Krok 3 — Self-review diffu (po implementaci)

Před commitem projdi vlastní diff jako nezávislý reviewer — nevylepšuj, jen hledej nálezy:
porušené invarianty, chybějící autorizace na serveru, neošetřené vstupy, únik secrets, chybějící
transakce, race conditions / duplicitní mutace, regresní dopad na jiné endpointy, chybějící
propagace pravidla (optika 1). Checklist v
[references/analytical-gate.md](references/analytical-gate.md) §Self-review.

## Krok 4 — Stop-ship před nasazením

Před deployem klienta nebo DB apply projdi **binární stop-ship checklist s důkazní
povinností** — každá položka ano/ne s konkrétním důkazem (soubor, výstup dotazu, log):
[references/stop-ship-checklist.md](references/stop-ship-checklist.md).

Tvrdé minimum (plný seznam v referenci):

- U odděleného nasazení DB a klienta jede DB změna **první**, klient **druhý**; nová serverová
  volání (RPC/endpointy) ověř v cílovém prostředí **přímým dotazem** (`pg_proc`, health check),
  ne z evidence migrací
- `git log origin/main..HEAD` před pushem — push bere celou větev včetně cizích commitů
- schéma se mění jen verzovanou migrací + smoke blokem v repu
- žádný secret/service klíč v bundlu (audit build-time env proměnných), žádná policy
  `using (true)` bez zapsaného rozhodnutí
- po reseedu/refreshi sdíleného prostředí je QA baseline neplatná ⇒ smoke re-run

## Když detekci provádíš ty (audit, verifikace, self-review)

Selhává i hledání chyb samo — systematicky směrem k **falešné jistotě**, a falešné PASS je
horší než falešný nález (falešný nález zabije verifikace; falešné PASS nezachytí nikdo).
Proto asymetricky: nálezy prohnat skeptikem/reprodukcí, ale „čisto" nepřijmout bez důkazu
práce a **funkčního negativního testu** (obrana se ověřuje chováním, ne přítomností).
Pět tříd chyb detekce + protiopatření:
[references/verification-failure-modes.md](references/verification-failure-modes.md).

## Co tento skill NEřeší

Gate je tenký záměrně — je to proces, ne architektonická příručka. Mimo scope: návrh schématu
a RBAC/RLS architektura, stavové modely entit, návrh seed/demo dat, psaní specifikací.
Na to si drž vlastní referenční materiály nebo specializované skilly. Veřejně dostupné
doplňky, které se s gatem dobře skládají:

| Téma | Kam |
|---|---|
| Supabase auth/klienti/SSR, kritické vzory | oficiální `supabase` skill (`npx skills add supabase/agent-skills`) |
| Postgres výkon a best practices | `supabase-postgres-best-practices` (Supabase) |
| hloubkový bezpečnostní scan diffu/větve | vestavěný `/security-review` (Claude Code) |
| manuální kontroly člověkem | benavlabs/vibe-check (inspirace pro sekci G stop-shipu) |

Unikátní území tohoto skillu: **moment gate** (kdy se zastavit), **tři optiky**, **stop-ship
verdikt s důkazní povinností**, **disciplína malého diffu** a **chyby detekce**. Nic víc —
pokud tě láká sem přidat pravidlo o RLS syntaxi, patří do architektonické reference, ne sem.
