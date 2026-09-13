# Vibecoding Production Guard

Skill pro Claude Code (a kompatibilní AI coding agenty): **produkční gate pro AI-asistované
změny** v aplikacích, které mají produkční ambici. Destilovaný z praxe hardeningu a auditů
AI-asistovaně vyvíjených aplikací a ukotvený ve výzkumu bezpečnosti vibe-coded aplikací.

Aktuální verze: **1.1** (2026-09-13). Historie změn: [CHANGELOG.md](CHANGELOG.md).

## Proč

Aplikace, které „fungují", nejsou totéž jako aplikace konzistentní, zabezpečené a obnovitelné.
Z 200 auditovaných nasazených vibe-coded aplikací jich 91 % obsahovalo aspoň jednu zranitelnost
a 65,77 % nálezů bylo Critical/High (Deng, Fan, Meng: *Understanding the (In)Security of
Vibe-Coded Applications*, [arXiv:2606.23130](https://arxiv.org/abs/2606.23130), v3 09/2026).
Většina těch chyb nevzniká neznalostí zranitelností, ale opakovanými **mechanismy**
AI-asistované práce: neúplným šířením změny, fixem přes oslabení obranné vrstvy, nevysloveným
invariantem — a tichým selháním, kdy se chyba převede na prázdnou obrazovku bez signálu.
Tenhle skill je proces, který ty mechanismy chytá dřív, než dojedou do produkce.

## Co je uvnitř

| Soubor | Obsah |
|---|---|
| `SKILL.md` | tenký gate: kdy se zastavit, 4 kroky, tvar výstupu auditu, co je mimo scope |
| `references/defect-classes.md` | čtyři defektní třídy s ilustrativními scénáři a loveckými heuristikami |
| `references/analytical-gate.md` | 10bodový gate před změnou + self-review checklist diffu |
| `references/stop-ship-checklist.md` | binární checklist před nasazením (🔴/🟡) s důkazní povinností |
| `references/verification-failure-modes.md` | sedm tříd chyb detekce — jak selhává audit samotný — a důkazní žebřík |
| `references/audit-output.md` | tvar výstupu auditu: třídění, stavy, jedna ID řada, hranice mandátu |
| `references/sources.md` | anotovaná bibliografie: co skill tvrdí a o co se to opírá |

Reference o chybách detekce je nejméně obvyklá část: neřeší chyby v kódu, ale chyby v jejich
*hledání* — falešné PASS, plauzibilní nereprodukovatelné nálezy, testování přítomnosti
obrany místo jejího chování, slepé skvrny měřicího nástroje. Vychází z praxe multi-agentních
auditů s adversariální verifikací nálezů.

## Instalace

Zkopíruj adresář do skills lokace svého agenta:

```bash
# Claude Code — globálně (všechny projekty)
cp -R vibecoding-production-guard ~/.claude/skills/

# nebo per-projekt
cp -R vibecoding-production-guard <projekt>/.claude/skills/
```

Skill se aktivuje automaticky, když se změna dotkne gate povrchů (auth, schéma, stavy,
vstupy, secrets, nasazení, chování při selhání), nebo když zadání zní jako oprava symptomu
(„oprav login", „vrací to jen 1000 řádků").

## Scope

Příklady předpokládají dnes nejběžnější vibecoding stack: klientská SPA + managed
Postgres/BaaS (Supabase-style). Principy platí obecně — názvy si přelož na svůj stack.
Scénáře v referencích jsou ilustrativní: destilované vzory, ne případové studie
konkrétního systému. Prahové hodnoty (strop řádků, délka URL) jsou vlastnosti konkrétní
infrastruktury; přenáší se metoda, kterou se hrana najde, ne číslo.

## Poděkování a zdroje

Úplný seznam s anotacemi: [references/sources.md](references/sources.md). Klíčové opory:

- Deng, Fan, Meng: *Understanding the (In)Security of Vibe-Coded Applications*,
  arXiv:2606.23130 (2026) — rámování defektních tříd 1–3
- Yuan et al.: *Simple Testing Can Prevent Most Critical Failures*, USENIX OSDI 2014 —
  empirický základ třídy 4 (tiché selhání)
- OWASP Top 10:2025, kategorie A10 *Mishandling of Exceptional Conditions* a A03 *Software
  Supply Chain Failures*
- [benavlabs/vibe-check](https://github.com/benavlabs/vibe-check) — inspirace pro vrstvu
  manuálních kontrol člověkem (sekce G stop-ship checklistu)

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — viz [LICENSE](LICENSE).
Používej, upravuj, sdílej; zachovej uvedení zdroje.
