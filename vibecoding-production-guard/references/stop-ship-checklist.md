# Gate vydání — kontroly a důkazy

Pro konkrétní revizi, artefakt, prostředí a fázi vyber relevantní kontroly.
Každá má ID, výsledek PASS / FAIL / UNKNOWN / N/A, důkaz, roli a datum.
N/A zdůvodni; chybějící přístup, nespustitelný test nebo nedostupný skener znamená UNKNOWN.

**🔴** Relevantní FAIL nebo UNKNOWN blokuje příslušnou fázi.
**🟡** Nesplnění vyžaduje opravu nebo existující přijatelné rozhodnutí podle pravidel projektu.
Neověřenou 🟡 kontrolu uveď výslovně; neuzavřené riziko bez rozhodnutí blokuje.
Výjimka nese vlastníka, rozsah, důvod, platnost nebo podmínku přezkumu a odkaz.
Neznámé riziko lze přijmout jen výslovně jako neověřené, pokud to místní kontrakt dovoluje;
nesmí se změnit na PASS. Schválené projektové výjimky znovu neotvírej bez doložené změny předpokladů.

## A. Secrets a publikovaný obsah

- **A1 🔴** Žádný privilegovaný klíč, heslo nebo osobní/citlivý dump v publikovaném
  artefaktu, dostupném endpointu, zdrojovém repu či jeho historii nebo veřejném logu.
  Důkaz: kontrola změny, buildu a relevantní historie; citlivé hodnoty nezapisuj do reportu.
  Aktivní únik vyžaduje zastavit příslušné vydání a eskalovat revokaci/rotaci podle oprávnění.
- **A2 🔴** Rozlišené veřejné identifikátory a secrets; env proměnné v klientu posouzené
  podle skutečné konfigurace bundleru. Supabase publishable / legacy anon klíč je veřejný;
  secret / service_role je privilegovaný. Veřejný klíč není důkazem správné autorizace.
- **A3 🟡** Zkontrolovaný obsah skutečně distribuovaného balíčku/buildu:
  žádné neplánované .env, .git, zálohy, testovací data nebo interní konfigurace.
  Source mapa je nález podle svého obsahu a zamýšlené expozice; sama její existence
  není únik tajemství. Confidential zdrojáky/secrets v mapě posuzuj podle A1.

## B. Identita a autorizace

- **B1 🔴** Přístupová matice identita × operace × objekt/tenant odpovídá schválenému
  modelu a je vynucená na serveru/DB. Změna ID, role či pole requestu ji neobejde.
  Důkaz zahrnuje povolený i zakázaný případ pod reálnou identitou.
- **B2 🔴** Anonymní a privilegované vstupní body odpovídají záměru.
  Prověř API/RPC, exporty, storage, views a alternativní cesty ke stejným datům.
  V DB vystavené přímo klientu ověř grants + RLS/policies na exponovaných tabulkách;
  nevyžaduj RLS pro každý backend každého stacku.
- **B3 🟡** Nové tabulky mají zamýšlené explicitní granty a privilege asserty.
  Prověř default privileges, function EXECUTE, SECURITY DEFINER/search_path a vlastníka.
  U views ověř efektivní identitu; RLS základní tabulky automaticky nedokazuje ochranu view.
  Přesná syntaxe/omezení závisí na verzi a místních schválených výjimkách.
- **B4 🔴**, když se mění auth: ověř podpis a povinné claims tokenu podle použitého
  protokolu, expiraci a chování odhlášení/revokace podle kontraktu; hesla přes udržovaný
  auth systém, reset/pozvánky bez bypassu a neomezeného opakovaného použití.
  Nepředpokládej, že expirovaný refresh token ihned zneplatní každý již vydaný access token.
- **B5 🟡** Session cookies a CSRF ochrana odpovídají mechanismu autentizace.
  U browserem automaticky přikládaných credentials ověř cross-site mutaci.
  CORS nepovažuj za autorizaci ani sám o sobě za CSRF ochranu.
  Wildcard na veřejném assetu není automaticky chyba; cookie požadavky nevynucuj
  na aplikaci bez cookie session.

## C. Integrita, souběh a externí efekty

- **C1 🔴** Business invarianty přežijí přímé volání API bez UI; constrainty a serverové
  operace chrání stavové přechody, povolená pole a vícekrokové zápisy.
- **C2 🟡** Dvojklik, retry po neznámém výsledku a dva zapisovatelé nevedou k nechtěné
  duplicitě či ztracenému zápisu. Důkaz: souběžný test, constraint, CAS nebo jiný
  doložený mechanismus; žádná povinná univerzální strategie.
- **C3 🔴**, jsou-li webhooky/placené nebo nevratné efekty: ověření původu/podpisu podle
  poskytovatele, replay/freshness pravidel a trvalá idempotence.
  U podpisů závislých na raw payload ověř přesné přijaté bytes.
  Duplicita a změna pořadí událostí nesmějí zdvojit efekt nebo vrátit stav zpět.
  Síťovou službu nelze automaticky zahrnout do DB transakce; ověř retry/outbox/kompenzaci
  podle kontraktu, ne pouze existenci BEGIN/COMMIT.

## D. Vstupy, selhání a limity

- **D1 🔴** Runtime validace na hranicích, explicitní povolená pole,
  parametrizace SQL a kontextově správné kódování/sanitizace výstupu.
  Typy ani UI validace nechrání samostatné API.
- **D2 🔴**, jsou-li příslušné vstupy: URL fetch, upload, archivy a cesty
  neumožní SSRF, path traversal, nepovolené schéma, přístup k metadatům ani únik dat.
  Ověř limity velikosti/dekomprese a relevantní redirect/DNS chování.
- **D3 🔴** Selhání důležité čtecí či zápisové cesty je rozlišitelné od prázdna/úspěchu.
  Výpadek nezpůsobí fail-open, částečný zápis bez signalizace nebo nekonečný retry.
  Podrobnosti pro diagnostiku se logují bezpečně, uživatel nedostává secrets/stack trace.
- **D4 🟡** Stránkování, exporty a agregace zachovávají úplnost podle kontraktu.
  Otestuj překročení reálného stropu řádků a velikosti filtru/URL.
  Ořez ani odhad počtu nejsou úplný report; nepotřebné exact count nevynucuj všude.
- **D5 🟡** Zneužitelné či nákladné operace mají přiměřené limity:
  velikost dávky, čas, počet pokusů, fan-out, frontu, spotřebu externího API a náklady.
  Zvlášť login/reset, veřejné tokeny a automatizovatelné obchodní operace.
  Zátěžové a fault testy prováděj izolovaně, ne proti živým uživatelům.

## E. Migrace a dodání

- **E1 🔴 — kandidát:** schéma má verzovanou migraci, test/smoke a aktualizované
  mechanické pojistky. Ověř migrační řetězec a související testy v izolovaném prostředí
  podle místního postupu.
- **E2 🔴 — aktivace:** ověř delivery model, přesný commit a celý přenášený diff
  proti skutečné cílové větvi; nehádej, že deployuje main.
  Závislá DB/API schopnost musí existovat dříve, než ji začne používat klient.
  Její existenci/kontrakt dolož přímo cílovým systémem, nikoli jen migrační historií.
- **E3 🔴 — přechod:** starý klient, cache a dlouhé relace fungují během kompatibilní
  přípravné změny; destruktivní krok má explicitní plán přechodu.
  Záloha sama nedokazuje bezvýpadkovost ani bezztrátový rollback.
- **E4 🟡** Známá cesta návratu/roll-forward, bod nevratnosti a vlastník.
  U obnovy po nových zápisech popiš ztrátu dat nebo jejich slučování; neslibuj
  „vrátíme build“ jako úplnou obnovu databáze.
- **E5 🔴 — důkazní kontinuita:** důkazy se vztahují k dodávanému commitu/dirty diffu,
  buildu, DB stavu, konfiguraci a datům. Změny zneplatňující kontrolu vyvolají její opakování.
  Kandidátní PASS neznamená „nasazeno“; post-deploy ověření má samostatný záznam.

## F. Supply chain, CI a provoz

- **F1 🟡** Reprodukovatelná instalace ze zkontrolovaného lockfilu, známé názvy/registry
  a účel nových balíčků. Zkontroluj instalační skripty i změny agentních hooks/MCP/CI.
  Samotné stáří nebo download count balíčku nedokazuje bezpečnost.
- **F2 🟡** SCA/advisory kontrola odpovídá přesným verzím a prostředí.
  Rozlišuj nález scanneru, dosažitelnost a dopad; neaktualizuj vše naslepo ani
  nepoužívej force fix bez posouzení. Doložená kritická exploatovatelná cesta blokuje
  podle dotčené 🔴 ochrany, i kdyby SCA report měl jiné souhrnné skóre.
- **F3 🟡** Build/CI má minimální oprávnění a secrets nejsou dostupné nedůvěryhodným PR
  nebo instalačním skriptům. Třetí strany jsou připnuté dle politiky projektu;
  u GitHub Actions preferuj ověřený plný commit SHA. Omezení lifecycle skriptů nastav
  podle skutečné verze package manageru; přepínač nenahrazuje izolaci.
- **F4 🟡** Zkontrolovaná skutečná expozice TLS, cookies, bezpečnostních hlaviček,
  storage a debug rozhraní v cílovém prostředí; localhost nedokazuje CDN/reverse proxy.
  Zvol kontroly podle stacku a obsahu, nevynucuj plošně všechny hlavičky každému assetu.
- **F5 🟡** Kritické chyby a operace mají bezpečnou auditní stopu a vlastníka reakce.
  Známe rozsah/retenci záloh, poslední doloženou obnovu a přijatelné RPO/RTO
  podle potřeb projektu; nevyzkoušená obnova se neoznačuje PASS.
- **F6 🟡** Shadow/audit prostředí má omezenou síťovou expozici a bezpečná testovací data.
  Prověř mapované porty a efektivní dostupnost po startu/restartu a ukončení.
  Reporty/harness bez mandátu nezveřejňuj ani neposílej do externí služby.

## G. Ověření z perspektivy uživatele

Podle změny proveď v izolovaném prostředí: druhou identitu/roli, odhlášený přístup,
přímé API bez UI, přerušené načítání, dva zapisovatele a obnovu po chybě.
Automatizace je platná, pokud zachovává skutečnou roli, prostředí a průkaznost.
Lidský test doplň tam, kde rozhoduje srozumitelnost nebo skutečný browserový tok;
povinného owner kroku se drž podle místního kontraktu.

## H. Podmíněně: aplikace sama používá LLM / agenta

Samotné AI-asistované programování tuto sekci neaktivuje.

- **H1 🔴** Obsah uživatele, RAG, webu a tool output nemůže změnit serverové oprávnění.
  Oprávnění se ověřuje mimo model pro uživatele i každý nástroj/tenant.
  „Ignore previous instructions“ filtr ani systémový prompt nejsou bezpečnostní hranice.
- **H2 🔴** Výstup modelu jde přes validaci před SQL/shell/HTML/API akcí;
  nástroje mají omezené cíle a rozsah. Citlivé efekty respektují místní schvalovací proces.
- **H3 🟡** Limity počtu kroků, času, nákladů a egress, korelovaná stopa volání,
  bezpečné zastavení a zrušení. Změna modelu/promptu/tool konfigurace vyvolá
  opakování relevantních adversariálních scénářů.

## I. Výkon a škálovatelnost

Audit projektu/vydání vždy vyhodnotí následující body; u změny podle dopadu.
Postup, pole důkazu a volba nástrojů: [prověrka výkonu](performance-scalability.md).
Závažnost podle kritičnosti toku: kritický nesplněný/neověřený požadavek je 🔴,
ostatní výkonový dluh 🟡. Chybějící měření se uvádí UNKNOWN, nikoli N/A.

- **I1 — kontrakt:** úlohy, kritéria a status jejich schválení, data/růst, mix práce,
  síť/zařízení a prostředí. Chybějící požadavek evidován i mimo existující spec.
- **I2 — celý tok:** čas do použitelného výsledku a dokončení, requesty/objem/render,
  backend/DB/zdroje. Skóre stránky ani SQL p95 nejsou end-to-end důkaz.
- **I3 — platná zátěž:** model příchodů, plánovaná/odstartovaná/dokončená práce,
  chyby/timeouty/dropped práce, hranice měření a zdraví generátoru.
- **I4 — růst a zotavení:** zdůvodněné datové a zátěžové body, skew/cache,
  špička/delší běh/mez podle rizika a návrat do normálu; vynechání zdůvodněno.
- **I5 — platný výsledek:** integrita a úplnost při tlaku, počet vzorků/interval,
  opakování/variabilita, obě populace úspěchů i selhání a limity přenosu důkazu.
- **I6 — rozhodnutí:** výsledek po tocích, rozsah ověřené kapacity a neověřené oblasti,
  konkrétní náprava/ověření; přijetí rizika nepřepíše UNKNOWN či FAIL.
