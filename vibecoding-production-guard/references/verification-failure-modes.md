# Chyby detekce — jak selhává audit a verifikace samotná

Defect-classes popisují, jak vznikají chyby *v kódu*. Tahle reference popisuje, jak selhává
*jejich hledání* — ať už audit běží přes subagenty, nebo přes tebe. Všech sedm tříd se
opakovaně objevuje v praxi multi-agentních auditů s adversariální verifikací nálezů.

**Řídící princip — asymetrie falešné jistoty:** chyby detekce nejsou náhodný šum; systematicky
táhnou k falešné jistotě v obou směrech. **Falešné PASS je nebezpečnější než falešný nález** —
falešný nález zabije verifikační krok, ale falešné PASS už nezachytí nikdo. Architektura
detekce proto musí být asymetrická: nálezy prohnat skeptiky, ale „čisto" nikdy nepřijmout
bez důkazu práce a funkčního negativního testu.

**Druhý princip — nejdražší chyba auditu není přehlédnutý nález, ale nález, který není
pravda.** Přehlédnutí najde příští běh. Nepravdivý nález stojí ownera refaktoring, který nebyl
potřeba, a auditora důvěru, kterou už nezíská zpátky. Proto se zapisují i pasti, do kterých
detekce spadla (§6), a každý nález nese úroveň doložení (§7).

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
- Nulový nález s důkazem práce je naopak plnohodnotný výsledek: dostává číslo v registru
  jako „prověřeno bez nálezu" (audit-output.md). Bez toho není vidět rozsah auditu a příští
  běh to dělá znovu.
- Žádné tiché stropy: pokud detekce něco vzorkovala nebo ořezala (top-N souborů, jen část
  schématu), musí to říct — jinak se ořez čte jako pokrytí.

## 2. Plauzibilní, ale nereprodukovatelný nález

**Mechanismus:** Nález má správný žargon, věrohodný file:line, přesvědčivý mechanismus —
a při pokusu o reprodukci se rozpadne. LLM detektory jsou optimalizované na plauzibilitu,
ne na pravdivost; forma nálezu nekoreluje s jeho realitou.

**Typický průběh:** část nálezů auditu umře v adversariální verifikaci (nezávislý skeptik
s instrukcí „vyvrať to"). Bez ní remedy plán obsahuje fixy neexistujících děr — a pálí
důvěru ownera na nálezy, které reálné byly.

**Ilustrativní scénář (oba směry):** hledání textového vzoru ve zdrojácích funkcí místo
posouzení sémantiky dalo nejdřív „ochranu má jen malá část funkcí" (měla ji většina, jen
psanou jinak) a pak „nález je opravený" (nebyl — hledaný výraz seděl v logovací hlášce).

**Protiopatření:**

- Každý nález před remediací **reprodukuj** (dotaz, negativní test, proof-of-concept), nebo
  ho prožeň nezávislým skeptikem, jehož úkolem je vyvracet, ne potvrzovat.
- Skeptik musí být slepý k závěru finderu — dostane tvrzení a evidenci, ne jistotu.
- **Vyhledávání v kódu hledá kandidáty, nikdy nepodává důkaz** — ani o nálezu, ani o opravě.
  Každý kandidát se potvrzuje čtením nebo pokusem.
- **První měření je kandidát, ne nález.** Nález vzniká až druhým, nezávislým způsobem ověření.
- **Verifikuj opravy stejně jako nálezy.** Oprava se dokládá diferenčním testem proti
  ponechanému orákulu: stará implementace vedle nové na stejných vstupech, včetně vstupu za
  hranou, kde se stará láme. „Opraveno" bez diferenčního důkazu je stejně nedoložené jako
  nález bez reprodukce.
- Neopravuj nic jen na základě toho, že „to zní správně". Kritérium je reprodukce.

## 3. Ověření přítomnosti mechanismu místo jeho chování

**Mechanismus:** Detekce kontroluje, že ochrana *existuje* (trigger je v pg_trigger, policy
v pg_policies, validace v kódu) — ne že ochrana *zastaví útočníka*. Pozitivní/strukturální
testy bez negativních dávají zelenou i mrtvé obraně.

**Ilustrativní scénář:** guard trigger napsaný jako SECURITY DEFINER projde celým
strukturálním smokem — a funkční negativní test ukáže, že bypassuje úplně všechny: uvnitř
DEFINER funkce je `current_user` vlastník, ne volající, takže bypass větev „pro systémové
role" pustí každého. Fix = přepnout na INVOKER + trvalý invariant assert na `prosecdef`.

**Druhý scénář — negativní test, který nedoletěl:** test ztraceného zápisu použil umělou
hodnotu, kterou databáze odmítla constraintem. Výsledek „bez ztráty" vypadal jako PASS; po
opravě na platnou hodnotu tentýž scénář selhal. Obrana „držela" jen proto, že k ní útok
vůbec nedorazil.

**Protiopatření:**

- Ke každé obranné vrstvě patří **funkční negativní test**: pokus o akci, kterou má vrstva
  zakázat, provedený identitou, které ji má zakázat. PASS = akce selhala.
- **Negativní test musí doložit, že zásah k obraně doletěl.** Pokud selže dřív (constraint,
  validace, typ, chybějící oprávnění na jiné vrstvě), je PASS falešný. Kontrola: bez testované
  obrany by tentýž zásah prošel (mutační ověření — obranu dočasně vypni v izolovaném
  prostředí, nebo použij vstup, o kterém víš, že jinak projde).
- Strukturální smoke (existence, konfigurace) je nutný, ne postačující. Verdikt „obrana
  drží" smí padnout jen z chování, ne z přítomnosti.
- U pravidla, které „přece platí" (policy, CSS, konfigurační vrstva): změř výsledek; při
  podezření vlož identické pravidlo na konec pořadí a změř, že se změnila právě jedna veličina.
- Zvlášť u SECURITY DEFINER / service účtů / bypass větví: otestuj guard z perspektivy
  běžného přihlášeného útočníka, ne z perspektivy, kterou máš při vývoji po ruce.

## 4. Chybějící ground truth — záměr flagnut jako defekt

**Mechanismus:** Auditor bez přístupu k rozhodovacím záznamům nerozliší **dluh od designu**.
Vědomé rozhodnutí (dočasně plochá trust postura, záměrný rozpor datový klíč ≠ UI label)
flagne jako HIGH nález; v horším případě ho remediace „opraví" — a vyrobí regresi třídy
„nevyslovený invariant".

**Ilustrativní scénář:** always-true read policies jako vědomé pre-pilot rozhodnutí zapsané
v ADR; stabilní datový klíč pod přejmenovaným UI labelem taky. Auditor bez těchto ADR obojí
flagne jako defekt a „opraví" k regresi. Podobně: tabulky bez FK, které používají ratifikovaný
polymorfní vzor; číselníky, jejichž sloučení by zničilo referenční integritu.

**Protiopatření:**

- Audit prompty **groundovat v ADR + dokumentu invariantů** — decision recordy jsou strojově
  čitelná paměť záměru pro budoucí auditory; přesně proto se vedou.
- Nález proti dokumentovanému rozhodnutí není defekt, ale podnět: „platí ADR-00NN ještě?"
  Eskaluje se jako **rozhodnutí** (audit-output.md) s odkazem na zdroj a s popisem ceny,
  kterou nese — ne jako položka remedy.
- Subagenti startují bez kontextu session — každý audit prompt nese relevantní ADR/invarianty
  explicitně, nebo je jeho verdikt o záměrných místech nedůvěryhodný.
- Kapitola „co vypadá jako nález a není" patří do každého reportu; jinak se tentýž „nález"
  objeví při příštím auditu znovu.

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

## 6. Slepá skvrna měřicího nástroje

**Mechanismus:** Audit je měřicí přístroj a má vlastní chybu. Sonda, harness nebo selektor
pokrývá jen část jevu, a *absence signálu* se pak čte jako *absence selhání*. Na rozdíl od
§1 tu není prázdný výstup agenta, ale neúplný vstup — nástroj nevidí, co má měřit, a neví to.

**Zaznamenané pasti (každá vedla k chybnému závěru, než byla odhalena):**

- Sonda sledovala jen kanál **odpovědí**. Požadavek odmítnutý bránou bez odpovědi (414 bez
  CORS hlaviček) žádnou odpověď nevyvolal — závěr „žádná plocha neselhává", ve skutečnosti
  selhávaly. Stejný důvod, proč tahle třída uniká i běžnému monitoringu.
- Měření začínalo na úvodní ploše, která přesměrovává; její selhání se přičetlo každé měřené
  ploše — „problém trefuje všechno", trefoval tři.
- Testovací nástroj restartoval pracovní proces po každém pádu testu; výsledky agregované
  v paměti přežily jen z posledního testu — neúplná sada vypadající jako úplná.
- Odvozený ukazatel („počet vykreslených řádků" přes selektor) nepokrýval všechny typy
  zobrazení — podezření na prázdnou plochu, která ve skutečnosti vykreslila tisíc řádků.
- Profil generovaných dat kolidoval s obchodním pravidlem systému; část dat se tiše
  nevygenerovala a všechny metriky nad nimi byly zkreslené.

**Protiopatření:**

- **Oba kanály.** Sleduj i selhané požadavky (bez odpovědi, přerušené, timeout), ne jen
  odpovědi. Totéž platí pro produkční monitoring.
- **Měř až po ustálení, přímo na cíli.** Naviguj rovnou na měřenou plochu; mezivýsledek
  se nikdy necituje jako závěr.
- **Zapisuj průběžně na disk**, nikdy neagreguj v paměti procesu, který se může restartovat.
- **Závěry stav na primárních veličinách** (počty požadavků, objemy dat, snímky obrazovky),
  ne na odvozených ukazatelích. Odvozený ukazatel se před použitím kalibruje proti primárnímu.
- **Po vygenerování dat porovnej skutečné počty s očekávanými** a odchylku popiš v reportu,
  nepřecházej ji.
- **Odhad je odhad**: v reportu ho označ a při pozdějším přesném měření dohledej, jak vyšel.
- **Sbírej surová data štědře, vyhodnocuj cíleně** a surová data nemaž: nálezy s nejvyšší
  závažností často vypadnou z dat sbíraných k jiné otázce.

## 7. Důkazní žebřík — co znamená „doloženo"

Každý nález i každá oprava se zařadí do jedné ze tří úrovní. Bez zařazení není nález
doložený, jen tvrzený.

| Úroveň | Co znamená | Kdy stačí |
|---|---|---|
| **Změřeno** | Existuje číslo z opakovatelného měření a je uvedeno, jak vzniklo. | Většina nálezů. Výchozí úroveň. |
| **Reprodukováno** | Existuje postup, kterým se chování vyvolá na vyžádání — u vad se stopou před opravou i po ní. | **Povinné** u všeho s bezpečnostním nebo finančním dopadem. |
| **Doloženo experimentem** | Příčina je izolována zásahem, po kterém se změní právě jedna veličina. | Tam, kde hrozí záměna příčiny s korelací (tři pravděpodobně vypadající příčiny, jedna skutečná). |

Nejvýš stojí experiment a vyplácí se právě u „pravidlo existuje a neplatí": teprve vložení
identického pravidla na konec pořadí a změření, že se změnila jediná veličina, oddělí
skutečnou příčinu od pravděpodobných.

## Provozní poznámka — degradace detektoru za běhu

Kapacita detektoru se může změnit uprostřed běhu, aniž to orchestrátor vidí (např. tichý
model-switch za běhu session). Kritické verdikty (verifikace, stop-ship) proto neopírat
o předpoklad konstantní kvality jednoho běhu — nezávislé ověření klíčových závěrů je i pojistka
proti degradaci detektoru, ne jen proti jeho omylnosti.
