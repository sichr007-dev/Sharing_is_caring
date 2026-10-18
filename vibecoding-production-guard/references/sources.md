# Zdroje a rozhodnutí o jejich použití

Rozšíření **1.3.0** o výkon/škálovatelnost má vlastní
[registr výzkumu, nástrojů a diskusí](performance-sources.md). Níže zůstává
důkazní základ revize 1.2.0; jeho zdroje nebyly touto rozšiřující rešerší všechny znovu ověřovány.

Revize: **2026-10-18**, skill **1.2.0**. Jde o cílenou aktualizační rešerši,
nikoli systematický přehled veškeré literatury ani certifikaci bezpečnosti projektu.
U webových dokumentací je datum revize datem přístupu; neznamená datum publikace.
U GitHubu je níže cesta a větev; tyto zdroje nejsou pinované na commit.

## Jak vážit důkazy

- Primární studie dokládá výsledek ve svém vzorku; nikoli četnost vad všech dnešních modelů
  či konkrétního projektu. Rozlišuj napadení za běhu a nález statického scanneru.
- Oficiální standard/dokumentace dokládá požadavek nebo chování dané technologie/verze.
  OWASP Top 10 je přehled rizik; ASVS poskytuje ověřovací požadavky. Gate není plný ASVS audit.
- Vendor benchmark je primární měření výrobce s jeho testy a omezeními.
- GitHub skill je inspirace provedením, nikoli důkaz účinnosti nebo statistika.
- Reddit je zkušenostní signál s neověřenou identitou a samovýběrem.
  Technické pravidlo podepři nezávislým primárním zdrojem.
- Projektové incidenty v ostatních referencích jsou anonymizované zkušenosti, nikoli
  nezávisle publikovaná data. Kandidátní pravidlo nesmí přepsat projektový kontrakt.

Číselné výsledky níže se nesčítají ani nesrovnávají jako jedna metrika bezpečnosti.

## Primární studie a novější empirie

**P01 — Deng, Fan, Meng: Understanding the (In)Security of Vibe-Coded Applications.**
[arXiv:2606.23130v4](https://arxiv.org/abs/2606.23130v4), 2026-09-14.
V4 uvádí korpus 9 041 aplikací; audit 200 veřejně nasazených, 1 186 zranitelností,
91,0 % auditovaných aplikací s nálezem a 65,77 % nálezů Critical/High.
Proti starému seznamu se mění korpus 10 517 → 9 041, nikoli zde uvedené dva podíly.
**Použití:** zachovat první tři optiky; označit je jako praktickou interpretaci
memory/objective/knowledge defects. Omezení: vybraný vzorek konkrétních nástrojů,
ne odhad bezpečnosti všech AI projektů.

**P02 — Zhao et al.: Is Vibe Coding Safe? (SUSVIBES).**
[arXiv:2512.03262v4](https://arxiv.org/abs/2512.03262v4), 2026-09-21; ICML 2026.
186 úloh z projektů s historicky zranitelnými implementacemi, 12 konfigurací agentů.
Uváděných 57 % funkčně správných a 11,8 % bezpečných řešení se týká jedné konfigurace,
nikoli „nejlepší bezpečnosti všech agentů“. **Použití:** funkční průchod nenahrazuje
bezpečnostní důkaz. Výběr úloh není reprezentativní pro libovolnou aplikaci.

**P03 — Khalid et al.: (Don't) Trust, but (Don't) Verify.**
[arXiv:2609.21020v1](https://arxiv.org/abs/2609.21020v1),
[výsledky v plném textu](https://arxiv.org/html/2609.21020v1), 2026-09-17.
Pozorovací studie 100 účastníků nad čtyřmi úlohami v C: výběr nerozlišoval spolehlivě
bezpečnější návrhy; časté byly povrchní indicie a důraz na funkčnost/styl.
**Použití:** review má vyžadovat důkaz chování a identifikaci skutečné hranice.
Omezení: C linked-list úlohy; studie nedokazuje obecnou neschopnost lidského review
ani úspěšnost konkrétního webového gate.

**P04 — Surikov: Securing AI-Generated Code: A Just-in-Time Vulnerability Detection
and Remediation Pipeline.**
[arXiv:2608.16187v1](https://arxiv.org/abs/2608.16187v1), 2026-08-17.
Malý Python experiment nad 26 prompty kombinuje CodeQL, Bandit a LLM validátor;
opravy někdy zavádějí nové nálezy. **Použití:** po opravě prověřit regrese a varianty.
Omezení: studentský preprint, převážně statické nálezy; pokles nálezů není důkaz
plošné runtime bezpečnosti. Nepřebíráme univerzální procenta ani předepsaný toolchain.

**P05 — Vero et al.: BaxBench.**
[arXiv:2502.11844v3](https://arxiv.org/abs/2502.11844v3), 2025-05-30.
392 backend úloh, oddělené funkční testy a end-to-end exploity.
**Použití:** oddělit funkční a adversariální ověření, zachovat legitimní kontrolní případ.
Jde o starší doplnění mezery v bibliografii, ne novou událost po září 2026.

**P06 — Spracklen et al.: We Have a Package for You!**
[USENIX Security 2025](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen).
Studie halucinovaných balíčků napříč 16 modely a dvěma jazyky.
**Použití:** původ a identitu balíčku ověřovat nezávisle na doporučení LLM.
Starší měření není aktuální míra halucinací dnešního modelu.

**P07 — Yuan et al.: Simple Testing Can Prevent Most Critical Failures.**
[USENIX OSDI 2014](https://www.usenix.org/conference/osdi14/technical-sessions/presentation/yuan).
Produkční selhání distribuovaných systémů zdůrazňují ošetření explicitních chyb.
**Použití:** čtvrtá optika a cílené fault testy. Přenos do SPA je metodická inference,
nikoli doklad stejné četnosti selhání ve webových aplikacích.

**P08 — Nogueira, Vieira, Campos: Unreliable in Practice?**
[arXiv:2608.00661v1](https://arxiv.org/abs/2608.00661v1), 2026-08-01, ISSRE 2026.
Korpus chybových vzorků ve čtyřech kompilovaných jazycích popisuje i vynechanou validaci.
**Použití:** podpůrná motivace vstupních a resource kontrol; vzorek již chybných programů
není prevalence chyb mezi všemi generovanými programy.

**V01 — Veracode: 2026 GenAI Code Security Report.**
[Report](https://www.veracode.com/resources/analyst-reports/2026-genai-code-security-report/),
[shrnutí výrobce](https://www.veracode.com/blog/2026-genai-code-security-report-ai-risk/),
2026-07-28. Výrobce uvádí průměrný pass rate 56 % v jeho sadě bezpečnostních úloh.
**Použití:** funkční pokrok nelze automaticky přenášet na bezpečnost.
Není to nezávislý populační odhad; nepřidává nové pravidlo oproti P01/P02.

## Standardy a oficiální dokumentace → konkrétní kontroly

- **S01 [OWASP ASVS 5.0.0](https://owasp.org/projects/asvs)**:
  ověřitelné požadavky a verzované odkazy → B/D/F. Verzi ASVS u konkrétního mapování uvést;
  tento skill netvrdí pokrytí celé úrovně ASVS.
- **S02 [OWASP Top 10:2025](https://top10.owasp.org/2025/)**:
  A03 Software Supply Chain Failures a A10 Mishandling of Exceptional Conditions
  → F1–F3 a čtvrtá optika.
- **S03 [OWASP API4:2023](https://api-security.owasp.org/editions/2023/en/0xa4-unrestricted-resource-consumption/)**:
  objem, čas, rate a náklady externích služeb → D5; limity podle business potřeb.
- **S04 [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)**:
  automaticky přikládané credentials a volba ochrany podle mechanismu → B5.
  Nepřebíráme univerzální soud o každé SPA nebo samotném CORS.
- **S05 [Supabase API keys](https://supabase.com/docs/guides/getting-started/api-keys)**:
  publishable/anon vs. secret/service_role, aplikační klíč vs. uživatelská identita
  → A2/B1. Cloudový plán deprecace legacy klíčů automaticky nepřenášet na self-hosted stack.
- **S06 [Supabase RLS](https://supabase.com/docs/guides/database/postgres/row-level-security)**:
  grants + policies, exposed schemas a efektivní identita views → B2/B3.
  security_invoker u views má verzové podmínky; místní ratifikované výjimky mají kontext.
- **S07 [PostgREST pagination/count](https://docs.postgrest.org/en/stable/references/api/pagination_count.html),
  [db-max-rows](https://docs.postgrest.org/en/stable/references/configuration.html#db-max-rows)**:
  range/count, limity a cena exact count → D4. V revizi dokumentace odpovídala PostgREST 16;
  nasazenou verzi a konfiguraci je nutné ověřit.
- **S08 [npm ci, CLI v11](https://docs.npmjs.com/cli/v11/commands/npm-ci/)**:
  lockfile a řízení lifecycle skriptů → F1/F3.
  Volby allowScripts/ignore-scripts jsou verzově závislé; samotný lockfile není bezpečnostní audit.
- **S09 [GitHub Actions secure use](https://docs.github.com/en/actions/reference/security/secure-use)**:
  omezení tokenů, nedůvěryhodné vstupy a ověřený plný SHA → F3.
- **S10 [Stripe webhook endpoints](https://docs.stripe.com/events/manage-webhook-endpoints)**:
  podpis, raw payload podle protokolu a duplicitní události → C3.
  Princip přenášíme na jiné poskytovatele přes jejich dokumentaci, ne přes Stripe syntaxi.
- **S11 [OWASP Agentic Top 10 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)**:
  hranice instrukce/data, oprávnění nástrojů a řetězení efektů → podmíněná H.
- **S12 [OWASP Agent Control Standard](https://genai.owasp.org/resource/agent-control-standard-acs/)**,
  stránka z 2026-09-01: runtime enforcement přes kontrolní body → H1/H2.
  Zachyceno jako nový doplněk; nevyžaduje konkrétní framework ani implementaci ACS.

## Podobné skilly — metoda, nikoli převzatý kód

- **G01 [benavlabs/vibe-check](https://github.com/benavlabs/vibe-check),
  [SKILL.md](https://github.com/benavlabs/vibe-check/blob/main/skills/vibe-check/SKILL.md)**:
  vnější scan a kódové review mají jiné pokrytí; scanner není celý audit.
  Převzata myšlenka explicitně přiznat nepokryté plochy a rozdíl localhost/host.
  Nepřebíráme plošné FAIL pro source mapy, každý veřejný klíč nebo wildcard.
- **G02 [Trail of Bits differential-review](https://github.com/trailofbits/skills/blob/master/plugins/differential-review/skills/differential-review/SKILL.md)**:
  hloubka podle rizika, kontext historie a dopad na volající → gate/review.
- **G03 [Trail of Bits fp-check](https://github.com/trailofbits/skills/blob/master/plugins/fp-check/skills/fp-check/SKILL.md),
  [post-patch-validation](https://github.com/trailofbits/skills/blob/master/plugins/post-patch-validation/skills/post-patch-validation/SKILL.md)**:
  konkrétní předpoklady nálezu, pokus o vyvrácení, dosažení testované cesty a baseline/patch
  → verification. Inspirace principem; nepřebíráme runner, marker, povinné delegace či formát.
- **G04 [supabase/agent-skills](https://github.com/supabase/agent-skills)**:
  potvrzené oddělení obecného gate od stackové hloubky → routing v hlavním skillu.
  Nic neinstalováno. Oficiální dokumentace rozhoduje o technickém chování.

## Komunitní signály — nepoužívat jako míru rizika

- **C01 [How are you handling security…](https://www.reddit.com/r/vibecoding/comments/1uiu1c5/how_are_you_handling_security_in_your_vibecoded/)**:
  debata od 2026-06-29; témata runtime vs. source audit, deterministické kontroly
  a přehnané „opravy“ architektury. Vazba: P03/P05 a G02/G03; žádná tvrzení o četnosti.
- **C02 [Solving the citizen developer SDLC](https://www.reddit.com/r/EngineeringManagers/comments/1woj75v/solving_the_citizen_developer_sdlc/)**:
  diskuse 2026-09; odpovědnost a hranice sandbox/produkce. Vazba: smlouva kontroly
  a místní mandát; samostatně nedokládá účinnost postupu.
- **C03 [VibeCoding Security Playbook](https://www.reddit.com/r/AskVibecoders/comments/1rp954n/vibecoding_security_playbook/)**:
  diskuse 2026-03; checklistové rady a izolace prostředí. Nepřebíráme rady, že samotný
  NODE_ENV nebo prompt zajistí oddělení prostředí.

## Původní zdroje a tvrzení, která nyní nenesou pravidla

- CSA research note [Vibe Coding's Security Debt](https://labs.cloudsecurityalliance.org/research/csa-research-note-ai-generated-code-vulnerability-surge-2026/)
  byla otevřena. Je to syntéza jiných měření; původní 2,74× a 10× nepřenášíme,
  protože jejich přesný původ, jmenovatel a srovnatelnost nebyly v revizi doloženy.
- CSA [Agentjacking](https://labs.cloudsecurityalliance.org/research/csa-research-note-agentjacking-mcp-sentry-injection-20260612/)
  byla otevřena. Popisuje zprostředkovanou PoC studii; není důkaz plošného aktivního zneužití.
  Bezpečnost auditora v této verzi podpírají primárně S09/S11.
- [Singapore CSA AD-2026-009](https://www.csa.gov.sg/alerts-and-advisories/advisories/ad-2026-009/)
  bylo otevřeno, ale čitelný obsah se nepodařilo získat. Podrobnosti o jménech kampaní,
  počtech verzí a časové ose npm útoků proto netvoří důkazní základ aktualizace.
- [VibeGuard v1](https://arxiv.org/abs/2604.01052v1) byl ověřen v abstraktu:
  experiment s osmi syntetickými projekty. Hygiena artefaktu je užitečná; jeho výsledky
  nepřenášíme jako záruku účinnosti našeho skillu.
- Wu 2606.14589 a Liu et al. 2603.28592: odkazy otevřeny, nepoužity k novým závěrům;
  podrobná metodika/číselné závěry nebyly v této revizi znovu posouzeny.
- Kleppmann kap. 7, Supabase discussion #40482, postgrest-js #393/#423, Principles of Chaos
  Engineering a Benchmarking Crimes: historický kontext. Plné podklady nebyly nově ověřeny;
  původní kategorické „CAS je jediná praktická cesta“ se ruší. Pravidla se opírají o
  konkrétní invariant/test a oficiální dokumentaci, ne o tuto výlučnost.

## Příští revize

Při zásadní změně stacku/modelového runtime nebo novém relevantním incidentu ověř dotčené
pravidlo; nespoléhej na kalendářní stáří jako důkaz zastaralosti.
U nového zdroje uveď verzi/datum, metodu, omezení a konkrétní rozhodnutí pro skill.
Novinku, doplnění staré mezery, opravu tvrzení a změnu místního projektu označ odlišně.
