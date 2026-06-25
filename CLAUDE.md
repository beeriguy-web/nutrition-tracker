# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Two standalone, self-contained Hebrew-language web apps — no build system, no package manager, no server. Both files run directly in the browser and contain all HTML, CSS, and JavaScript inline.

- **`index.html`** — Daily nutrition tracker PWA (Progressive Web App) in Hebrew
- **`work-order.html`** — Internal work order form for שמדר טכנולוגיות (Shamadar Technologies)

## Running / Development

Open either HTML file directly in a browser. No build step, no `npm install`, no server required.

To serve locally (useful for PWA features like the manifest):
```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

There are no tests, no linter, and no CI configuration.

## Architecture

### index.html — Nutrition Tracker

- **Data persistence**: `localStorage` only. Two keys:
  - `nutrition_entries` — array of logged food entries
  - `user_foods` — user-defined custom food entries
- **Food database** (`FOOD_DB`): Large inline array (~100+ entries) of Israeli foods. Each entry: `{ kw: string[], name, cal, portion, alts: [{name, cal, portion, note}] }`. The `alts` array powers the "healthier alternatives" suggestion card.
- **Food lookup** (`findFood`, `findAllFoods`): Keyword matching against `FOOD_DB` and `userFoods`. `findAllFoods` uses a longest-keyword-wins algorithm to handle compound phrases and Hebrew conjunctions (e.g. "ו" prefix).
- **Voice input**: Web Speech API (`SpeechRecognition`/`webkitSpeechRecognition`). `processVoice()` parses the transcript → `findAllFoods()` → if multiple foods detected, shows the multi-food card; if one, adds directly; if none, shows the unknown-food banner.
- **Excel export**: Uses `xlsx-js-style` CDN library. `doExport()` builds a styled workbook with per-day sheets plus a summary sheet.
- **PWA**: Manifest is generated inline at runtime via a Blob URL and injected into `<head>`. No separate `manifest.json` file.
- **Meal types**: breakfast / lunch / dinner / snack / drink — stored on each entry as `meal` field; auto-selected by time of day via `autoMeal()`.
- **UI flow**: Voice or manual input → food lookup → suggestion card (healthier alts) or multi-food card → `addEntryData()` → `save()` → `render()`.

### work-order.html — Work Order Form

- **No persistence** — form state is not saved; reloading clears it.
- **Contacts database** (`CONTACTS_DB`): Large inline JSON object. Keys are client company names; values are arrays of `{ n, p, e }` (name, phone, email). Currently contains contacts for two companies: `דניה סיבוס` and `רמי שבירו הנדסה`.
- **Client autocomplete**: `onClientInput()` filters `CONTACTS_DB` keys; selecting a client enables contact-row autocomplete filtered to that client's contacts.
- **Signature pad**: Canvas-based, supports both mouse and touch events. `getSignatureDataUrl()` returns `null` if the canvas is blank (detected by comparing to an empty canvas).
- **PDF export**: `buildPdfHTML()` constructs an HTML string → injected into `#pdf-zone` → `window.print()`. Print CSS hides all interactive elements.
- **Email**: `sendEmail()` constructs a `mailto:` link with pre-filled subject/body and opens it; no backend.
- **Items table**: Dynamically rendered rows. `addRow()` / `removeRow()` manage them; `collectRows()` harvests values.

## Key Conventions

- **Language**: All UI text, variable names in comments, and data are in Hebrew. Code identifiers and function names are English.
- **RTL layout**: Both files use `dir="rtl"` and `lang="he"`. `index.html` uses Heebo font; `work-order.html` uses Bootstrap 5.3.2 RTL CDN.
- **No external state**: Everything is self-contained. No API calls, no backend, no cookies — only `localStorage` in `index.html`.
- **Inline everything**: CSS and JS live inside each HTML file. There are no separate `.css` or `.js` files.
- **FOOD_DB is the source of truth**: When adding or changing food entries in `index.html`, follow the existing schema exactly: `{ kw, name, cal, portion, alts }`. Keywords (`kw`) drive all lookup logic — add synonyms and common misspellings as additional entries in the `kw` array.
- **CONTACTS_DB is hardcoded**: Editing contacts requires modifying the large inline JSON object in `work-order.html`. The structure per contact is `{ n, p, e }` (name, phone, email).
