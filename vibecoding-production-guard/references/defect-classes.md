# Čtyři defektní optiky

První tři optiky jsou praktická interpretace memory/objective/knowledge defects ze studie
[Deng et al., v4](https://arxiv.org/abs/2606.23130v4). Čtvrtá spojuje provozní zkušenost
s výzkumem ošetření chyb a OWASP A10:2025. Nejde o tvrzení, že tyto čtyři kategorie
pokrývají všechny vady nebo jsou stejně časté ve všech aplikacích.

Projektové příklady níže jsou anonymizované zkušenostní vzory; jejich počty nejsou
reprezentativní měření. Rozsah veřejných studií: [zdroje](sources.md).

## 1. Neúplné šíření změny

Agent změní viditelné místo a ponechá staré pravidlo v dalších konzumentech.

Příklady: nové pole chybí ve whitelistu přechodového RPC; nová tabulka má policy bez
potřebného grantu; guard přibude jen do jedné z kopií writeru; nová migrace zneplatní
navazující testy nebo metadata vydání; export používá jiný filtr než obrazovka.

Hledej všechna místa se stejným pravidlem a sestav malou matici kopie × vlastnost.
Grep pomáhá hledat kandidáty, průchod správností musí ověřit tok.
Opakování stejného mismatch může ukazovat na chybějící kanonický kontrakt; navrhni
constraint, sdílený zdroj nebo cílený assert. Rozsáhlý refaktor není automatická oprava.

## 2. Fix oslabením obrany

Symptom zmizí, protože oprava odstraní ochranu nebo detektor.

Příklady: privilegovaný klient místo správného oprávnění; vypnutí validace;
nekontrolované rozšíření grantů; odstranění testu; catch vracející úspěch;
zvednutí limitu místo korektní práce s objemem; obejití owner gate.

Ptej se, jaká ochrana při změně zmizela a kdo rozhodl, že ji lze postrádat.
Povolení veřejného čtení může být legitimní kontrakt, nikoli vada podle textového vzoru.
Stejně tak `using (true)`, wildcard CORS nebo vypnuté lint pravidlo vyžadují kontext.
Vyhodnoť expozici a invariant; automaticky je nepřepisuj.

## 3. Nevyslovený invariant

Pravidlo není v zadání ani udržované dokumentaci, ačkoli na něm systém závisí.

Příklady: oprávnění chrání server; klíč enumu může být stabilní pod novým UI labelem;
technické datum importu není doménové datum pro KPI; refresh testovacích dat zneplatní
dřívější důkazy; evidenční migrace nedokazuje skutečnou podobu funkce.

Ptej se: kdo smí provést tuto operaci, v jakém stavu a nad čím?
Které pravidlo jsi zachoval pouze díky kontextu sezení?
Zaznamenej je jako kandidáta do místního registru invariantů.
Chybějící rozhodovací kritérium není oprávnění auditora vymyslet nové pravidlo.

## 4. Tiché selhání

Systém vrátí výsledek, který vypadá věrohodně, ale ztratil část významu nebo efektu.

Příklady:
- timeout či 403 se převede na `[]` a UI hlásí „žádné záznamy“;
- limit API ořízne report nebo export, aniž je neúplnost vidět;
- klient po nepovedeném zápisu ponechá optimistický úspěch;
- dva uživatelé přepíší změny bez konfliktu;
- vedlejší služba selže a hlavní transakce nepravdivě hlásí úplný úspěch.

Sestav matici důležitá plocha × porucha (timeout, odmítnutí, částečná odpověď,
výpadek závislosti). Testuj nad izolovaným reprezentativním prostředím.
Zvětši data přes reálnou hranu infrastruktury; konkrétní limit zjisti z nasazené verze.
U stránkování použij stabilní pořadí a ověř úplnost podle kontraktu.
`count=exact` může být drahé, odhad počtu není důkaz úplnosti exportu.
U dvou sezení ověř detekci konfliktu; zámek končící s jedním API requestem
nechrání navazující request.

Tyto optiky se doplňují s checklistem zranitelností: mechanismus vysvětluje vznik chyby,
bezpečnostní kontrola ověřuje konkrétní hranici.
