# Analytický gate a review diffu

Použij před změnou na gate povrchu a při jejím ověření. Délku zápisu přizpůsob riziku;
stručný zápis nesmí přeskočit důvěryhodnostní hranici.

## Před změnou

1. **Mandát a kotva:** oprava, audit, nebo vydání? Jaká revize a prostředí?
   Co je povolené měnit a kam smí odcházet data?
2. **Příčina:** dolož symptom a jeho mechanismus logem, dotazem nebo reprodukcí.
   Odmítnutí RLS/validací může být správné chování. Do určení příčiny nezmírňuj ochranu.
3. **Kontrakt:** přečti místní specy, ADR, invarianty a schválené výjimky.
   Rozpor bez určitelného zdroje pravdy eskaluj podle projektu.
4. **Tok a hranice:** klient → API/RPC → grant/policy → data → externí služba.
   Urči, kdo ovládá vstup, kdo nese identitu a kde se vynucuje oprávnění.
5. **Invarianty a kopie:** které vlastnosti se nesmějí změnit a kde všude jsou implementované?
   Zahrň whitelisty, typy, kopie zápisových cest, piny, manifesty a evidenci testů.
6. **Oslabení:** nevyřeší návrh problém přidáním privilegovaného účtu, odstraněním validace,
   vypnutím TLS, rozšířením grantů nebo obejitím schvalovacího kroku?
   CORS není autorizace; jeho chování posuzuj podle typu dat a autentizace.
7. **Selhání a limity:** timeout, 403, částečná odpověď, chybějící pole, výpadek služby,
   překročená velikost, rozpad stránky dat. Co uvidí uživatel a co provoz?
   Prázdno, chyba, neúplná a zastaralá data mají mít rozlišitelnou sémantiku.
8. **Souběh:** dvojklik, retry po timeoutu s neznámým výsledkem, souběžný zápis,
   duplicitní či přeházený webhook. Urči atomické hranice a trvalou deduplikaci tam,
   kde by opakování poškodilo data nebo provedlo placenou akci znovu.
9. **Dodání a obnova:** starý klient / nová DB, cache a dlouho otevřený tab,
   nová aplikace / staré API, rollback se změněnými daty.
   DB-first je bezpečné jen s kompatibilní přípravnou změnou; destruktivní odstranění
   přichází až po odchodu starých konzumentů. Nevnucuj jeden delivery model všem projektům.
10. **Ověření a diff:** navrhni důkaz chování, legitimní kontrolní případ a negativní test
    pod skutečnou rolí. U chyby preferuj porovnání baseline/oprava.
    Implementuj minimální soudržnou změnu; nesouvisející refaktory odděl.

11. **Výkonový dopad:** změní se počet requestů, objem/rozložení dat, souběh,
    cache, render, fronta nebo resource limity? U auditu projektu/vydání vždy použij
    [I1–I6](performance-scalability.md); u změny dotčené kontroly. Chybějící výkonový
    požadavek nebrání objevení a změření provozního problému.

U datového symptomu ověř původ dat a dnešní zapisovatel. Historická data mohou být legitimní
produkční data, která musí změna podporovat; nerozhoduj o jejich zahození podle stáří.

## Review po implementaci

- Přístup: nová cesta nemůže obejít serverovou autorizaci přes jiné ID, tenant, roli,
  hromadný endpoint, view, export, storage objekt nebo privilegované RPC.
- Vstupy: runtime validace na hranici, explicitně povolená měnitelná pole,
  parametrizované dotazy a kontextově správné zacházení s HTML, URL a cestami.
- Integrita: transakce, constrainty a stavové přechody chrání invariant i bez UI.
  Síťový timeout neznamená, že zápis neproběhl; retry nesmí nechtěně zdvojit efekt.
- Selhání: žádný catch převádějící neúspěch na úspěch, nepozorovaný ořez,
  optimistické UI bez obnovy stavu nebo nekonečný retry.
- Secrets a artefakt: zkontroluj skutečný výstup buildu a veřejné endpointy.
  Rozlišuj veřejné identifikátory od klíčů s privilegii.
- Supply chain: změny lockfilu, instalačních skriptů, CI oprávnění a agentních konfigurací
  mají vysvětlení. Lockfile zamyká i škodlivou verzi; není důkazem důvěryhodnosti.
- Propagace: prověř další implementace stejného pravidla a vedlejší evidence
  (testovací konfigurace, manifest vydání, přehled provedených kontrol).
- Důkaz opravy: test opravuje chování, není pouze upraven tak, aby přestal hlásit chybu.
  Zaznamenej použitý commit/diff, role, data, příkazy a výsledek bez citlivých hodnot.

## Rozsah a rozhodnutí

Výstup analýzy má umožnit rozhodnutí: příčina → ohrožené pravidlo → změna → důkaz.
Nové doménové rozhodnutí předej místnímu spec/ADR procesu. Audit sám neuděluje mandát
k nové funkci, refaktoringu, deployi nebo změně trust modelu.
