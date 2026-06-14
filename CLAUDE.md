# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**文匯法律事務所工時管理暨請款單系統** — a single-page web application for a Taiwanese law firm to track billable hours, generate invoices, search legal judgments, and produce statistical reports.

## Running the Application

There is no build process. Open `index.html` directly in a browser:

```bash
# Quick local server (if needed to avoid CORS issues with file:// protocol)
python3 -m http.server 8080
# then open http://localhost:8080
```

Or simply open `index.html` directly — the app fully works as a local file.

## Architecture

The entire application is a **single file: `index.html`** (~1,700+ lines) containing inline CSS, HTML, and JavaScript. There is no framework, no bundler, no npm, and no backend.

**External libraries loaded via CDN:**
- `xlsx.js` (v0.18.5) — Excel import/export
- `html-docx-js` — Word document (.docx) generation

**Data persistence:** Browser `localStorage` only.
- `wh_time_entries` — JSON array of work entry objects
- `wh_search_history` — JSON array of past judgment searches

## Four Main Modules (Tab-based UI)

| Tab | Chinese | Purpose |
|-----|---------|---------|
| 工時管理 | Work Hours | CRUD for billable time entries; Excel/CSV import; export |
| 判決查詢 | Judgment Search | Opens external legal databases with composed search queries |
| 統計報表 | Reports | Filter/aggregate entries by month/attorney; export to Excel |
| 請款單 | Invoice | Generate Word invoices with ROC-calendar dates and Chinese monetary formatting |

## Work Entry Data Model

```js
{
  id: string,           // UUID
  caseName: string,
  client: string,
  opponent: string,     // optional
  attorney: string,
  date: string,         // YYYY-MM-DD
  hours: number,
  billingType: '計時' | '顧問' | '定額' | '無酬',
  rate: number,         // NT$/hour
  fixedAmount: number,
  disbursement: number, // optional
  summary: string,
  description: string,
  notes: string
}
```

## Key Conventions

**Dates:** The app uses both Gregorian (YYYY-MM-DD in storage) and ROC calendar (民國年, e.g. 民114年) in display. Conversion utilities are inline in the JS.

**Money formatting:** Large amounts are converted to Chinese legal numeral strings (e.g. 新台幣壹仟元整). This logic lives in a `numberToChinese()` function in the script.

**HTML escaping:** User-supplied strings rendered into HTML are escaped via a `htmlEscape()` helper to prevent XSS.

**Auto-complete:** Case names, clients, and attorneys are suggested from existing `localStorage` entries — no separate data store.

**Import flexibility:** The CSV/Excel importer uses fuzzy header keyword matching (not exact column names) to map columns, and handles merged cells, Big5 encoding, Excel serial dates, and HH:MM or decimal time formats.

## UI / Styling

CSS custom properties define the visual theme at the top of the `<style>` block:
- `--navy: #1a2847` — primary brand color
- `--gold: #b8962e` — accent color
- `--bg: #f0f2f5` — page background

The layout uses CSS Grid and Flexbox. Modal dialogs are shown/hidden by toggling a `.active` class.

## What Does NOT Exist

- No package.json, no npm, no node_modules
- No tests or test runner
- No linter or formatter config
- No backend or API calls (all data is localStorage)
- No routing — tab switching is pure DOM manipulation
