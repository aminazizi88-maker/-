# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **single-file Persian-language sports statistics management system** (`سامانه_آمار_ورزشی_v8_نهایی.html`) for the Sports and Youth Organization of Sistan and Baluchestan province in Iran. The entire application — HTML, CSS, JavaScript, and an embedded XLSX library — lives in one self-contained `.html` file with no build step, no dependencies, and no server.

Open the file in any modern browser to run it. Default credentials: `admin` / `admin123`.

## Architecture

The file is structured in three major sections:

### 1. Embedded XLSX-Lite Library (top `<script>` block, lines ~10–356)
A custom, self-contained spreadsheet engine that handles:
- **Reading**: XLSX (via `DecompressionStream` + ZIP parsing + XML parsing), CSV (synchronous), and `.xls` detection (throws, asks user to convert)
- **Writing**: builds a valid XLSX ZIP in-browser using CRC32, stores all cell values as shared strings
- Key API: `XLSX.read(arrayBuffer)` → workbook, `XLSX.utils.sheet_to_json(ws)`, `XLSX.utils.json_to_sheet(data)`, `XLSX.writeFile(wb, filename)`

### 2. CSS (inside `<style>`, lines ~360–592)
- RTL layout with CSS custom properties (`--primary`, `--accent`, `--bg`, `--sidebar-w`, `--header-h`, etc.)
- Fixed header + fixed right-side sidebar + scrollable main content
- Print stylesheet using `B Nazanin`/`Vazirmatn` fonts; hides sidebar/header, shows `#print-area`
- Utility classes: `.btn`, `.card`, `.badge`, `.modal-bg`, `.stat-card`, `.tab-btn`, etc.

### 3. Application JavaScript (bottom `<script>` block, lines ~1391–end)

**State management** — all data lives in `localStorage` under three keys:
- `sports_stat_v4` — main app state (users, years, per-year data)
- `sports_stat_history` — audit log (max 200 entries)
- `sports_stat_snapshots` — undo points (max 20, auto-created before each save)

The `state` object shape:
```js
{
  user: { username, password, orgName, reportUser },
  years: ['1402', '1403', ...],
  activeYear: '1403',
  pageTitles: { 'dashboard': '...', 'national-team': '...', ... },
  data: {
    '1403': {
      summary: { اعزام_کشور, میزبانی_کشور, مدال_تیمی, ... },  // 18 numeric fields
      national: [...],      // national team invitees/selectees
      intlTrips: [...],     // international trips/hosting
      intlMedals: [...],    // international medalists
      competitions: [...],  // national competitions
      provMedals: [...],    // provincial medalists
      provChamp: [...],     // provincial championship events
      training: [...],      // training courses
      leagues: [...],       // national league participation
      assemblies: [...],    // sports federation assemblies
      councils: [...]       // sports council meetings
    }
  }
}
```

**Key functions to know:**
- `saveState(action, page, detail)` — persists to localStorage, auto-creates snapshot if data changed
- `getYearData()` — returns `state.data[state.activeYear]`, initializing it if needed
- `showPage(page)` — SPA router; hides all `.main > div`, shows `#page-{page}`, calls the matching `render*()` function
- `renderDashboard()` — aggregates all year data into stat cards + canvas charts; uses `setTimeout` for canvas sizing
- `openAddModal(type)` / `closeAddModal()` — modal-based CRUD for all 10 data collections; `type` matches collection names (`'national'`, `'intl'`, `'comp'`, etc.)
- `exportPageExcel(page)` / `exportPageWord(page)` / `exportPagePDF(page)` — export current page data; PDF triggers `window.print()` after populating `#print-area`
- `importExcelForPage(page)` — opens Excel import modal, maps header columns to record fields
- `handlePageExcelImport(input)` — reads XLSX/CSV, maps rows to the active page's schema, appends to state

**Charts** are drawn on `<canvas>` elements using custom `drawBarChart()` and `drawPieChart()` functions (no Chart.js or external library). Charts are redrawn on every `renderDashboard()` call inside `setTimeout` to allow the DOM to lay out first.

**Persian/Jalali utilities:**
- `toFarsiDigits(n)` — converts ASCII digits to Persian Unicode
- `toArabicDigits(s)` — inverse; used when reading user input
- Date picker modal (`#date-modal`) provides a Jalali calendar selector; `confirmDate()` writes the selected date back to the target input field

## Data Flow for CRUD Operations

1. User clicks "افزودن" (Add) → `openAddModal(type)` dynamically renders a form in `#add-modal-body`
2. `document.getElementById('add-modal-save').onclick` is set to a closure capturing the type
3. On save: new record gets `id: Date.now()`, is pushed to the appropriate array in `getYearData()`, then `saveState()` is called
4. `renderXxx()` is called to refresh the table; `updateBadges()` updates sidebar counts
5. Edit: same modal, pre-filled; record is found by `id` and replaced in-place
6. Delete: `confirm()` → splice by `id` → `saveState()` → re-render

## Export Architecture

- **Excel**: `XLSX.utils.json_to_sheet(rows)` → `XLSX.writeFile(wb, filename.xlsx)` — triggers browser download
- **Word**: builds an HTML string with inline RTL styles, wraps in an Office XML namespace `<html>` document, creates a Blob with `application/msword` MIME type, downloads as `.doc`
- **PDF/Print**: populates `#print-area` with print-optimized HTML (hidden by default, shown via `@media print`), calls `window.print()`, then hides `#print-area` again

## Conventions

- All user-facing text is in **Persian (Farsi)**, RTL direction. Variable names in `state.data[year].summary` use Persian Unicode keys (e.g., `اعزام_کشور`).
- IDs for data records are `Date.now()` timestamps.
- The `intlTrips` array was added later; `getYearData()` defensively initializes it when missing from older saved states.
- Canvas charts must be redrawn after DOM layout — always wrap `drawBarChart`/`drawPieChart` calls in `setTimeout(() => { ... }, 0)` or after layout resolves.
- The app uses `localStorage` only — there is no backend, no network requests (except the Google Fonts import in CSS).
- Authentication is entirely client-side; credentials are stored in plaintext in `localStorage`.
