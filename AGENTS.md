# MyLolloTool — AGENTS.md

Contenitore statico di tool single-file HTML, deploy su GitHub Pages (`.github/workflows/static.yml`, push su `main`).
Nessuna build, nessun bundler. UI in italiano.

## Struttura

- `index.html` — shell: sidebar + `<iframe id="frame">`. Sidebar apre via hash (`#lollotool`, `#turni`, `#datasales`, `#agenti`). Nessun framework. `#sidebar-foot{margin-top:auto}` contiene `#changelog-box` + `#theme-box`: Novità resta ancorata al fondo giusto sopra lingua/tema. `#sidebar-logo` 40px tonda in alto a destra (`top:14px;right:14px`), stessa altezza del pulsante menu.
- `LolloTool V3.11.html` — SCT TOOL (Clima RAME/FIBRA/Clima3/TV-BOX/Monitoring). Pattern "store in JS" su paste (non mettere valori enormi nei textarea). Watermark autore + istruzioni agente nel commento iniziale: non rimuoverli.
- `Turni Swisscom.html` — turni con `html[data-theme]`. Storage: `turni_v7`, `turni_tema`, `turni_link`, `turni_meta7`. Header senza logo (logo spostato in `#sidebar-logo`): `.brand{margin-left:46px}`, `header` centrato `min-height:40px`, header compatto (`.wrap` `14px 20px 6px` desktop / `14px 12px 6px` mobile, bordo titolo sempre a 66px dal bordo: `.brand` 46px desktop / 54px mobile, ma `@media (min-width:1060px){.brand{margin-left:0}}` così su schermo largo il titolo si allinea alla riga blu (nessuna sovrapposizione col pulsante oltre i 1052px). Transizione `margin-left .25s ease` su `.brand`: il passaggio 46↔0 (e 46↔54 mobile) è animato, nessun salto al resize, `header` `padding-bottom:8px`/`margin-bottom:12px`, `h1` 20px/`1.25`, `.sub` 11px; pulsanti header 40px su pc e mobile (come il pulsante menu), icona-sola `.stato` su mobile per lo spazio).
- `DataSales.html` — DB SQLite via sql.js + Chart.js/jspdf da CDN. Default scuro. Banner modifiche non salvate: default CSS `display:none`, mostrato solo via JS (`updateUnsavedBackupBanner`). Su telefono (`≤600px`) le mini-barre KPI header sono nascoste (restano nelle card) così logo + anno ci stanno; altezza header 56px.
- `agenti.html` — card link agenti, storage `mylollotool-agenti`. Toggle larghezza card (280px/360px) in toolbar, stato in `mylollotool-agenti-width`. Nessuna dipendenza esterna.
- `novita.html` — pagina "Novità": storico modifiche funzionali (solo uso essenziale, niente grafica). Stessa palette agenti, `body.dark`, i18n it/de/fr/en, nessun bottone tema/lingua. Costante `SITE_VERSION` nel `<script>`.
- `DataSales.html`, `Turni Swisscom.html` — hanno dipendenze CDN: i test offline devono tollerare errori rete.

## Tema + lingua globali (2026-11-12)

Fonte unica in fondo alla sidebar di `index.html` (`#theme-box`, riga `.pref-row`): pulsante tema icona (click = cambia, metodo SCT) + pulsante lingua `IT/DE/FR/EN` (click = ciclo, metodo SCT).

- Chiavi globali: `localStorage['mylollotool-theme']` (`light`|`dark`, default `light`), `localStorage['mylollotool-lang']` (`it|de|fr|en`, default `it`).
- Chiavi storiche sincronizzate da `index.html`: tema (`syncLegacyKeys`: `sct-theme`, `turni_tema`=`white|dark`, `theme`) e lingua (`sct-lang`, legacy SCT Tool).
- Propagazione live: `index.html` → iframe via `postMessage({type:'mylollotool-theme'|'mylollotool-lang', ...})` su click e su `iframe load` (+ retry tema 120ms); backup via `storage` event.
- Nessun reload richiesto: `paintFrameNow()` scrive direttamente nel DOM dell'iframe (stessa origine): `body.dark` (Lollo/agenti), `.light-theme` (DataSales), `dataset.theme` (Turni).
- Sidebar `index.html`: segue il tema via `body[data-mtheme]` (`paintMenuTheme` in `setGlobalTheme` + handler `storage`): light `#E5E7EB`/testo `#1F2937`, dark `#26262E`/testo `#EEEEF0` (più chiaro del dark pagine `#0D0D10`).
- `fixToggleOverlap()` in `index.html`: il pulsante menu (`#toggle` 40px a 14,14, centro 34px come header DataSales 68px) sposta a destra gli header toccati (`.app-header`, `.header-row`, `header`, `.brand`, `h1`) con `marginLeft` fino a 66px + `width:auto` (evita overflow destro con `width:100%`). Usa posizione originale per non flappare; retry su `load`+400ms e `resize`.
- Agenti: barra superiore allineata al pulsante (`h1`/`sub` con `margin-left:42px`, `h1 min-height:40px` centrato a 34px; niente più `padding-top:70px`).
- Menu personalizzabile: matita in testata sidebar → modalità modifica: trascina le voci col mouse per riordinare, occhio SVG a destra per mostra/nascondi. Stato in `localStorage['mylollotool-menu']` (ordine + hidden); nuove voci in coda; `loadPage` usa solo voci visibili.
- Backup via `storage` event in entrambe le direzioni (stessa origine, anche altre schede).
- Ritorno: nessuna pagina figlia cambia più tema/lingua da sola (pulsanti rimossi); l'handler `message` in `index.html` resta come backup (broadcast=false per evitare loop).
- Apertura diretta (senza iframe): ogni pagina legge prima la chiave globale, poi fallback storico.

Dettagli per file (palette unica DataSales — dark `#0D0D10/#141417/#1A1A1F`, light `#F3F4F6/#FFFFFF/#F9FAFB`; testo dark `#EEEEF0`, light `#1F2937`; accent `#3B82F6`/`#60A5FA` dark, `#2563EB` light; bordi dark `#2A2A35`, light `#E5E7EB`):

- LolloTool: `body.dark`. Usa variabile `GLOBAL_KEY`, non stringa letterale (grep cerca `GLOBAL_KEY`). Listener `message` + `applyTheme(t, notifyParent)`. Pulsante tema rimosso (2026-11-12): fonte unica nel menu di `index.html`; JS con guard su `btn` nullo. Pulsante lingua `#langBtn` rimosso: `setLang` esposta come `window.__sctSetLang`, comandata da menu via `message`/`storage` (`mylollotool-lang`); dizionario it/de/fr/en già esistente.
- Turni: `documentElement.dataset.theme` (`white`|`dark`). Helper `readTemaGlobale()` / `setTema(t, notifyParent)`. `white` = light globale. Pulsante `#tema` rimosso (2026-11-12): `setTema` con guard su elemento mancante. i18n it/de/fr/en: dizionario `T` + `GIORNI_L`/`MESI_L`/`NOMI_L`, `applyLang()` (testi statici via `data-t`/`data-th`/`data-t-ph`/`data-t-title` + `paintButtons()` + re-render), listener `message`/`storage`.
- DataSales: `:root.light-theme` (assenza classe = dark). Helper `applyThemeFromGlobal()`. Non chiamare `renderStats()` prima del boot (`chMonthly/chTypes` in TDZ): usare try/catch esistente. Pulsante `#themeToggleBtn` rimosso (2026-11-12): `updateThemeIcon()` no-op, sync solo da `index.html`. i18n it/de/fr/en: dizionario `T` + `MONTHS_L`, `applyLang()` (attributi `data-t*` + re-render parziale). ATTENZIONE: `applyLang` NON deve chiamare `refresh()` (migrate/sync scrivono su DB e settano flag modifiche) ma solo `renderSales/renderProducts/loadGoals/renderStats/renderArchive`. Header CSV tradotto: l'import accetta header in tutte le lingue (`HDR` map).
- agenti: `body.dark` (CSS aggiunto in questo repo). Solo ricezione + lettura iniziale, nessun bottone tema nella pagina. i18n it/de/fr/en: dizionario `T` + `ICONL`, `applyLang()` (attributi `data-t*` + `renderIconPicker()` + re-render), listener `message`/`storage`.
- novita: `body.dark`, solo ricezione + lettura iniziale. i18n it/de/fr/en via `T` + attributi `data-t`, listener `message`/`storage` unificati (tema + lingua). Badge `#ver-badge` = `'v' + SITE_VERSION`.

## Novità + versione sito

- Link discreto `#changelog-link` in `#changelog-box`, sopra `#theme-box` (sotto `#menu-links`): icona campana + `<span id="changelog-txt">` + badge `<span id="site-version">`. Stile discreto: `font-size 12px`, `opacity .72`, hover/active come voci menu. Non fa parte di `DEFAULT_MENU` (non riordinabile/nascondibile).
- `index.html`: costante `SITE_VERSION = '1.01'`, testo badge `siteVersionEl.textContent = 'v' + SITE_VERSION`. `MENU_T.news`/`newsTitle` per i18n. `loadPage('#novita')` carica `novita.html` via `changelogLink.dataset.src`, attiva solo `#changelog-link`, aggiorna `document.title`. Click chiude il menu.
- `novita.html`: costante `SITE_VERSION` propria (stesso valore), badge `#ver-badge`. Voci `.entry` con `<strong>vX.XX</strong><time>gg/mm/aaaa</time>` + `<li>` con `<span class="tag">Pagina</span>`.
- REGOLA OBBLIGATORIA: ogni modifica funzionale importante (uso reale: lingua, tema, menu, import/export, DB, turni, calcoli LolloTool — mai grafica pura) va annotata in `novita.html` in italiano + traduzioni `T` (de/fr/en), con tag pagina e data. Solo modifiche funzionali, niente restyling.
- Versione: `+0.01` al massimo UNA volta al giorno, solo se c'è almeno una modifica funzionale importante quel giorno. Anche con più modifiche nello stesso giorno, un solo bump. Aggiornare ENTRAMBE le costanti `SITE_VERSION` (`index.html` + `novita.html`) e aggiungere la voce in `novita.html`. Data formato `gg/mm/aaaa`.

## Test locale

```bash
cd /tmp/opencode/MyLolloTool  # o la cartella clonata
python3 -m http.server 8901 &
# apri http://localhost:8901/index.html
```

Test rapidi senza browser:

```bash
grep -c "mylollotool-theme" index.html DataSales.html agenti.html
grep -c "GLOBAL_KEY\|mylollotool-theme" "LolloTool V3.11.html"
grep -c "mylollotool-lang" index.html DataSales.html agenti.html "Turni Swisscom.html"
```

Check sintassi JS (estrarre solo `<script>` senza `src`, dopo `<!DOCTYPE html>` per LolloTool — il commento iniziale contiene la stringa `<script>` e dà falso errore):

```bash
python3 -c "estrai script e node --check"
```

Con Playwright: naviga su `index.html`, apri sidebar (`#toggle`), clicca `#theme-toggle` / `#lang-btn`, verifica `localStorage` (`mylollotool-theme`, `mylollotool-lang`, `sct-theme`, `turni_tema`, `theme`, `sct-lang`) e classi iframe (`body.dark` / `dataset.theme` / `.light-theme`) + testi tradotti. Screenshot sempre con `scale:"css"`. Errori console attesi non bloccanti: `favicon.ico 404`, immagine esterna `rcp.scsstatic.ch`.

## Convenzioni

- Single-file: niente nuovi file asset salvo necessità; inline CSS/JS.
- Niente `localStorage.clear()` aggressivi: solo chiavi proprie.
- Testi UI in italiano. Pesi file: DataSales ~160KB, Lollo ~110KB — non duplicare librerie.
- Push su `main` = deploy Pages automatico. Verificare `git status` prima del commit; non committare `.playwright-mcp/`, screenshot `*.png`, log server.
