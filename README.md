# Pentatonica Lab

Guida didattica interattiva per chitarra blues-rock — fretboard SVG, mood board dei gradi, sistema CAGED, 4 brani di Gary Moore.

## Come aprirla

**Doppio click su `pentatonica_lab.html`**. Funziona offline su qualsiasi browser moderno (Chrome, Edge, Firefox, Safari). Non serve server.

## Struttura

Tutto è contenuto in un singolo file HTML (~2200 righe). Internamente è organizzato in tre zone:

| Zona | Righe | Contenuto |
|---|---|---|
| `<style>` | ~10–460 | Variabili tema (dark/light), tipografia, layout, fretboard SVG, card, controlli |
| `<body>` | ~460–1560 | 9 sezioni didattiche (`#mood`, `#box`, `#caged`, `#fullneck`, `#triadi`, `#progr`, `#allenamento`, `#brani`, `#aggiungi`) |
| `<script>` | ~1560–end | Constanti, theme manager, music helpers, fretboard renderer, pattern e dati per ogni sezione |

I banner-comment dentro `<style>` e `<script>` (`/* ============ ... ============ */`) marcano le aree logiche.

## Dati che probabilmente vorrai modificare

| Cosa | Dove | Note |
|---|---|---|
| Colori dei 12 gradi | `:root { --R, --b9, …, --M7 }` in CSS + `DEG_COLOR` e `DEG_TXT_COLOR` in JS | Cambiando un colore qui si aggiorna ovunque. Il colore del testo dentro al cerchio è scelto per WCAG AA. |
| Palette tema chiaro | `:root[data-theme="light"] { … }` | Tutti i colori "panna calda · ambra scura". |
| 5 box pentatoniche | `PENTA_BOXES_BASE` | Riferimento in Am. Cambia tonalità via toggle: i fret e le descrizioni si calcolano in automatico. |
| 5 forme CAGED | `CAGED_SHAPES` | Ogni shape ha `off`, `maxFret`, `triadsMinor`, `triadsMajor`. Per un fix musicale ricalcola i 3 punti della triade. |
| 7 pattern 3NPS | `THREE_NPS_AEOLIAN` | Offset relativi al "tonic fret" della 6ª corda. Si trasla automaticamente per ogni tonalità. |
| Brani | `SONGS_DATA` | Array di 4 brani con `progression`, `tones`, `tips`. Aggiungi un oggetto per inserire un brano nuovo. |

## Cosa è stato fatto in questa iterazione (v2 · fondamenta)

### TASK 1 — Riorganizzazione interna
- File singolo conservato per garantire offline-first via doppio click.
- Banner-comment estesi in CSS e JS.

### TASK 2 — Bugfix critici
- **Sezione 02**: `PENTA_BOXES` ora è parametrico sulla tonalità via `boxesFor(key)` + `deltaFromA(key)`. Cambiando tonalità si aggiornano numeri tasto, descrizioni e fret signature. Le tonalità basse (Gm) shiftano verso il basso, quelle alte (Bm/Em) verso l'alto in modo coerente.
- **Sezione 03**: `CAGED_SHAPES` riscritto con `triadsMinor` e `triadsMajor` di **esattamente 3 punti** che corrispondono a 1-b3-5 (o 1-3-5). Il poligono è ora un triangolo "pulito" (ordering polare attorno al centroide). Aggiunto toggle **"Mostra outline"**.
- **Sezione 04**: rimossa la pseudo-vista "3 note per corda · diagonale". Sostituita con i **7 pattern 3NPS reali** della scala minore naturale (Aeolian → Locrian → Ionian → Dorian → Phrygian → Lydian → Mixolydian), trasponibili automaticamente su ogni tonalità.

### TASK 3 — Palette + dual theme
- **Nuova palette dei 12 gradi** (oro / pesca / rosso-arancio / rosa salmone / oliva / viola / turchese / magenta / giallo / arancio-bruciato / smeraldo / indaco). Massima distinzione visiva, identica nei due temi → memoria visiva del grado.
- **Theme switcher** in alto a destra nel nav. Persiste in `localStorage.pentaLab.theme`. Lo script di pre-paint nel `<head>` evita il flash del tema sbagliato al refresh.
- **Tema light** caldo (panna · cuoio · ambra scura), nessun bianco freddo.
- Tutti gli `#1a1410` e `#f5ead5` hardcoded nel CSS sono stati migrati a variabili `var(--bg)`, `var(--bg-code)`, `var(--paper)` ecc.
- Il **colore del testo** dentro ai cerchi (`note-txt`, `mood-dot`, `lc-dot`) è ora calcolato da `DEG_TXT_COLOR` per garantire contrasto WCAG AA su qualsiasi colore di sfondo.

## Cosa è stato fatto in v3 · Fase 1 (visual refresh + label mode)

### Step 1.1 — Style refresh dei pallini
- Bordi più definiti: `.note-bg` stroke 2px, `.note` stroke 2.5px
- Root note: drop-shadow da 6px → 10px con `var(--shadow-glow)`, stroke 2px
- Chord tone: stroke 2.5px, drop-shadow 8px
- Testo: font-size 9.5px → **11px**, aggiunto `letter-spacing:.02em`
- Raggi cerchi: root 12→**14**, chord-tone 11→**13**, normale 10→**11**

### Step 1.2 — Label mode universale
- Tre modalità globali: **Gradi** (default) / **Note** / **Nota+Grado** — persistite in `localStorage.pentaLab.labelMode`
- Tre pulsanti sticky nel nav (`[data-label-mode]`) aggiornano in tempo reale tutti i fretboard del documento (sezioni 02, 03, 04, 08)
- Funzione `setLabelMode(m)` chiama `rerenderAllFretboards()` già esistente
- *(Copre la parte "selettore label-mode" di TASK 4)*

### Step 1.3 — Zone colorate nelle box pentatoniche (sezione 02)
- `buildBoxHull(notes, sfR, efR, W, H)` → path SVG con rettangolo arrotondato che racchiude le note di ogni box
- `BOX_HULL_COLORS` = 5 colori caldi/freddi (ambra, rosso, teal, verde, viola)
- Il rettangolo colorato è iniettato nel SVG via `insertAdjacentHTML('afterbegin')` → dipinto sotto i pallini
- CSS `.box-zone`: `fill-opacity:.10`, `stroke-opacity:.5`

### Step 1.4 — Prompt sezione 09 aggiornato
- Schema v3 con tutti i campi: `id`, `progId`, `fbId`, `chId`, `scId`, `tipId`, `rootKey`, `scaleNotes`, `fbStart`, `fbEnd`, `progression`, `tips`, `lickTitle`, `lickTab`, `lickWhy`, `analysis`
- Convenzione ID DOM documentata (`progId = "prog-" + id`, ecc.)
- Nota sulla sintassi `fb-ref` span per linkare testo → fretboard (Feature da implementare in Fase 2)
- Regola: `tones` = nomi-nota reali, `sym` usa ♭/♯ non b/#
- Istruzione apertura: restituisci SOLO blocco JSON

## Cosa NON è incluso (rinviato)

- **TASK 4** (parte rimanente) — overlay fretboard multipli per sezione, hover-link `fb-ref` testo↔fretboard (il selettore label-mode è stato completato in v3 Fase 1).
- **TASK 5** — Sistema import brani con localStorage (textarea + validazione + lista brani aggiunti).
- **TASK 6** — Pulsante "Scarica progetto" (export del file con dati embeddati).
- **TASK 7** (parte) — Test finale browser-by-browser completo.

## Dove riprendere

Per continuare:
1. **TASK 4**: la spina dorsale c'è già — basta aggiungere una funzione `setupFBControls(fbId, opts)` che inietti label-mode + overlay-picker sopra ogni SVG, e un event delegator su `.fb-ref[data-target]`.
2. **TASK 5**: aggiungere markup nella sezione 09, una funzione `loadUserSongs()` che faccia `JSON.parse(localStorage.getItem('pentaLab.userSongs'))` e concateni a `SONGS_DATA` prima del rendering.
3. **TASK 6**: serializzare `document.documentElement.outerHTML` + iniezione del JSON di `userSongs` in un `<script>` placeholder; `Blob` + `URL.createObjectURL` + `<a download>`.

## Backup

`pentatonica_lab.backup.html` è la copia esatta del file pre-refactor (v1). Se qualcosa si rompe, copia quello su `pentatonica_lab.html` e riparti.
