# Portfolio Tracker — Project Architecture

## Overview

`portfolio-tracker.html` is a **single-file, zero-dependency** trading portfolio tracker. All HTML, CSS, and JavaScript live inline in one file. No build step, no package manager, no server required — open the file in a browser and it works.

---

## File Structure

```
portfolio-tracker.html
├── <head>  — CDN fonts + Chart.js 4.4.0
├── <style> — all CSS (dark dashboard theme)
├── <body>  — all HTML markup
└── <script>— all application JavaScript
```

---

## Visual Theme

| Variable | Value | Usage |
|---|---|---|
| `--bg-primary` | `#0a0e1a` | Page background |
| `--bg-secondary` | `#151b2e` | Card backgrounds |
| `--bg-tertiary` | `#1e2840` | Table headers, inputs |
| `--accent-cyan` | `#00f5ff` | Primary accent, links |
| `--accent-green` | `#00ff88` | Positive P&L, today column |
| `--accent-red` | `#ff3366` | Negative P&L, short badges |
| `--border` | `#2a3752` | All borders |

Fonts: **JetBrains Mono** (primary), **Space Mono** (secondary) from Google Fonts.

---

## State Model

### Root state

```js
let allMonthsData = {};          // keyed by "${year}-${month}" (month is zero-based)
let currentMonth  = 0;           // zero-based active month
let currentYear   = 2026;        // active year
```

### Active month slice — `portfolioData`

```js
{
  contracts:       string[],               // ordered contract names
  dates:           string[],               // date labels: "DD-MMM" e.g. "04-Jun"
  cells:           Record<string, Cell>,   // see Cell Key Rules below
  hiddenRows:      Set<number>,            // runtime Set (serialised as array)
  hiddenCols:      Set<number>,            // runtime Set (serialised as array)
  notes:           string,                 // HTML content of trading notes
  contractPositions: Record<number, "L"|"S">,  // keyed by row index
  contractTargets:   Record<string, { monthTarget: string, currentValue: string }>
                                           // keyed by contractDisplayKey(name)
}
```

### `Cell` shape

```js
{ value: string, bold: boolean, larger: boolean }
```

`bold` = **realized P&L** throughout the app — charts and summaries both read this flag.  
`larger` = presentation only (bigger font), no semantic meaning.

---

## Cell Key Rules

| Key pattern | Meaning |
|---|---|
| `"${rowIndex}-${colIndex}"` | Regular editable contract cell |
| `"contract-${rowIndex}"` | Contract name cell (not persisted separately, name lives in `contracts[]`) |
| `"unrealized-${colIndex}"` | UnRealized P&L row cell |
| `"realized-${colIndex}"` | Net Realized P&L row cell |

The **Standing AT** row is derived at render-time as `unrealized + realized` — it is never stored in `cells`.

---

## Persistence

### LocalStorage

Key: `allMonthsPortfolioData`

- `hiddenRows` and `hiddenCols` are serialised as arrays; converted back to `Set` on load.
- Backward-compatible migration from the old single-month `portfolioData` key is preserved.

### Cloud Sync — JSONBin.io

```js
const SYNC_BIN_ID     = '...';   // JSONBin bin ID
const SYNC_MASTER_KEY = '...';   // JSONBin master key (client-side — known limitation)
const JSONBIN_URL     = `https://api.jsonbin.io/v3/b/${SYNC_BIN_ID}`;
```

- Manual **Push** and **Pull** only. No auto-sync on changes.
- `beforeunload` sync is best-effort — do not build logic that depends on it.
- Credentials are hardcoded in client JS (security trade-off accepted by owner).

---

## Month Navigation

`loadMonthData(month, year)` — saves current month first, then switches.  
`saveCurrentMonthData()` — spreads `portfolioData`, serialises Sets to arrays, writes to localStorage.

Month key format: `"${year}-${month}"` where `month` is **zero-based** (Jan = 0).

---

## Date Column Format

Dates in `portfolioData.dates` use the format `"DD-MMM"` (e.g. `"04-Jun"`, `"27-May"`).

**Today's column** is identified by:
```js
const todayStr = String(today.getDate()).padStart(2, '0') + '-' +
                 today.toLocaleDateString('en-US', { month: 'short' });
```

**Custom columns** (user-added beyond the natural days of the month) have `index >= daysInMonth`. Date auto-population only runs when a month has zero dates (idempotent — preserves user-added columns).

---

## Contract Grouping — `contractDisplayKey(name)`

Normalises a raw contract name to a canonical display key for the Targets card:

| Raw name | Display key |
|---|---|
| `"NIFTY JUNE"` | `"NIFTY FUT"` |
| `"NIFTY JULY"` | `"NIFTY FUT"` |
| `"BANKNIFTY FUT"` | `"BANKNIFTY FUT"` |
| `"NIFTY 24500 CE"` | `"NIFTY OPTION"` |

Rules: strip month names (JAN–DEC / full names) and "FUT" → append " FUT"; strip "CE"/"PE" → append " OPTION"; strip bare numbers (strike prices).

---

## Key Functions Reference

| Function | Location (approx. line) | Description |
|---|---|---|
| `renderTableHead()` | ~2065 | Builds `<thead>`, today-column highlight, expiry-column highlight, day-name tooltip |
| `renderTableBody()` | ~2170 | Contract rows with L/S badges, drag handles, row actions |
| `renderDailyPnLChart()` | ~2700 | Chart.js daily P&L bar chart with per-contract tooltip breakdowns |
| `contractDisplayKey(name)` | ~3836 | Normalises contract name → canonical group key |
| `renderTargetsCard()` | ~3940 | Builds Targets This Month card rows, progress bars, summary |
| `syncCurrentValues()` | ~4123 | Syncs Current (Today) using Realized+Running formula (see above) |
| `computeStatusForCol(col, hidden)` | ~3953 | Returns realized+running total for a date column; used by history popup |
| `openStatusHistoryPopup()` | ~3967 | Computes and renders the daily history heatmap in the popup |
| `closeStatusHistoryPopup()` | ~4052 | Hides popup and removes Escape key listener |
| `calcProgress(mt, cv)` | ~4068 | Returns `{pctText, pctColor, barColor, barWidth}` |
| `updateTargetsSummary(keys)` | ~4085 | Rebuilds summary tiles in Targets card (Current Status tile is clickable) |
| `saveCurrentMonthData()` | ~3641 | Serialises active month to localStorage |
| `loadAllData()` | ~3655 | Reads all months from localStorage |
| `loadMonthData(month, year)` | ~4300 | Switches active month (saves current first) |
| `parseDateStr(dateStr, m, y)` | ~2021 | Parses "DD-MMM" → `Date` object |
| `isLastTuesdayOfMonth(d)` | ~2031 | Returns true if `d` is the last Tuesday (monthly expiry) |
| `showDayTooltip(e, dayName)` | ~2045 | Shows fixed-position day-name tooltip on th hover |
| `hideDayTooltip()` | ~2057 | Hides the day-name tooltip |
| `addCalculatedRow(label, type)` | ~3360 | Adds UnRealized / Net Realized editable rows |
| `addStandingRow()` | ~3380 | Adds derived Standing AT row (unrealized + realized, not stored) |
| `toggleCard(id)` | ~4130 | Accordion expand/collapse; triggers `renderChart` delay for analytics |
| `showNotification(title, msg, type)` | ~1665 | Toast notification (type: 'success' \| 'warning' \| 'info' \| 'error') |

---

## UI Sections — Card Accordion

All 4 sections use the `.card-section` accordion pattern:

| Card ID | Default state | Title |
|---|---|---|
| `card-table` | Expanded | ◢ Contracts |
| `card-targets` | Collapsed | ◢ Targets — {Month} {Year} |
| `card-notes` | Collapsed | ◢ Trading Notes |
| `card-analytics` | Collapsed | ◢ Portfolio Analytics |

`toggleCard(id)` flips `card-collapsed` class and updates `aria-expanded`. Analytics card additionally triggers `setTimeout(renderChart, 310)` to allow CSS transition to complete before Chart.js renders.

---

## Targets Card — Sync Current Values Feature

The **⟳ Sync** button in the Targets card header calls `syncCurrentValues()`.

### Aggregation formula

A contract group's **Current (Today)** value is composed of two parts:

```
Current (Today) = Realized Component + Running Component
```

| Component | Source | Why |
|---|---|---|
| **Realized** | SUM of ALL `bold` cells across **every** date column in the month | Bold = a closed/booked P&L event. Multiple trade closings during the same month each produce a separate bold cell and must all be accumulated. |
| **Running** | SUM of **non-bold** cells in **today's column only** | Non-bold today = current open-position running (unrealized) P&L. |

This correctly handles multiple re-entry / re-exit cycles during the same month:

```
Example — NIFTY FUT group on 17-Jun:
  09-Jun  NIFTY June  bold   +10,000   ← realized (exit 1)
  16-Jun  NIFTY June  bold    +5,000   ← realized (exit 2 after re-entry)
  15-Jun  NIFTY July  bold    -3,000   ← realized (different expiry)
  17-Jun  NIFTY June  non-bold -20,000 ← running open position

  Realized  = 10,000 + 5,000 + (-3,000) = 12,000
  Running   = -20,000
  Current   = 12,000 + (-20,000)        = -8,000
```

### Sync logic steps

1. Confirm the active month is the current calendar month — today's column only exists in the current month view.
2. Locate today's date string (`"DD-MMM"`) in `portfolioData.dates` via `indexOf`.
3. Respect the **Include Hidden** toggle (same rule as `renderTargetsCard`).
4. For each visible contract row:
   - Iterate **all** columns: accumulate `cell.value` where `cell.bold === true` into `realizedSums[group]`.
   - Read today's column: accumulate `cell.value` where `cell.bold === false` into `runningSums[group]`.
5. Merge: `contractTargets[key].currentValue = realized + running` for every group that had any data.
6. Call `saveCurrentMonthData()` then `renderTargetsCard()` to persist and refresh.
7. Show a toast notification with the count of groups updated.

### Error handling

| Condition | Behaviour |
|---|---|
| Active month ≠ current month | Warning toast, no changes |
| Today's column not in `dates` | Warning toast, no changes |
| All cells empty | Info toast noting no data found |

---

## Current Status History Popup

Clicking the **Current Status** tile in the Targets summary row opens a glassmorphism popup showing daily historical values for the active month.

### Data source

Values are **computed retroactively** from existing `cells` data — no separate history array is stored and no migration is needed.

For each past date column D the formula mirrors the Sync button:
```
Status at D = SUM(bold cells in cols 0→D) + SUM(non-bold cells in col D)
```
This gives a cumulative realized P&L up to date D, plus the open-position running P&L on that date.

### Scope rules

| Condition | Dates shown |
|---|---|
| Active month = current month | Only dates **before today** |
| Past month | All date columns |
| Future month | No dates (empty state message) |
| Include Hidden toggle | Mirrors the Targets card toggle — same contracts |
| Column with no cell data | Hidden (not shown) |

### Heatmap coloring

Cell background intensity is scaled proportionally to each day's absolute value relative to the month's peak:
- Positive: green tint `rgba(0,255,136, 0.06–0.35)`, value text `#00ff88`
- Negative: red tint `rgba(255,51,102, 0.06–0.35)`, value text `#ff3366`
- Zero: neutral gray, muted text

Text colour is always the bright accent colour on the dark background — readability is never sacrificed for visual intensity.

### Close behaviour

✕ button, clicking the backdrop, or pressing **Escape** all close the popup. The Escape `keydown` listener is attached on open and removed on close (no listener leaks).

### Key elements

| Element ID | Purpose |
|---|---|
| `status-history-backdrop` | Fixed-position glassmorphism overlay (`backdrop-filter: blur`) |
| `status-history-panel` | The content card |
| `status-history-body` | Populated by `openStatusHistoryPopup()` with `.sh-grid` |

---

## `computeStatusForCol(colIndex, includeHidden)`

Shared helper used by both the history popup and (indirectly) the Sync button logic.

```js
// Realized: sum of all bold cells in columns 0 → colIndex
// Running:  sum of non-bold cells in colIndex only
// Returns null if no data exists for any visible row in this column range.
```

---

## Table Column Highlights

| CSS class | Colour | Applied to |
|---|---|---|
| `.today-column` | Green tint | Both `th` and `td` of today's date |
| `.expiry-column` | Light yellow / dark green text | `th` only — last Tuesday of the month (monthly expiry) |
| Custom column | Gold border (inline style) | `th` only — columns beyond natural month days |

---

## Date Header Tooltip

Day-name tooltip uses a JS-driven `<div id="day-name-tooltip">` with `position: fixed` (not CSS `::after`, which is clipped by sticky/overflow parents). Triggered by `mouseenter`/`mouseleave` on each date `th`.

---

## Safe Change Checklist

Before modifying any section, verify:

- **Analytics**: all three views agree — Standing AT, By Contract, Daily P&L.
- **Table rendering**: contract rows, calculated rows, hidden items, today-column highlight all render correctly.
- **Persistence**: save/load works across month navigation and browser reload.
- **Sets**: `hiddenRows` and `hiddenCols` are always `Set` at runtime, arrays in storage.
- **Sync**: any new state fields must be included in `saveCurrentMonthData()` spread and restored with fallback `|| {}` in `loadMonthData()`.
- **Cell keys**: column deletion and row reordering require reindexing of all `cells` keys and hidden-index sets.
