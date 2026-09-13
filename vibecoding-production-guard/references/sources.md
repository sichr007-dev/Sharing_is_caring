# Zdroje — anotovaná bibliografie

Co skill tvrdí a o co se to opírá. U každého zdroje je jedna věta, **co pro skill dokládá**,
aby šlo při příští revizi ověřit, jestli tvrzení ještě platí. Stav ověření: 2026-09-13.
Pozor na verze: arXiv preprinty se mění (2606.23130 změnil klíčová čísla mezi v1 a v3);
citovat vždy s verzí a datem.

## A. Bezpečnost AI-generovaného kódu — empirie

- **Deng, Fan, Meng — *Understanding the (In)Security of Vibe-Coded Applications***,
  arXiv:2606.23130, v3 2026-09-05. https://arxiv.org/abs/2606.23130
  Korpus 10 517 vibe-coded aplikací, hloubkový audit 200 nasazených: 91,0 % s aspoň jednou
  zranitelností, 65,77 % nálezů Critical/High; osm opakujících se failure modes odvozených ze
  tří limitů agentů (memory / objective / knowledge defects). **Dokládá:** defektní třídy 1–3
  a rámování „mechanismus, ne kategorie zranitelnosti". Lepší harness a prompting incidenci
  snižují, riziko neodstraňují.
- **Zhao et al. — *Is Vibe Coding Safe? Benchmarking Vulnerability of Agent-Generated Code in
  Real-World Tasks* (SUSVIBES)**, arXiv:2512.03262, ICML 2026. https://arxiv.org/abs/2512.03262
  186 feature-request úloh z reálných projektů: u nejlepší konfigurace 57 % řešení funkčně
  správných, jen 11,8 % bezpečných; hinty o zranitelnostech nepomohly. **Dokládá:** lokální
  správnost ≠ systémová správnost — jádro motivace skillu.
- **Veracode — *2026 GenAI Code Security Report***, 2026-07-28.
  https://www.veracode.com/resources/analyst-reports/2026-genai-code-security-report/
  100+ modelů, čtyři měřicí vlny od 2025: průměrný security pass rate 56 %, beze změny
  meziročně; nejlepší model 68 %. **Dokládá:** zlepšení modelů nepřináší zlepšení bezpečnosti
  výstupu; gate je potřeba nezávisle na modelu.
- **Cloud Security Alliance — *Vibe Coding's Security Debt: The AI-Generated CVE Surge***,
  research note, 04/2026. https://labs.cloudsecurityalliance.org/research/csa-research-note-ai-generated-code-vulnerability-surge-2026/
  CVE atribuované AI kódu: 2,74× meziroční nárůst v Q1 2026; AI-asistovaní vývojáři commitují
  3–4× rychleji a zavádějí bezpečnostní nálezy 10× častěji. **Dokládá:** dluh roste rychleji,
  než se splácí — argument pro gate před nasazením, ne po něm.
- **Liu et al. — *Debt Behind the AI Boom: A Large-Scale Empirical Study of AI-Generated Code
  in the Wild***, arXiv:2603.28592, 03–04/2026. https://arxiv.org/abs/2603.28592
  302 600 ověřených AI commitů z 6 299 repozitářů: 23 % zavedených problémů přetrvává
  v aktuální verzi. **Dokládá:** co projde, zůstane — ráčna invariantů a trvalá pravidla
  místo jednorázových kontrol.
- **Nogueira, Vieira, Campos — *Unreliable in Practice? A Comprehensive Study of Errors in
  LLM-Generated Code***, arXiv:2608.00661, ISSRE 2026. https://arxiv.org/abs/2608.00661
  86 726 chybných vzorků od sedmi LLM: generovaný kód často vynechává základní validaci
  vstupů a bezpečnostní kontroly; i největší modely dělají jednoduché chyby. **Dokládá:**
  stop-ship D (runtime validace na hranici) a třídu 3 (nevyslovené požadavky).

## B. Tiché selhání a ošetření chyb

- **Yuan et al. — *Simple Testing Can Prevent Most Critical Failures: An Analysis of
  Production Failures in Distributed Data-Intensive Systems***, USENIX OSDI 2014.
  https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-yuan.pdf
  198 produkčních selhání (Cassandra, HBase, HDFS, MapReduce, Redis): **92 % katastrofických
  selhání = špatné ošetření nefatální, explicitně signalizované chyby**; ~⅓ triviálně
  odhalitelných inspekcí (prázdný handler, abort v over-catch, TODO); 77 % reprodukovatelných
  unit testem; statický checker Aspirator našel 500 nových chyb. **Dokládá:** defektní třída 4
  není okrajová a levné kontroly ji chytají.
- **OWASP Top 10:2025 — A10 *Mishandling of Exceptional Conditions***, 11/2025, finální 01/2026.
  https://top10.owasp.org/2025/A10_2025-Mishandling_of_Exceptional_Conditions
  Nová kategorie, 24 CWE (mj. CWE-636 Failing Open, CWE-234 Failure to Handle Missing
  Parameter, CWE-209 chybová hláška s citlivými daty). Prevence: chytat chybu tam, kde
  vzniká, a ošetřit smysluplně; fail closed; rollback transakcí; centralizované ošetření.
  **Dokládá:** třída 4 a stop-ship D.
- **Wu — *When Errors Become Narratives: A Longitudinal Taxonomy of Silent Failures in a
  Production LLM Agent Runtime***, arXiv:2606.14589, 06/2026. https://arxiv.org/abs/2606.14589
  22 incidentů s postmortemy; pět tříd tichých selhání (prostředí, nesoulad návrhových
  předpokladů, **error swallowing and dilution**, řetězená halucinace, provozní slepé skvrny);
  ~70 % odhalil člověk, ne testy; latence odhalení 13 h – 60 dní; doporučení „loud,
  attributable, boring failures". **Dokládá:** třída 4, §6 chyb detekce, a proč se ticho
  neeskaluje jako incident.
- **Basiri et al. — *Chaos Engineering***, IEEE Software 2016; *Principles of Chaos
  Engineering*, https://principlesofchaos.org/
  Hypotéza o ustáleném stavu, reálné poruchy, minimalizace vedlejších účinků. **Dokládá:**
  matici porucha × plocha jako legitimní auditní nástroj — a proč se ptá na uživatelsky
  viditelný dopad, ne na vnitřní stav.

## C. Souběh a ztracený zápis

- **Kleppmann — *Designing Data-Intensive Applications***, O'Reilly 2017, kap. 7
  „Preventing Lost Updates". Read-modify-write cyklus, atomické operace, explicitní zámky,
  compare-and-set (`UPDATE … WHERE id = ? AND version = ?`), automatická detekce ztracených
  zápisů. **Dokládá:** stop-ship C položku o ztraceném zápisu a heuristiku testu dvou sezení.
- **Supabase discussion #40482** (Storage versioning / edit locking), doporučení používat
  Postgres: optimistický update přes `WHERE version = X`, nebo tabulku zámků.
  https://github.com/orgs/supabase/discussions/40482
  **Dokládá:** v BaaS bez dlouhých transakcí je compare-and-set jediná praktická cesta;
  `SELECT … FOR UPDATE` přes auto-commit API nedrží.

## D. Limity stacku (Supabase-style)

- **PostgREST — konfigurace `db-max-rows`**, https://docs.postgrest.org/en/stable/references/configuration.html
  „A hard limit to the number of rows PostgREST will fetch from a view, table, **or
  function**. Limits payload size for accidental or malicious requests." Výchozí ∞; Supabase
  nastavuje per projekt (typicky 1000). **Dokládá:** tichý ořez platí i pro RPC; limit je
  ochrana, zvednutí stropu místo stránkování je třída 2.
- **PostgREST — stránkování a count**, https://docs.postgrest.org/en/stable/references/api/pagination_count.html
  `Prefer: count=exact|planned|estimated`, `Content-Range`, 206 Partial Content. **Dokládá:**
  detekce ořezu je možná jen s explicitním počtem nebo stránkováním, ne z tvaru odpovědi.
- **postgrest-js issues #393, #423** — `.in()` filtr s větším seznamem ID → „URI too long"
  (414) z brány; doporučení POST / RPC / dávkování.
  https://github.com/supabase/postgrest-js/issues/393 · https://github.com/supabase/postgrest-js/issues/423
  **Dokládá:** třída 4 „tvrdý limit infrastruktury bez odpovědi" a stop-ship D položku o
  filtrech ze seznamu ID.

## E. Metodika měření

- **van der Kouwe et al. — *Benchmarking Crimes: An Emerging Threat in Systems Security***,
  arXiv:1801.02381; rozšířeno jako *SoK: Benchmarking Flaws in Systems Security*, IEEE
  EuroS&P 2019. https://arxiv.org/abs/1801.02381
  22 „benchmarking crimes" (chybějící baseline, nespecifikovaná platforma, selektivní
  metriky…) v 50 špičkových pracích. **Dokládá:** §6 chyb detekce — měřicí přístroj má
  vlastní chybu a musí ji zapisovat; a pravidlo „metrika bez jediné definice se nepoužívá
  jako kritérium" (dvě měření duplicity na témže kódu dají různá čísla, když měří jiný rozsah).

## F. Supply chain a publikovaný artefakt

- **OWASP Top 10:2025 — A03 *Software Supply Chain Failures***, https://top10.owasp.org/
  Nová kategorie; SSRF sloučeno do Broken Access Control. **Dokládá:** stop-ship F položky
  o závislostech.
- **Shai-Hulud / ChainDrop — vlny samošířících se npm červů, 2025–2026.** CSA Singapore
  advisory AD-2026-009 (https://www.csa.gov.sg/alerts-and-advisories/advisories/ad-2026-009/);
  Datadog Security Labs (https://securitylabs.datadoghq.com/articles/npm-worm-compromises-popular-npm-packages/);
  Unit 42 *The npm Threat Landscape* (aktualizace 2026-07-15,
  https://unit42.paloaltonetworks.com/monitoring-npm-supply-chain-attacks/); CISA alert
  k axios 2026-04-20. Kompromitace `keyv` a dalších 2026-08-04; přes 1 300 verzí balíčků;
  exfiltrace cloud credentials, **konfigurací AI agentů**, perzistence přes Claude Code hooks
  a VS Code `tasks.json`; jeden token maintainera → stovky otrávených balíčků bez lidského
  zásahu. **Dokládá:** nový vektor od 07/2026 — instalace z lockfile, cooldown nových verzí,
  audit postinstall skriptů, klíče agentů jako secrets.
- **Xie — *VibeGuard: A Security Gate Framework for AI-Generated Code — Lessons from the
  Claude Code Source Leak***, arXiv:2604.01052, 04/2026. https://arxiv.org/abs/2604.01052
  Incident 2026-03-31: 59,8 MB source map s ~512 000 řádky zdrojáků v publikovaném balíčku —
  chyba packaging konfigurace, ne logiky. Gate s pěti kategoriemi: hygiena artefaktu, drift
  packaging konfigurace, expozice source map, hardcodované secrets, supply chain. **Dokládá:**
  stop-ship A položku o obsahu artefaktu — logický review tuhle třídu nevidí.

## G. Agentní vektory (relevantní pro aplikace, které samy volají LLM)

- **OWASP Top 10 for Agentic Applications 2026**, 2025-12-09.
  https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/
  ASI01 Goal Hijack, ASI02 Tool Misuse, ASI03 Identity & Privilege Abuse, ASI04 Agentic
  Supply Chain, ASI05 Unexpected Code Execution, ASI06 Memory/Context Poisoning, ASI07
  Inter-Agent Communication, ASI08 Cascading Failures, ASI09 Human-Agent Trust, ASI10 Rogue
  Agents. **Dokládá:** stop-ship D položku pro aplikace s LLM funkcemi. Mimo scope gate jako
  celek — katalog „co je rozbité", ne mechanismus vzniku.
- **Cloud Security Alliance — *Agentjacking: MCP Injection Hijacks AI Coding Agents***,
  research note 2026-06-12. https://labs.cloudsecurityalliance.org/research/csa-research-note-agentjacking-mcp-sentry-injection-20260612/
  Nepřímá prompt injection přes obsah, který agent čte (issue, ticket, log) → vytvoření
  škodlivé MCP konfigurace → RCE. **Dokládá:** riziko na straně *nástroje*, kterým se skill
  spouští: subagenti a auditní prompty čtou obsah cizího repa jako data, ne jako instrukce.

## H. Komplementy (ne zdroje tvrzení)

- **benavlabs/vibe-check** — https://github.com/benavlabs/vibe-check — inspirace pro vrstvu
  manuálních kontrol člověkem (stop-ship G).
- **supabase/agent-skills** — `npx skills add supabase/agent-skills` — oficiální vzory pro
  auth, klienty, SSR; architektonická vrstva, kterou gate záměrně neduplikuje.

## Jak s tímto seznamem pracovat

1. Při revizi skillu projít A–G a ověřit, zda citovaná čísla a verze ještě platí (arXiv
   verze, roční reporty).
2. Nový zdroj se přidává jen s větou „co dokládá pro skill". Zdroj bez vazby na konkrétní
   pravidlo skillu sem nepatří.
3. Prahové hodnoty z jednotlivých studií nebo projektů (1000 řádků, 8 kB URL…) jsou
   ilustrace stacku, ne pravidla. Pravidlem je metoda, kterou se hrana najde.
