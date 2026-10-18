# Zdroje prověrky výkonu a škálovatelnosti

Revize **2026-10-18**, rozšíření guardu **1.3.0**. Cílená rešerše, nikoli systematický
přehled literatury ani měření účinnosti skillu. Otevřené zdroje byly čteny jako data;
žádný cizí skript/skill nebyl nainstalován. Pravidla I1–I6 jsou naše syntéza.
Datum přístupu není datum publikace. GitHub odkazy míří na uvedené živé větve,
ne na neměnný commit; při zavádění runneru pinovat ověřenou verzi zvlášť.

## Výzkum

| ID a primární zdroj | Co jsme použili | Omezení |
|---|---|---|
| **PS01** Dean, Barroso: [The Tail at Scale](https://research.google/pubs/the-tail-at-scale/), CACM 2013, autorský abstrakt | Sledovat tail a end-to-end složení latencí, ne jen průměr jedné vrstvy → I2/I5. | Velké distribuované služby; nepřebíráme jejich architekturu, 100ms cíl ani slib stejného efektu v malé SPA. |
| **PS02** Schroeder, Wierman, Harchol-Balter: [Open Versus Closed: A Cautionary Tale](https://static.usenix.org/events/nsdi06/tech/full_papers/schroeder/schroeder_html/index.html), NSDI 2006, plný text | Model příchodů může změnit závěr benchmarku; otevřený, uzavřený i smíšený model podle práce → I1/I3. | Starší systémy a konkrétní workloady. Nezakládá univerzální zákaz closed-loop ani přenos naměřených násobků. |
| **PS03** Maricq et al.: [Taming Performance Variability](https://www.usenix.org/conference/osdi18/presentation/maricq), OSDI 2018, abstrakt/metodický rozsah | Variabilita stroje a běhů omezuje reprodukovatelnost → I4/I5. | Hardwarové měření; neurčuje univerzální počet opakování, délku testu ani práh regrese. |
| **PS04** Hasselbring: [Benchmarking as Empirical Standard in Software Engineering Research](https://arxiv.org/abs/2105.00272v1), EASE 2021, autorský abstrakt | Benchmark má explicitní metodu a rozsah tvrzení → profil experimentu a I6. | Metodický příspěvek, není důkaz účinnosti našich konkrétních kontrol. |

## Odborné metodiky a oficiální dokumentace

| ID a zdroj | Přijaté pravidlo / místo | Omezení |
|---|---|---|
| **PS05** Brendan Gregg: [Benchmarking Checklist](https://www.brendangregg.com/blog/2018-06-30/benchmarking-checklist.html), 2018 | Prověřit co benchmark dělá, limit generátoru, opakování a příčinu výsledku → I2–I5. | Odborná metoda autora, ne kontrolovaný experiment úspěšnosti tohoto skillu. |
| **PS06** Google SRE: [Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) | Latence, provoz, chyby a saturace jako společný kontext → I2/I5. | Monitoring nenahrazuje workload ani akceptační kritéria. |
| **PS07** Grafana: [Open and closed models](https://grafana.com/docs/k6/latest/using-k6/scenarios/concepts/open-vs-closed/) | Délka iterace ovlivňuje přítok uzavřeného modelu → I3. | Ověřit executor a verzi. Open-loop sám neřeší každý způsob vynechávání čekání. |
| **PS08** Grafana: [Dropped iterations](https://grafana.com/docs/k6/latest/using-k6/scenarios/concepts/dropped-iterations/) | Evidovat neodstartovanou práci a rozlišit konfiguraci od zpomalení cíle → I3. | Čtená dokumentace se označovala v2.3.x; není to pin instalovaného k6. |
| **PS09** Grafana: [Thresholds](https://grafana.com/docs/k6/latest/using-k6/thresholds/) | Explicitní metrické podmínky a neúspěšný exit při porušení → automatizace I6. | Ukázkové časy/procenta nepřebíráme. Check sám nenahrazuje threshold. |
| **PS10** PostgreSQL: [Using EXPLAIN](https://www.postgresql.org/docs/18/using-explain.html) | Měřený plán/řádky/buffers; ANALYZE vykonává dotaz → I2. | Čtená verze 18; cílovou DB ověřit, produkční diagnostika má samostatný mandát. |
| **PS11** Playwright: [Request](https://playwright.dev/docs/api/class-request) | Síťové selhání a HTTP error mají odlišné události → I5. | Neověřuje správnost business obsahu; tu přidává test. |
| **PS12** GoogleChrome: [Lighthouse user flows](https://github.com/GoogleChrome/lighthouse/blob/main/docs/user-flows.md), main | Výběr navigation/timespan/snapshot podle uživatelského děje → I2 a volba nástroje. | Režimy nepodporují totožné metriky; žádný příslib univerzálního skóre SPA. |
| **PS23** NIST: [Confidence Limits for the Mean](https://www.itl.nist.gov/div898/handbook/eda/section3/eda352.htm) | Rozlišit variabilitu měření od nejistoty odhadu → I5, doplněno při review. | Interval průměru není návod na interval p99; správná metoda závisí na metrice a nezávislosti vzorků. |

## GitHub: nástroje a podobné skilly

| ID a zdroj | Vyhodnocení pro náš skill |
|---|---|
| **PS13** [grafana/k6](https://github.com/grafana/k6), master | Kandidát pro protokolové zatížení a dostupné browser schopnosti. Převzata volba podle otázky, nikoli povinná instalace či cloud. |
| **PS14** [grafana/skills — k6-perf-test-website](https://github.com/grafana/skills/blob/main/skills/grafana-k6/k6-perf-test-website/SKILL.md), main, čtený SKILL | Užitečné: business workflows, oddělený browser/protokol, monitoring generátoru, cílené typy testů. Nepřebíráme pevné prahy/délky/VU, cloud jako výchozí cestu, duplicitu kódu ani automatický commit HAR. Tvrzení o prerequisitách neověřujeme a nepřenášíme. |
| **PS15** [sitespeedio/sitespeed.io](https://github.com/sitespeedio/sitespeed.io), main | Browser měření a opakované scénáře jako alternativa k existujícímu harnessu. Není doklad kapacity backendu. |
| **PS16** [GoogleChrome/lighthouse-ci](https://github.com/GoogleChrome/lighthouse-ci), main | Automatizace browser auditů/regresí; doplněk k uživatelským tokům. Skóre jedné URL není verdict aplikace. |
| **PS17** [locustio/locust](https://github.com/locustio/locust), master | Alternativa pro tým s Python scénáři; explicitně ověřit model práce a limity generátoru. Samotná dostupnost není důvod přidávat druhý load stack do projektu. |
| **PS18** [giltene/wrk2](https://github.com/giltene/wrk2), master, README | Úzká HTTP diagnostika, měření od plánovaného okamžiku a coordinated omission → I3. Ukázkové výsledky a poměry nejsou univerzální; nenahrazuje browser/business test. |
| **PS19** [rcampos09/performance-testing-skills](https://github.com/rcampos09/performance-testing-skills/blob/main/README.md), main | README ukazuje členění strategie vs. tool-specific postupy. Jen discovery inspirace; vnitřní skilly nebyly detailně ověřeny, nenesou naše technická pravidla. |

## Komunitní signály

Tyto zdroje nejsou odborným důkazem samy o sobě. Jde o podněty, které byly porovnány
s výše uvedenými primárními zdroji. Nehodnotíme nástroj podle hlasů či popularity.

- **PS20 — [What tools are you using for load tests?](https://www.reddit.com/r/ExperiencedDevs/comments/16oiqwf/what_tools_are_you_using_for_load_tests/)**,
  diskuse 2023: výběr podle týmového jazyka a vyčerpání lokálního generátoru před cílem.
  Přijato ověřování generátoru (PS05/PS08/PS14), nikoli automatický přechod do placeného cloudu.
- **PS21 — [Load testing is not about the number of requests — it's about the pattern](https://www.reddit.com/r/QualityAssurance/comments/1tuky1d/load_testing_is_not_about_the_number_of_requests/)**:
  čtený příspěvek a odpovědi k relacím/soak. Autor propaguje vlastní nástroj, jde o
  zkušenostní signál se střetem zájmů. Přijato realistické modelování práce (PS02),
  nikoli tvrzení o převaze jeho nástroje nebo pětihodinový běh jako obecná povinnost.
- **PS22 — [Grafana forum: dropped_iterations_percentage](https://community.grafana.com/t/how-can-i-make-a-dropped-iterations-percentage-metric/108357)**,
  2023: vypsaný poměr v souhrnu nemusí způsobit selhání CI. Přijato ověření návratového
  kódu a skutečné threshold brány (PS09); staré workaroundy/API nepřebíráme.

## Co není nová empirie

Scénáře N+1, tichého ořezu, síťových vln a chyb měřicího nástroje jsou zobecněné
ilustrace pro návrh kontroly. Samy nedokládají výskyt ani četnost vad v konkrétním
systému. Behaviorální test agenta není benchmark aplikace ani populační měření
účinnosti této metody.
