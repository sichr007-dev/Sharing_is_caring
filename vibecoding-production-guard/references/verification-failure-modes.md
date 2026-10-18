# Jak může selhat samotná kontrola

Heuristiky jsou založené na anonymizovaných projektových zkušenostech a veřejných
metodikách; nevyvozuj z nich statistiky o všech auditorech. Evidence:
[zdroje](sources.md). Důkaz úspěchu i vady má vždy omezený rozsah.

## 1. Prázdný výstup vydávaný za čistý výsledek

Schema-validní výstup s findings=[] může znamenat, že se nic nekontrolovalo.
Vyžaduj důkaz práce: prošlé soubory/toky, role, příkazy, výsledky, vynechané části.
Výpadek, timeout nástroje či useknutý výstup je UNKNOWN.
Pravidlo platí pro vlastní kontrolu i případnou autorizovanou delegaci.

## 2. Věrohodný nález bez dosažitelné cesty

Nález musí popsat předpoklady útočníka, vstup, tok, již existující ochrany a dopad.
Samotné slovo „service_role“, grep na true policy či chybějící middleware není reprodukce.
Před opravou zkus tvrzení vyvrátit. Ověř konkrétní kontext, ne pouze podobnost s CWE.
U vážného dopadu může být dostatečný průkazný zdrojový důkaz, když by exploit byl
nebezpečný; označ úroveň a zbývající nejistotu, nesimuluj runtime reprodukci.

Opravu ověř stejným orákulem na baseline a opravené verzi, pokud jsou dostupné.
Zahrň legitimní operaci a související vstupy; samotné přejmenování vzoru nebo pokles
počtu nálezů scanneru nedokazuje bezpečné chování. Pokud baseline chybí, řekni to.

## 3. Negativní test nedoletěl k testované obraně

Existující trigger/policy/validace nedokazuje účinnost.
Pokus zakázanou identitou musí mít validní předpoklady a dosáhnout správné hranice.
401 z expirovaného tokenu neověřuje oprávnění přihlášeného uživatele k cizímu záznamu;
neexistující záznam, import error či nedostupný server také nic nedokazují.

Doplň kontrolní povolenou akci se stejným tokem/fixture nebo průkaznou trasu a bod
odmítnutí. Ověř i výsledek v datech, ne pouze nenulový exit code.
U SECURITY DEFINER, owner a BYPASSRLS rolí zkontroluj skutečnou identitu uvnitř operace.
Testuj přes produkčně odpovídající trust boundary, ne přes privilegované vývojové spojení.

## 4. Dokumentovaný záměr označený za chybu

Přečti ADR, spec, výjimky a invarianty před klasifikací.
Doložená oprávnění mohou být záměrně širší než obvyklý výchozí profil.
Ověř jejich rozsah a předpoklady před klasifikací vady. Je-li doloženo, že se změnila
expozice nebo předpoklady, zapiš samostatnou otázku platnosti rozhodnutí; samotný
odlišný výchozí profil není důvodem k automatické opravě.
Nesoulad zdrojů pravdy bez možnosti určit autoritu řeš místním eskalačním postupem.

## 5. Slepota k druhotným efektům vlastní opravy

Prověř další kopie pravidla a evidence, které diff zneplatnil: testovací konfiguraci,
manifest vydání, generované typy, dokumentaci nebo artefakt.
Mechanickou pojistku skutečně spusť. „Přidal jsem assert“ není důkaz, že běží a chrání.
Oprava může zavést novou vadu; regresní a variantové kontroly ověřují i legitimní chování.

## 6. Záměna historických dat za vadu — nebo za bezcenné reziduum

Ověř původ dat, starý i dnešní writer, podporovaný migrační kontrakt a reprezentativnost.
Problém jen na syntetickém pozůstatku může patřit do úklidu testovacích dat.
Problém na legitimních historických/importovaných produkčních datech zůstává reálný,
i když dnešní čistý seed projde. Fresh DB není jediný zdroj pravdy.
Rozlišuj vadu kódu, vadu migrace/data kontraktu a jednorázový datový úklid.

## 7. Slepá skvrna měřicího nástroje

U každé metriky zapiš definici, rozsah, verzi nástroje a omezení.
Textový vzor není parsování, statická analýza není runtime exploit, počet HTTP 200
není počet úspěšných business operací. Zkrácený seznam není audit celého repa.
Neúspěšný start testu nepočítej jako bezpečnostní odmítnutí.
Limity z jiného projektu nepřenášej jako univerzální práh.
Audit tool může odesílat kód či metadata ven; ověř jeho chování před spuštěním.

## 8. Přenos důkazu na jinou revizi nebo prostředí

Zaznamenej commit a dirty diff/artefakt, konfiguraci, verzi DB, roli, data a čas.
Rozdíly mezi shadow a produkcí uveď; „stejný kód“ neznamená stejná oprávnění,
proxy hlavičky ani secrets. Dřívější PASS není nadčasový certifikát.
Po změně relevantních předpokladů opakuj dotčené kontroly.

## Úrovně důkazu a výsledek

- **Zdrojově doloženo:** přesný tok v kódu/konfiguraci; runtime se neprovedl.
- **Změřeno:** skutečné pozorování s nástrojem, rozsahem a prostředím.
- **Reprodukováno:** konkrétní operace dosáhla hranice a ukázala tvrzený dopad.
- **Diferenciálně ověřeno:** stejný relevantní test odlišuje baseline od opravy,
  legitimní kontrolní chování zůstává zachované.

Nejde o žebříček, kde je každá další úroveň nutná pro každou položku. Konfigurační kontrola
může mít dostatečný zdrojový důkaz; tvrzení „ochrana odmítne požadavek“ vyžaduje chování.

Výsledek kontroly: PASS / FAIL / UNKNOWN / N/A. Doložená vada a její stav, důkazní
úroveň a schválená výjimka jsou různé osy; neslučuj je do jednoho „vyřešeno“.
