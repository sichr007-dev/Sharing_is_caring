# Changelog

Formát: verze, datum, co se změnilo a **proč** (odkud přírůstek vznikl). Skill je produkt,
ne dokumentace: po každém auditním běhu se kontroluje delta. Prázdná delta je signál.

## 1.1 — 2026-09-13

Přírůstek z jednoho kompletního auditního běhu nad produkční SPA + BaaS aplikací (čtyři vlny:
statika, model, výkon, odolnost) a z rešerše literatury od poslední verze. Anonymizováno;
přenášejí se vzory a metody, ne čísla.

**Přidáno**

- Čtvrtá defektní optika **tiché selhání** (SKILL.md, defect-classes.md §4): prázdný
  výsledek místo chyby, chybové stavy jen u akcí, tichý ořez na stropu API, tvrdý limit
  infrastruktury bez odpovědi, ztracený zápis, pravidlo které existuje a neplatí. Lovecké
  heuristiky: poměr chybových stavů k čtecím dotazům, matice porucha × plocha, desetinásobek
  dat, bisekce limitu, test dvou sezení.
- Gate bod **Chování při selhání** (analytical-gate.md, bod 8) a příslušné položky
  self-review.
- Stop-ship: 🔴 chybový stav čtecí cesty odlišitelný od prázdna; 🟡 stránkování / count
  kontrola; 🟡 filtry ze seznamu ID; 🟡 ztracený zápis (compare-and-set); 🟡 obsah
  publikovaného artefaktu (source mapy); 🟡 aplikace s LLM funkcemi; 🟡 auditovatelnost
  projektu; 🟡 expozice stínového prostředí; 🟡 mezistav klient/DB; zpřísněná položka
  o závislostech (lockfile, cooldown, postinstall). Sekce G: porucha uprostřed načtení,
  dva prohlížeče nad jedním záznamem. Pravidla 4–5 v „Jak používat" (čistá kontrola → trvalé
  pravidlo; nepřenášet čísla).
- Chyby detekce: §6 **Slepá skvrna měřicího nástroje** (pět zaznamenaných pastí),
  §7 **Důkazní žebřík** (změřeno / reprodukováno / experiment); §2 rozšířeno o verifikaci
  oprav diferenčním testem a pravidlo „grep hledá kandidáty, nikdy nedokazuje"; §3 o
  negativní test, který musí doložit, že k obraně doletěl.
- Nová reference **audit-output.md**: třídění vada / dluh / rozhodnutí / provoz / prověřeno
  bez nálezu; čtyři stavy; jedna ID řada; hranice mandátu; prostředí auditu jako povrch;
  předpoklady projektu; delta po běhu.
- Nová reference **sources.md**: anotovaná bibliografie s větou „co dokládá pro skill"
  u každého zdroje.

**Změněno**

- Citace arXiv:2606.23130 aktualizována na v3 (2026-09-05): 91,0 % aplikací se zranitelností,
  65,77 % nálezů Critical/High (v1 uváděla 90 % / 76,7 %). Doplněno mapování tříd 1–3 na
  memory / objective / knowledge defects ze studie.
- Třída 1 doplněna o scénář „guard došířený jen do části kopií" a heuristiku tabulky
  kopie × vlastnost. Třída 2 o podobu „zvednout strop místo stránkování". Třída 3 o vzor
  „chybějící rozhodovací kritérium".
- Popis skillu (frontmatter) rozšířen o trigger na chování při selhání.

**Nové vektory od verze 1.0 zohledněné v 1.1**

- OWASP Top 10:2025 A10 Mishandling of Exceptional Conditions a A03 Software Supply Chain
  Failures (finální 01/2026).
- Vlny samošířících se npm červů (Shai-Hulud, ChainDrop; srpen 2026) cílící na konfigurace
  a klíče AI coding agentů.
- Únik zdrojáků přes source mapy v publikovaném balíčku (03/2026) a gate frameworky
  na hygienu artefaktu.
- OWASP Top 10 for Agentic Applications 2026 pro aplikace, které samy volají LLM.

## 1.0 — 2026-07-28

První veřejná verze: tenký gate (4 kroky), tři defektní třídy z arXiv:2606.23130,
9bodový analytický gate, stop-ship checklist A–G, pět tříd chyb detekce.
