# Vibecoding Production Guard

Skill pro Claude Code (a kompatibilní AI coding agenty): **produkční gate pro AI-asistované
změny** v aplikacích, které mají produkční ambici. Destilovaný z praxe hardeningu a auditů
AI-asistovaně vyvíjených aplikací a ukotvený ve výzkumu bezpečnosti vibe-coded aplikací.

## Proč

Aplikace, které „fungují", nejsou totéž jako aplikace konzistentní, zabezpečené a obnovitelné.
Z 200 auditovaných nasazených vibe-coded aplikací jich 90 % obsahovalo aspoň jednu zranitelnost
a 76,7 % nálezů bylo Critical/High (Deng, Fan, Meng: *Understanding the (In)Security of
Vibe-Coded Applications*, [arXiv:2606.23130](https://arxiv.org/abs/2606.23130), 06/2026).
Většina těch chyb nevzniká neznalostí zranitelností, ale třemi opakovanými **mechanismy**
AI-asistované práce: neúplným šířením změny, fixem přes oslabení obranné vrstvy a nevysloveným
invariantem. Tenhle skill je proces, který ty mechanismy chytá dřív, než dojedou do produkce.

## Co je uvnitř

| Soubor | Obsah |
|---|---|
| `SKILL.md` | tenký gate: kdy se zastavit, 4 kroky, co je mimo scope |
| `references/defect-classes.md` | tři defektní třídy s ilustrativními scénáři a loveckými heuristikami |
| `references/analytical-gate.md` | 9bodový gate před změnou + self-review checklist diffu |
| `references/stop-ship-checklist.md` | binární checklist před nasazením (🔴/🟡) s důkazní povinností |
| `references/verification-failure-modes.md` | pět tříd chyb detekce — jak selhává audit samotný |

Poslední reference je nejméně obvyklá část: neřeší chyby v kódu, ale chyby v jejich
*hledání* — falešné PASS, plauzibilní nereprodukovatelné nálezy, testování přítomnosti
obrany místo jejího chování. Vychází z praxe multi-agentních auditů s adversariální
verifikací nálezů.

## Instalace

Zkopíruj adresář do skills lokace svého agenta:

```bash
# Claude Code — globálně (všechny projekty)
cp -R vibecoding-production-guard ~/.claude/skills/

# nebo per-projekt
cp -R vibecoding-production-guard <projekt>/.claude/skills/
```

Skill se aktivuje automaticky, když se změna dotkne gate povrchů (auth, schéma, stavy,
vstupy, secrets, nasazení), nebo když zadání zní jako oprava symptomu („oprav login").

## Scope

Příklady předpokládají dnes nejběžnější vibecoding stack: klientská SPA + managed
Postgres/BaaS (Supabase-style). Principy platí obecně — názvy si přelož na svůj stack.
Scénáře v referencích jsou ilustrativní: destilované vzory, ne případové studie
konkrétního systému.

## Poděkování a zdroje

- Deng, Fan, Meng: *Understanding the (In)Security of Vibe-Coded Applications*,
  arXiv:2606.23130 (2026) — rámování defektních tříd
- [benavlabs/vibe-check](https://github.com/benavlabs/vibe-check) — inspirace pro vrstvu
  manuálních kontrol člověkem (sekce G stop-ship checklistu)

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — viz [LICENSE](LICENSE).
Používej, upravuj, sdílej; zachovej uvedení zdroje.
