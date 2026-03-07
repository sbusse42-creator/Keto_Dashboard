# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static, single-page Keto food reference dashboard (German language). No build system, no package manager, no dependencies — pure HTML/CSS/JavaScript.

## Deployment

Push to `main` branch triggers automatic deployment to Azure Static Web Apps via GitHub Actions (`.github/workflows/azure-static-web-apps-gray-sea-001a48003.yml`). There is no local dev server or build step needed — open `index.html` directly in a browser to preview.

## Architecture

The app uses a two-file structure:
- `index.html` — Stable hosting entry point; embeds `content.html` via a full-viewport `<iframe>`.
- `content.html` — **The entire application**: inline CSS, inline JS, and all food data. This is the only file that should be modified for content/feature changes.

### content.html internals

All food data lives in a single `const RAW` array at the top of the `<script>` block. Each entry is a tuple:
```
[name, tauglichkeit, kcal, kh_gesamt, kh_netto, protein, fett, gruppe]
```
- `tauglichkeit`: `"voll"` (fully keto-compatible) or `"bedingt"` (conditionally compatible)
- `gruppe`: category string used for grouping (e.g. `"Fleisch & Geflügel"`, `"Gemüse"`)
- Nutritional values are per 100g

**State variables:**
- `activeTabTaug` — current tauglichkeit tab filter (`'all'`, `'voll'`, `'bedingt'`)
- `activeCat` — active category filter (empty string = all)
- `filterVals[0..6]` — filter inputs indexed as: 0=name, 1=tauglichkeit, 2=kcal, 3=kh_gesamt, 4=kh_netto, 5=protein, 6=fett
- `sortCol`, `sortDir` — current sort column index and direction

**Responsive layout:**
- Desktop (>768px): sticky sidebar with filter inputs + sortable table
- Mobile (<=768px): filter grid row + card-based view
- Desktop and mobile filter inputs are kept in sync via `syncFilter()` / `syncTaugFilter()`

**Key functions:** `render()` re-renders both table and cards from current filter/sort state; `passes(row)` applies all active filters; `resetAll()` clears all state.
