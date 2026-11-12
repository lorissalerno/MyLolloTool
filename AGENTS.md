# MyLolloTool — AGENTS.md

Contenitore statico di tool single-file HTML, deploy su GitHub Pages (`.github/workflows/static.yml`, push su `main`).
Nessuna build, nessun bundler. UI in italiano.

## Struttura

- `index.html` — shell: sidebar + `<iframe id="frame">`. Sidebar apre via hash (`#lollotool`, `#turni`, `#datasales`, `#agenti`). Nessun framework.
- `LolloTool V3.11.html` — SCT TOOL (Clima RAME/FIBRA/Clima3/TV-BOX/Monitoring). Pattern "store in JS" su paste (non mettere valori enormi nei textarea). Watermark autore + istruzioni agente nel commento iniziale: non rimuoverli.
- `Turni Swisscom.html` — turni con `html[data-theme]`. Storage: `turni_v7`, `turni_tema`, `turni_link`, `turni_meta7`.
- `DataSales.html` — DB SQLite via sql.js + Chart.js/jspdf da CDN. Default scuro. Banner modifiche non salvate: default CSS `display:none`, mostrato solo via JS (`updateUnsavedBackupBanner`).
- `agenti.html` — card link agenti, storage `mylollotool-agenti`. Toggle larghezza card (280px/360px) in toolbar, stato in `mylollotool-agenti-width`. Nessuna dipendenza esterna.
- `DataSales.html`, `Turni Swisscom.html` — hanno dipendenze CDN: i test offline devono tollerare errori rete.

## Tema globale unificato (2026-11-12)

Fonte unica: selettore Chiaro/Scuro in fondo alla sidebar di `index.html` (`#theme-box`, segmented control con SVG sole/luna, senza label testuale).

- Chiave globale: `localStorage['mylollotool-theme']` = `light` | `dark` (default `light`).
- Chiavi storiche sincronizzate da `index.html` (`syncLegacyKeys`): `sct-theme`=`light|dark`, `turni_tema`=`white|dark`, `theme`=`light|dark`.
- Propagazione live: `index.html` → iframe via `postMessage({type:'mylollotool-theme', theme})` su click e su `iframe load` + retry 120ms.
- Nessun reload richiesto: `paintFrameNow()` scrive direttamente nel DOM dell'iframe (stessa origine): `body.dark` (Lollo/agenti), `.light-theme` (DataSales), `dataset.theme` (Turni).
- `fixToggleOverlap()` in `index.html`: il pulsante menu (`#toggle` 40px a 14,14, centro 34px come header DataSales 68px) sposta a destra gli header toccati (`.app-header`, `.header-row`, `header`, `.brand`, `h1`) con `marginLeft` fino a 66px. Usa posizione originale per non flappare; retry su `load`+400ms e `resize`.
- Menu personalizzabile: matita in testata sidebar → modalità modifica con su/giù (riordina) e occhio (mostra/nascondi). Stato in `localStorage['mylollotool-menu']` (ordine + hidden); nuove voci in coda; `loadPage` usa solo voci visibili.
- Backup via `storage` event in entrambe le direzioni (stessa origine, anche altre schede).
- Ritorno: le pagine figlie rispondono con `window.parent.postMessage({type:'mylollotool-theme', theme})` quando l'utente cambia tema dentro la pagina; `index.html` aggiorna solo bottoni e chiavi (broadcast=false per evitare loop).
- Apertura diretta (senza iframe): ogni pagina legge prima la chiave globale, poi fallback storico.

Dettagli per file (palette unica DataSales — dark `#0D0D10/#141417/#1A1A1F`, light `#F3F4F6/#FFFFFF/#F9FAFB`; testo dark `#EEEEF0`, light `#1F2937`; accent `#3B82F6`/`#60A5FA` dark, `#2563EB` light; bordi dark `#2A2A35`, light `#E5E7EB`):

- LolloTool: `body.dark`. Usa variabile `GLOBAL_KEY`, non stringa letterale (grep cerca `GLOBAL_KEY`). Listener `message` + `applyTheme(t, notifyParent)`.
- Turni: `documentElement.dataset.theme` (`white`|`dark`). Helper `readTemaGlobale()` / `setTema(t, notifyParent)`. `white` = light globale.
- DataSales: `:root.light-theme` (assenza classe = dark). Helper `applyThemeFromGlobal()`. Non chiamare `renderStats()` prima del boot (`chMonthly/chTypes` in TDZ): usare try/catch esistente.
- agenti: `body.dark` (CSS aggiunto in questo repo). Solo ricezione + lettura iniziale, nessun bottone tema nella pagina.

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
```

Check sintassi JS (estrarre solo `<script>` senza `src`, dopo `<!DOCTYPE html>` per LolloTool — il commento iniziale contiene la stringa `<script>` e dà falso errore):

```bash
python3 -c "estrai script e node --check"
```

Con Playwright: naviga su `index.html`, apri sidebar (`#toggle`), clicca `#theme-dark`, verifica `localStorage` (`mylollotool-theme`, `sct-theme`, `turni_tema`, `theme`) e classi iframe (`body.dark` / `dataset.theme` / `.light-theme`). Screenshot sempre con `scale:"css"`. Errori console attesi non bloccanti: `favicon.ico 404`, immagine esterna `rcp.scsstatic.ch`.

## Convenzioni

- Single-file: niente nuovi file asset salvo necessità; inline CSS/JS.
- Niente `localStorage.clear()` aggressivi: solo chiavi proprie.
- Testi UI in italiano. Pesi file: DataSales ~160KB, Lollo ~110KB — non duplicare librerie.
- Push su `main` = deploy Pages automatico. Verificare `git status` prima del commit; non committare `.playwright-mcp/`, screenshot `*.png`, log server.
