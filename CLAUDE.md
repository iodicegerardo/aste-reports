# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Does

**aste-reports** is a static hosting repository for the Aste Cremona project. It serves as:

1. **GitHub Pages host** — the `gh-pages` branch is the live site at `https://iodicegerardo.github.io/aste-reports/`
2. **Persistent storage** for `seen_lotti.json`, the deduplication registry shared across runs of the scraper in the `iodicegerardo/aste` repository

There is no code to run here. All content is generated and pushed automatically by the GitHub Actions workflow in the `iodicegerardo/aste` repo.

## Repository Structure

| File | Description |
|---|---|
| `index.html` | Latest HTML report (overwritten on every run) |
| `report_YYYY-MM-DD.html` | Dated copy of each run's report |
| `seen_lotti.json` | Deduplication registry — dict keyed by lotto ID, value is full lotto data + `data_analisi` field |
| `manifest.json` | Web App Manifest per installabilità PWA (nome, icone, colore tema) |
| `sw.js` | Service Worker (network-first caching, offline fallback sull'ultimo report) |
| `icon-192.png` | App icon 192×192 px per Android Chrome e home screen |
| `icon-512.png` | App icon 512×512 px per splash screen PWA |

## How Content Gets Here

The `aste_daily.yml` workflow in `iodicegerardo/aste`:

1. Clones this repo (gh-pages branch) using `REPORTS_REPO_TOKEN`
2. Copies `seen_lotti.json` from here to the scraper's `output/` before running
3. After scraping, force-pushes back:
   - `index.html` ← `output/report_YYYY-MM-DD.html`
   - `report_YYYY-MM-DD.html` ← same file
   - `seen_lotti.json` ← updated registry
   - `manifest.json` ← `static/manifest.json` from `aste` repo
   - `sw.js` ← `static/sw.js` from `aste` repo
   - `icon-192.png`, `icon-512.png` ← generati da `generate_icons.py` a build-time

## PWA Assets

Questo repo ospita anche i file statici necessari per la PWA (Sprint 2):

- **`manifest.json`** — dichiara nome, icone, `theme_color: "#2c5282"`, `start_url: "./"` e `scope: "./"` (percorsi relativi obbligatori perché il sito vive su `/aste-reports/`)
- **`sw.js`** — service worker con strategia network-first: aggiorna la cache ad ogni visita online, serve l'ultima versione in cache quando offline
- **`icon-192.png`** / **`icon-512.png`** — icone PNG con sfondo blu `#2c5282` e glifo "A" crema; generate da `generate_icons.py` in `iodicegerardo/aste` ad ogni run CI

Non modificare questi file direttamente — vengono sovrascritti ad ogni deploy. Apporta le modifiche in `iodicegerardo/aste`.

## seen_lotti.json Format

```json
{
  "<lotto_id>": {
    "id": "B2416411",
    "titolo": "...",
    "indirizzo": "...",
    "comune": "Cremona",
    "prezzo_base": 43408.44,
    "data_asta": "23/07/2026",
    "url": "https://...",
    "pdf_urls": [],
    "pdf_paths": [],
    "pdf_names": [],
    "descrizione": "...",
    "superficie": "...",
    "tribunale": "Tribunale di Cremona",
    "numero_procedura": "...",
    "analisi_ai": { ... },
    "data_analisi": "2026-05-04"
  }
}
```

The `analisi_ai` object contains Claude's structured JSON output (sintesi, giudizio_colore, rischi, oneri, etc.).

## Do Not Edit Manually

Files in this repo are machine-generated. Manual edits will be overwritten on the next workflow run. To change report content or structure, edit `scraper.py` in `iodicegerardo/aste`.

## GitHub Pages Setup

- Branch: `gh-pages`
- Root: `/` (repo root)
- URL: `https://iodicegerardo.github.io/aste-reports/`
