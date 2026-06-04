# Pentatonica Lab — Refactoring v2

## Contesto
File `pentatonica_lab.html` (~2100 righe, monolitico): guida didattica chitarra blues-rock con fretboard SVG, mood board gradi, CAGED, 4 brani Gary Moore. Architettura attuale: HTML+CSS+JS inline. Funzionante ma con bug e da espandere.

## Obiettivo
Refactoring in progetto modulare offline-first, fix bug, aggiunta interattività, nuova palette, sistema import brani con persistenza.

---

## TASK 1 — Refactoring strutturale
Splitta il monolite in:
```
pentatonica-lab/
├── index.html          # markup + import script/css
├── css/
│   ├── theme.css       # variabili CSS (light + dark) + tipografia
│   └── components.css  # tutti gli stili componenti
├── js/
│   ├── theory.js       # NOTES, TUNING, scale builder, degree mapping
│   ├── fretboard.js    # renderFB() + overlay (triadi, arpeggi, mood)
│   ├── ui.js           # toggle, controlli, hover-link testo↔fretboard
│   ├── songs.js        # SONGS_DATA + renderSong + import handler
│   └── app.js          # init, event binding, theme toggle
└── data/
    └── songs.json      # dati brani (caricati via fetch o inline fallback)
```

**Vincolo offline**: il file deve funzionare con doppio click su `index.html` senza server. Se `fetch()` fallisce per protocollo `file://`, usa fallback con dati inline. Alternativa: tutto in un file unico se più semplice mantenere offline-first puro. **Decidi tu** — priorità è funzionare offline.

---

## TASK 2 — Bug fix critici

### 2.1 Sezione 02 (Box pentatoniche)
Quando cambia tonalità via toggle `[data-key]`:
- ❌ Attualmente cambiano solo le note sui fretboard
- ✅ Devono aggiornarsi anche: numeri di tasto nel `box-fret`, descrizioni `box-info` (es. "V tasto in Am" → "III tasto in Gm"), nomi CAGED associati, riferimenti a tasti nel testo

**Soluzione**: rendi `PENTA_BOXES` parametrico sul `key` selezionato. Calcola gli offset dei fret in base alla differenza tra root corrente e A.

### 2.2 Sezione 03 (CAGED)
- ❌ La linea tratteggiata che dovrebbe collegare le note della triade non corrisponde alle note reali sul fretboard
- ✅ Ricalcola le coordinate: i `triads` in `CAGED_SHAPES` devono essere mappati correttamente sulla posizione di Am scelta sul manico
- ✅ La poligonale deve **toccare i centri** dei cerchi delle 3 note della triade (1-3-5 o 1-b3-5)
- ✅ Aggiungi opzione mostra/nascondi outline

### 2.3 Sezione 04 — vista "3 note per corda"
La vista attuale è mal definita. Sostituisci con:
- **"Pattern 3NPS (3 Notes Per String)"**: scala diatonica (aeolian o ionian) con 3 note per corda — 7 posizioni reali del 3nps system
- Mostra una posizione alla volta con selettore (1-7), o pattern unico evidenziato sul manico completo

---

## TASK 3 — Nuova palette + theme switcher

### 3.1 Dual theme (dark/light)
Toggle in alto a destra (sticky, accanto al nav). Salva preferenza in `localStorage`.

**Vincolo**: entrambi i temi devono essere caldi e ad alto contrasto, non affaticanti.

- **Dark**: tabacco scuro, ocra, ruggine (mantieni vibe attuale ma raffinato)
- **Light**: panna calda, cuoio chiaro, ambra scura per testo (NO bianco freddo, NO grigio)

### 3.2 Colori dei gradi — nuova logica
Sostituisci la palette attuale (troppo simile tra gradi) con **12 colori massimamente distinti** che funzionino su entrambi i temi:

```
Tonica (1):    oro brillante      #FFB300
9:             pesca vivace       #FF8A50
b3:            rosso-arancio      #E53935
3:             rosa salmone       #FF6F61
4:             verde oliva caldo  #8BC34A
b5:            viola violaceo     #9C27B0  (eccezione: serve contrasto come "blue note")
5:             turchese caldo     #00ACC1  (eccezione necessaria per leggibilità)
b6:            magenta scuro      #C2185B
6:             giallo limone      #F9A825
b7:            arancio bruciato   #E65100
7:             verde smeraldo     #2E7D32
b9:            indaco caldo       #5E35B0
```

**Logica**: stesso colore per stesso grado in TUTTO il documento. Memoria visiva = fondamentale per riconoscimento pattern. Cerchi grandi e saturi, contrasto alto, etichetta del grado **leggibile** dentro il cerchio (testo bianco o nero scelto per contrasto WCAG AA).

---

## TASK 4 — Interattività universale dei fretboard

### 4.1 Selettore "label mode" per ogni fretboard
Tendina/pulsanti sopra ogni fretboard:
- `Solo gradi` (es. `1`, `b3`, `5`) — default
- `Solo note` (es. `A`, `C`, `E`)
- `Note + gradi` (es. `A·1`, `C·b3`)

### 4.2 Selettore "overlay" per ogni fretboard
Menu dropdown universale che permette di sovrapporre evidenze:
- **Triade** (1-3-5 o 1-b3-5) — cerchio o linea che racchiude le 3 note tipo cruciverba
- **Arpeggio** dell'accordo (1-3-5-b7) — sequenza con linea spezzata
- **Solo chord tones** dell'accordo corrente
- **Mood**: evidenzia solo le note "tese" (tutti i gradi 4, b5, b9, b6) o solo quelle "stabili" (1, 3/b3, 5)
- **Power chord** (1+5)
- **Triade in inversione** (3-5-1, 5-1-3)

Il rendering aggiunge linee/cerchi sul fretboard SVG. Pulsanti multi-select (più overlay attivi insieme).

### 4.3 Hover-link testo↔fretboard
Nei testi delle descrizioni e dei tip dei brani, ogni nota o tasto citato (es. `B`, `4° tasto 4ª corda`, `12° tasto 2ª corda`) diventa un **link interattivo**.

**Sintassi nel sorgente**:
```html
<span class="fb-ref" data-note="B" data-string="1" data-fret="12" data-target="fb-loner">B al 12° tasto, 2ª corda</span>
```

**Comportamento**:
- Hover/click → highlight (alone pulsante) sulla nota indicata nel fretboard `data-target`
- Tutte le occorrenze della stessa nota nel fretboard si illuminano insieme se `data-fret` è omesso
- Funziona per qualsiasi `data-target` indicato

Aggiorna **tutti i testi descrittivi** della guida (sezioni 02, 03, 04, 05, 06, 08) per usare questa sintassi sui riferimenti fretboard.

---

## TASK 5 — Sistema import brani

### 5.1 UI Import (sezione 09)
Aggiungi sotto al prompt copiabile:
1. **Textarea** etichettata "Incolla qui il JSON del brano"
2. **Pulsante "Aggiungi brano"** → valida JSON, lo aggiunge a `localStorage.userSongs[]`, ricarica sezione 08
3. **Pulsante "Reset brani aggiunti"** → svuota `localStorage.userSongs`
4. Mostra lista brani caricati con bottone elimina per ognuno
5. **Persistenza**: al refresh i brani restano

### 5.2 Aggiorna prompt copiabile
Il prompt nel `<pre id="prompt-text">` deve essere riscritto per riflettere:
- Schema JSON aggiornato (con campi per hover-link, overlay supportati, ecc.)
- Istruzioni esplicite per l'output: "Restituisci SOLO il JSON, senza commenti, in un blocco \`\`\`json"
- Esempi di sintassi `fb-ref` per i tip
- Validazione: ogni accordo deve avere `tones` come array di note (non gradi)

### 5.3 Validazione import
Prima di salvare:
- Parse JSON con try/catch
- Verifica campi obbligatori (`id`, `title`, `rootKey`, `progression`, `tips`)
- Verifica `id` univoco vs brani esistenti
- Mostra errore inline se non valido

---

## TASK 6 — Export del progetto

Pulsante "Scarica progetto" in fondo alla sezione 09:
- **Opzione A** (preferita): genera uno zip con tutti i file (usa client-side, libreria JSZip via CDN inline) — ma deve funzionare offline, quindi **embedding inline** della libreria
- **Opzione B** (fallback): genera **un singolo file HTML** con tutto inline (CSS+JS+JSON) tramite Blob+download. Inserisce anche i brani salvati in localStorage come dati hardcoded così sono portable

**Decidi tu** quale opzione: priorità è semplicità e offline. Probabilmente B.

---

## TASK 7 — Test finale
Una volta finito:
1. Apri `index.html` con doppio click → deve funzionare senza errori console
2. Switcha tema → deve persistere
3. Cambia tonalità in sezione 02 → tutto coerente
4. CAGED in sezione 03 → linee tratteggiate sui punti giusti
5. Hover su `fb-ref` in qualsiasi sezione → fretboard reagisce
6. Aggiungi brano via JSON → appare in sezione 08, persiste al refresh
7. Esporta → file scaricato funziona offline

---

## Note operative per Claude Code

- **Lavora in modalità autonoma**, non chiedere permessi per ogni file
- **Non rifare il lavoro già fatto**: parti da `pentatonica_lab.html` e refattorizza, non ricostruire da zero contenuti come mood board, brani, tabelle
- **Mantieni**: il contenuto didattico delle sezioni 01, 04, 05, 06, 07 è già curato — preserva i testi
- **Usa `git init` + commit progressivi** così posso fare rollback se serve
- **Crea un `README.md`** breve nella cartella che descrive struttura e come aggiornare
- Quando finisci, scrivimi un riassunto di **cosa hai fatto** e **cosa eventualmente non sei riuscito a fare**, con eventuali trade-off

## Priorità di esecuzione
Se il task è troppo lungo per una sessione, segui questo ordine:
1. TASK 1 (refactoring) + TASK 3 (palette/theme) — fondamenta
2. TASK 2 (bug fix) — qualità base
3. TASK 4 (interattività) — UX
4. TASK 5 (import brani) — espandibilità
5. TASK 6 (export) — distribuzione

## Output finale atteso
Una cartella `pentatonica-lab/` pulita, modulare, offline-first, con tutti i task completati o documentati come "non fatti" nel README con motivazione.
