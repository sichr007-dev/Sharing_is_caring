# Chyby detekce — jak selhává audit a verifikace samotná

Defect-classes popisují, jak vznikají chyby *v kódu*. Tahle reference popisuje, jak selhává
*jejich hledání* — ať už audit běží přes subagenty, nebo přes tebe. Všech pět tříd se
opakovaně objevuje v praxi multi-agentních auditů s adversariální verifikací nálezů.

**Řídící princip — asymetrie falešné jistoty:** chyby detekce nejsou náhodný šum; systematicky
táhnou k falešné jistotě v obou směrech. **Falešné PASS je nebezpečnější než falešný nález** —
falešný nález zabije verifikační krok, ale falešné PASS už nezachytí nikdo. Architektura
detekce proto musí být asymetrická: nálezy prohnat skeptiky, ale „čisto" nikdy nepřijmout
bez důkazu práce a funkčního negativního testu.

## 1. Degenerovaný výstup, který projde validací

**Mechanismus:** Agent vrátí schema-validní, ale obsahově prázdný výsledek. Nejzrádnější
vlastnost: **prázdný nález je nerozlišitelný od čistého nálezu** — dimenze auditu „projde",
protože findings=0, i když se reálně nic nekontrolovalo.

**Typický projev:** dimenze auditu vrátí `{"dimension":"test","summary":"test",
"findings":[]}` — placeholder, který v mechanickém zpracování projde jako „bez problémů".
Odhalí se jen čtením výstupu očima; dimenze se pak musí přeběhnout samostatně.

**Protiopatření:**

- Strukturální validita ≠ sémantická validita. Schema si vynuť, ale výstup vždy čti.
- „Nic jsem nenašel" musí nést **důkaz práce**: co konkrétně bylo prošlé (soubory, tabulky,
  dotazy). Bez toho je nulový výsledek nulová informace, ne zelená.
- Žádné tiché stropy: pokud detekce něco vzorkovala nebo ořezala (top-N souborů, jen část
  schématu), musí to říct — jinak se ořez čte jako pokrytí.

## 2. Plauzibilní, ale nereprodukovatelný nález

**Mechanismus:** Nález má správný žargon, věrohodný file:line, přesvědčivý mechanismus —
a při pokusu o reprodukci se rozpadne. LLM detektory jsou optimalizované na plauzibilitu,
ne na pravdivost; forma nálezu nekoreluje s jeho realitou.

**Typický průběh:** část nálezů auditu umře v adversariální verifikaci (nezávislý skeptik
s instrukcí „vyvrať to"). Bez ní remedy plán obsahuje fixy neexistujících děr — a pálí
důvěru ownera na nálezy, které reálné byly.

**Protiopatření:**

- Každý nález před remediací **reprodukuj** (dotaz, negativní test, proof-of-concept), nebo
  ho prožeň nezávislým skeptikem, jehož úkolem je vyvracet, ne potvrzovat.
- Skeptik musí být slepý k závěru finderu — dostane tvrzení a evidenci, ne jistotu.
- Neopravuj nic jen na základě toho, že „to zní správně". Kritérium je reprodukce.

## 3. Ověření přítomnosti mechanismu místo jeho chování

**Mechanismus:** Detekce kontroluje, že ochrana *existuje* (trigger je v pg_trigger, policy
v pg_policies, validace v kódu) — ne že ochrana *zastaví útočníka*. Pozitivní/strukturální
testy bez negativních dávají zelenou i mrtvé obraně.

**Ilustrativní scénář:** guard trigger napsaný jako SECURITY DEFINER projde celým
strukturálním smokem — a funkční negativní test ukáže, že bypassuje úplně všechny: uvnitř
DEFINER funkce je `current_user` vlastník, ne volající, takže bypass větev „pro systémové
role" pustí každého. Fix = přepnout na INVOKER + trvalý invariant assert na `prosecdef`.

**Protiopatření:**

- Ke každé obranné vrstvě patří **funkční negativní test**: pokus o akci, kterou má vrstva
  zakázat, provedený identitou, které ji má zakázat. PASS = akce selhala.
- Strukturální smoke (existence, konfigurace) je nutný, ne postačující. Verdikt „obrana
  drží" smí padnout jen z chování, ne z přítomnosti.
- Zvlášť u SECURITY DEFINER / service účtů / bypass větví: otestuj guard z perspektivy
  běžného přihlášeného útočníka, ne z perspektivy, kterou máš při vývoji po ruce.

## 4. Chybějící ground truth — záměr flagnut jako defekt

**Mechanismus:** Auditor bez přístupu k rozhodovacím záznamům nerozliší **dluh od designu**.
Vědomé rozhodnutí (dočasně plochá trust postura, záměrný rozpor datový klíč ≠ UI label)
flagne jako HIGH nález; v horším případě ho remediace „opraví" — a vyrobí regresi třídy
„nevyslovený invariant".

**Ilustrativní scénář:** always-true read policies jako vědomé pre-pilot rozhodnutí zapsané
v ADR; stabilní datový klíč pod přejmenovaným UI labelem taky. Auditor bez těchto ADR obojí
flagne jako defekt a „opraví" k regresi.

**Protiopatření:**

- Audit prompty **groundovat v ADR + dokumentu invariantů** — decision recordy jsou strojově
  čitelná paměť záměru pro budoucí auditory; přesně proto se vedou.
- Nález proti dokumentovanému rozhodnutí není defekt, ale podnět: „platí ADR-00NN ještě?"
  Eskaluje se jako otázka ownerovi, ne jako položka remedy.
- Subagenti startují bez kontextu session — každý audit prompt nese relevantní ADR/invarianty
  explicitně, nebo je jeho verdikt o záměrných místech nedůvěryhodný.

## 5. Slepota k druhotným efektům vlastní změny

**Mechanismus:** Agent (ani člověk) není spolehlivý detektor vlastních chyb — self-review
vidí to, na co autor myslel. Druhotné efekty (počítadlo, pin, manifest, evidence, kterou
změna zneplatnila) unikají přesně proto, že nejsou v diffu, jen diffem zastarají.

**Ilustrativní scénář:** zavedeš replay pin (assert počtu migrací), pak přidáš další migrace
bez bumpu — replay odmítne běžet. Přesně třída „neúplné šíření změny", kterou celý den lovíš
u cizího kódu, vyrobená vlastníma rukama tentýž den. Odhalí ji mechanický gate při
**skutečném spuštění**, ne sebekontrola.

**Protiopatření:**

- Spolehlivé jsou **levné, hloupé, deterministické pojistky, které se skutečně spouštějí**
  (pin, count assert, hash manifest, CI check) — ne předsevzetí „příště si vzpomenu".
- Pojistku po zavedení ihned **spusť naostro** — pojistka, která nikdy neběžela, je dekorace.
  (První ostrý běh navíc často odhalí reálnou divergenci — třeba že migrace nereprodukují
  hardened stav živé DB, protože platformní default dává nové tabulce plné granty.)
- Po každé vlastní dávce: „které počítadlo / pin / evidenci / baseline jsem právě zneplatnil?"
  Odpověď „žádné" chce vteřinu grep, ne pocit.

## Provozní poznámka — degradace detektoru za běhu

Kapacita detektoru se může změnit uprostřed běhu, aniž to orchestrátor vidí (např. tichý
model-switch za běhu session). Kritické verdikty (verifikace, stop-ship) proto neopírat
o předpoklad konstantní kvality jednoho běhu — nezávislé ověření klíčových závěrů je i pojistka
proti degradaci detektoru, ne jen proti jeho omylnosti.
