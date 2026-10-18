# Vibecoding Production Guard

Skill pro Claude Code a kompatibilní AI coding agenty: produkční gate pro
AI-asistované změny, připravenost vydání a vymezený audit projektu.

Aktuální verze: **1.3.0** (2026-10-18). Historie změn: [CHANGELOG.md](CHANGELOG.md).

## Co ověřuje

Pomáhá prověřit systémové kontrakty, oprávnění, integritu dat, chování při selhání,
výkon a provozní obnovu. Výsledek se vztahuje ke konkrétní revizi, prostředí a fázi
vydání. Funkční průchod sám nedokládá bezpečnost, použitelnou odezvu ani kapacitu.

Každá relevantní kontrola má **PASS / FAIL / UNKNOWN / N/A**, důkaz a rozsah.
Přijaté riziko se eviduje samostatně. Celkový verdikt je **READY**,
**READY WITH ACCEPTED RISKS** nebo **BLOCKED** pro pojmenovanou fázi.
Skill není certifikace projektu ani úplný audit podle bezpečnostního standardu.

## Co je uvnitř

| Soubor | Obsah |
|---|---|
| [SKILL.md](SKILL.md) | režim kontroly, čtyři defektní optiky, review a gate vydání |
| [Analytický gate](references/analytical-gate.md) | příčina, kontrakt, celý tok, souběh, dodání a výkonový dopad |
| [Defektní optiky](references/defect-classes.md) | neúplná propagace, oslabení obrany, nevyslovený invariant a tiché selhání |
| [Stop-ship checklist](references/stop-ship-checklist.md) | kontroly A–I s důkazy, relevancí a pravidly pro neověřený stav |
| [Chyby detekce](references/verification-failure-modes.md) | dosažitelnost nálezu, negativní test, legitimní data a platnost důkazů |
| [Výstup auditu](references/audit-output.md) | rozsah, záznamy, nálezy, fáze a hranice mandátu |
| [Výkon a škálovatelnost](references/performance-scalability.md) | profil práce, celý uživatelský tok, platnost zátěže, růst, správnost a zotavení |
| [Zdroje](references/sources.md) | bezpečnost, metodiky a jejich omezení |
| [Výkonové zdroje](references/performance-sources.md) | výzkum, oficiální dokumentace, otevřené nástroje a komunitní podněty |

## Výkon jako samostatná část auditu

Při auditu projektu nebo vydání se hodnotí šest oblastí I1–I6:

1. Úlohy, akceptační kritéria, data, očekávaná práce a prostředí.
2. Čas do použitelného výsledku napříč klientem, sítí, backendem a databází.
3. Vhodný model zátěže, plánovaná a dokončená práce i limity generátoru.
4. Růst dat a provozu, špičky, delší běh a zotavení podle rizika.
5. Správnost a úplnost výsledků, chyby, variabilita a reprodukovatelnost.
6. Samostatný verdikt s vymezením ověřené kapacity a neověřených oblastí.

U lokální změny se hloubka volí podle dopadu. Chybějící měření zůstane UNKNOWN.
Kritéria se odvozují z potřeb projektu; ukázkové latence, počty uživatelů nebo velikost
dat nejsou univerzální limity. Nástroje jako k6, Locust, sitespeed.io, Lighthouse CI
nebo wrk2 se vybírají podle měřené otázky a dostupného prostředí.

## Instalace a použití

Zkopíruj celý adresář do skills lokace svého agenta:

```bash
# Claude Code — globálně
cp -R vibecoding-production-guard ~/.claude/skills/

# nebo pro jeden projekt
cp -R vibecoding-production-guard <projekt>/.claude/skills/
```

Při aktualizaci zachovej zálohu a přenes konzistentní sadu souborů.
Automatický výběr závisí na agentním prostředí. Explicitní zadání může znít:

„Použij vibecoding-production-guard na tuto změnu / kandidáta vydání / vymezený audit.“

Skill zohledňuje místní pravidla, dostupné nástroje a již udělená oprávnění.
Aktivace skillu sama nepovoluje nasazení, zátěž produkce nebo odesílání dat do cloudu.

## Přenos mezi projekty a anonymizace

Scénáře jsou zobecněné ilustrace mechanismů chyb. Balíček neobsahuje interní auditní
reporty, projektové identifikátory, provozní data ani mapování nálezů na konkrétní systém.
Příklady často používají SPA a Postgres/BaaS; použitelnost posuď podle skutečného stacku.
Názvy větví, schvalovací postupy, architekturu ani číselné limity nepřenášej automaticky.

Čtyři optiky jsou pracovní model, nikoli úplná taxonomie nebo populační statistika.
Každý výzkumný zdroj má uvedené použití a omezení. Komunitní diskuse a cizí skilly
slouží jako podněty; technická pravidla mají oporu v primárních zdrojích.

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — viz [LICENSE](LICENSE).
Používej, upravuj, sdílej; zachovej uvedení zdroje.
