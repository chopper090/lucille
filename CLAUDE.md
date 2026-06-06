# CLAUDE.md — Lucille

**Scopo.** Guida interattiva alla **chitarra blues-rock**: scala pentatonica minore (5 box),
sistema **CAGED rinominato per funzione**, progressioni tipiche, 4 brani Gary Moore — su
**fretboard SVG**.

**Stack.** HTML **monolitico single-file** (`BLUE_Guide.html`, ~5260 righe: ~460 di `<style>`
con token dual + palette 12 gradi, ~1100 di markup, ~3700 di `<script>`). Vanilla JS, SVG,
`localStorage`, PWA. Font: Cormorant Garamond, JetBrains Mono, Bebas Neue. No build.

**Mappa file.** `BLUE_Guide.html` = **source autorevole**. `index.html` = landing. Dati inline:
`PENTA_BOXES`, `CAGED_SHAPES`, `THREE_NPS_AEOLIAN`, `PROGRESSIONS`, `SONGS_DATA`. `sw.js`,
`manifest`, icone.

**Dove stanno i dati.** Tutto inline nel file (nessun fetch). `localStorage` per tema/preferenze.

**Come si edita.** Box/forme/progressioni/brani → i rispettivi array in `BLUE_Guide.html`.
Palette gradi: mappatura `DEG_COLOR`/`DEG_TXT_COLOR`. Il monolite è una scelta (no split).

**Gotcha.** File paralleli da archiviare: `BLUE_Guide_Mobile.html` (deprecato),
`pentatonica_lab.backup.html` (backup), 5 `image-*.png` (screenshot di handoff → `docs/`).

**Deploy.** GitHub Pages (`chopper090.github.io/lucille/`). Versionare con `_scripts\Publish-Project.ps1`.

**Sovrapposizioni.** Fretboard in comune con **Nemo** (superset): Lucille è la guida blues focalizzata.
