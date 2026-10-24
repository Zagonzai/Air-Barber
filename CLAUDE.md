# Boston Chair Count

A mobile-first tracker for a month-long barbering trip to Boston. The one job of the app: show clearly whether the money made cutting hair covers everything the trip costs — money already spent plus money still to be spent.

## What it tracks
- **Trip setup:** first/last day, flight, lodging per night (owner's saved value is $45/night, negotiable — staying at a friend's Airbnb; not a default), lodging nights calendar (each night Staying / Away / Custom cost), nights (auto = staying nights, overridable), chair rent per week, weeks (auto = ceil(trip days / 7), overridable), other one-time costs (supplies etc.), estimated food and transport per day, optional profit goal (take-home after every cost).
- **Tap counter:** big "+1 Cut" button under the hero; each tap asks what the client paid and records the cut with its time and amount (see Current state).
- **Daily log:** date, optional in/out times, number of cuts, money made, transport spent, food spent. One entry per date; saving an existing date updates it.

## Core math (in `analyze()` in index.html)
- Fixed costs = flight + lodging total + (chair rent per week × weeks) + other
- Lodging total = (nights × lodging rate) + sum of custom-night costs. Nights = the staying count from the calendar (`nightStats()`) unless a number is typed in Nights. Away nights add nothing. With no marks every night is staying, so totals match the pre-calendar math exactly. (No rate set → legacy flat `lodging` + custom costs.) The "$5/night off saves" row uses nights (staying count).
- Spent so far = fixed + logged food + logged transport
- Still to spend = remaining unlogged trip days (today onward) × (estimated daily food + transport; falls back to logged averages if estimates are blank)
- Break-even goal = spent so far + still to spend
- Gap = goal − earned. Also derived: avg per cut, avg per day, cuts needed to break even, $ and cuts needed per remaining day, projected end-of-trip net at current pace, savings per $5/night lodging reduction.
- Profit goal (optional, hidden everywhere when blank or 0): goal target = break-even goal + profit goal; goal gap = target − earned. Derived: left to reach goal, $ per remaining day, cuts needed, hours needed (at $/hr), projected profit (= projected end-of-trip net) vs. goal → "On track" / "Behind by $X". The break-even math is unchanged.
- Hours: `dayHours()` = out − in; if out < in it's a shift past midnight (+24h); equal or missing times → null. Hourly stats (total hours, avg hours/day, $/hour, cuts/hour) use only days that have both times.

## Files
- `index.html` — the whole app (vanilla HTML/CSS/JS, no build step). Fonts from Google Fonts (Big Shoulders Display + Figtree).
- `manifest.json` — PWA manifest ("Boston Chair Count" / "Chair Count", standalone, navy `#13213B`).
- `sw.js` — service worker. Precaches the app files, icons, and the Google Fonts CSS + font files; serves cache-first and refreshes the cache in the background. **Bump `CACHE` in `sw.js` whenever app files change** so installed copies pick up the new version (it shows on the next launch after the update downloads).
- `icons/` — barber-pole icons: `icon-192.png`, `icon-512.png` (also used as maskable), `apple-touch-icon.png` (180px).
- `.nojekyll` — tells GitHub Pages to serve files as-is.

## Current state
- Day entries look like `{cuts, made, transport, food, inAt, outAt}` with times as `"HH:MM"` or `""`, plus optional `cutLog: [{time:"HH:MM", amount:number|null}]` (one per tapped cut; `null` = skipped/no amount) and `madeManual: true`. Older entries without `inAt`/`outAt`/`cutLog` (and old states without `profitGoal`, `chairRent`, `weeks`) load and calculate exactly as before.
- **Per-cut tally rules:** `cuts` and `made` stay the source of truth for all math. Tally changes go through `changeTally()`, which applies a delta: add cut → cuts+1, made+amount; fix amount → made += new−old; delete/−1 → cuts−1, made−amount. If `madeManual` is set, taps change `cuts` but never `made`. In Log a day, Cuts and Money made are prefilled from the entry; on save, a day with a `cutLog` becomes `madeManual` when the typed Money made differs from the loaded value (or, if already manual, from the tally sum); clearing Money made resets it to the tally sum and drops `madeManual`. `cutLog` is preserved when the day is saved. −1 on a day with no `cutLog` (counted before per-cut entries) just lowers the count.
- **Draft days:** the tap counter creates today's entry as `{cuts, made, cutLog, draft: true}`. `analyze()` skips drafts entirely (not in totals, averages, chart, or CSV rows; a draft today still counts as a remaining day for still-to-spend), so a half-finished day doesn't move the math. Saving the day in Log a day writes a normal entry (no `draft`). Days list shows drafts as "in progress". Restore keeps drafts.
- **Tap counter** (`renderCounter()`, `addCut()` / `removeCut()` / `setCutAmount()` → `changeTally()`): card shows "Today N cuts · $X" (X = the day's `made`), a "−1" button (removes the most recent cut and its amount; can't go below 0; deletes a draft that empties), a full-width "+1 Cut" button, and a collapsible "Today's cuts (N)" list (newest first, time + amount; tap a row to fix the amount or delete it). Notes: outside trip dates; draft reminder; "edited by hand" when `madeManual`. Uses the live date (`today()`), refreshes on `visibilitychange`, keeps Log a day's Cuts/Money made in step when it shows today. Save gives bump/pulse animation + `navigator.vibrate`.
- **Price sheet** (`openSheet()` / `commitSheet()`): "+1 Cut" opens a bottom sheet "How much did they pay?" with up to 4 quick-price buttons (`quickPrices()`: most frequent amounts across all `cutLog`s, ties → most recent, shown low→high; hint text until any exist), a large decimal amount field focused synchronously inside the tap (so iOS opens the keypad), Save (Enter works; blank shows an error), "Skip — no amount" (saves `amount: null`), Cancel/Escape. Edit mode (from the list) adds "Delete cut". Rapid-tap guards: +1 ignored while the sheet is open and for 400ms after it closes; one commit per opening; backdrop taps close it only after 500ms; mousedown in the sheet is prevented so focus/keypad stays put. `fitSheet()` lifts the sheet above the iOS keyboard via `visualViewport`.
- Counter card layout: count + money on the left and −1 on the right in the top row; "+1 Cut" spans the full width below.
- **Lodging nights calendar** (`renderCal()`, under Lodging per night / Nights): every trip night from the first day through the night before the last day (`tripNights()`), one 7-column month grid per month (Sunday first). Stored in `state.nightMarks` = `{ "YYYY-MM-DD": {s:"away"} | {s:"custom", cost:"<n>"|""} }`; unmarked = staying. Tap cycles Staying → Away → Custom (opens the cost field, focused) → Staying; the first tap on an unselected custom night just opens its cost, the next tap cycles it back to staying. "Mark range away": tap first night, then last (either order) → all set away. Summary line "N nights staying · N away · N custom · $X total" (notes when a typed Nights is used). Calendar edits go into `nightDraft` and are saved with "Save trip costs" (like the other fields); the calendar redraws live from the date/rate/nights inputs. On save, marks outside the new trip dates are dropped; ones still inside are kept. Where you stand: "Lodging (N nights × $rate, N away, N custom)". CSV summary adds nights staying/away, custom nights list, custom nights total. Restore sanitizes `nightMarks`; old backups without it → all staying.
- Trip costs grid order: first/last day, flight/other, lodging per night/nights, chair rent per week/weeks, food/transport estimates, profit goal. Where you stand shows "Chair rent (N weeks × $rate)" when rent is set.
- Hero: barber-pole break-even bar, plus a slim green goal bar (with a marker at the break-even point) when a profit goal is set. Chart draws the break-even line (grey dashed) and the goal line (green dotted).
- **New users start blank:** `defaults()` has every cost field and both trip dates empty (all inputs show a "0" placeholder). Until both dates are set the hero reads "Let's get started / Set your trip dates and costs to get started." and Trip costs stays open (also stays open after saving if dates are still missing). Saved data is merged over the blank defaults untouched; a saved state or backup with no `lodgingRate` key at all (pre-dates this change) keeps the old $45 (`LEGACY_LODGING_RATE`).
- Data persists only in `localStorage` (key `boston-cut-tracker-v1`). No account sync; the old claude.ai sync code was removed. Header reads "Saved on this phone only".
- All paths are relative so it works under the GitHub Pages repo subpath (`/Air-Barber/`).
- Installable on iPhone (Safari → Share → Add to Home Screen) and Android (Chrome → Install app). Works offline after the first load.
- **Backup section** (bottom of the page):
  - Export CSV — exports two files (one share sheet / two downloads): the days CSV and `chair-count-cuts-DATE.csv` with one row per tapped cut (date, time, amount; drafts included). Days CSV: one row per logged day (date, in at, out at, hours, cuts, money made, transport, food, day net), a Total row, then a "Trip summary" block (dates, costs, fixed costs, earned, spent, still to spend, break-even goal, gap, profit goal, goal target, hours, $/hour, cuts/hour).
  - Download backup — the full `state` object as JSON.
  - Restore backup — validates/sanitizes the JSON (`cleanBackup()`), confirms, then replaces `state`. Keeps `cutLog` (sanitized), `madeManual`, and `nightMarks`. Missing fields (times, profit goal, per-cut entries) default to blank.
  - On touch devices files go through the share sheet (`navigator.share`), otherwise a normal download.
- Layout: every element is `box-sizing:border-box` set directly (not inherited — `<details>` content doesn't inherit it, which caused inputs to overflow on iPhone). Form grids use `minmax(0,1fr)` columns with `min-width:0` children; inputs are `display:block; width:100%; min-width:0`; date/time inputs use `appearance:none` and left-aligned values. `[hidden]` is forced to `display:none`.
- Design: barber-pole progress bar (red/white/blue stripes) is the signature element; light and dark themes via CSS tokens on `:root`.

## Hosting
- GitHub repo `Zagonzai/Air-Barber`, default branch `main`. Do all work on `main`.
- GitHub Pages: deploy from `main`, root folder (must be switched on in repo Settings → Pages). Live at https://zagonzai.github.io/Air-Barber/

## Owner notes
- Owner communicates mostly by voice transcription — interpret requests generously.
- Keep it mobile-first; it's used on a phone at the end of each day.

## Ideas not yet built
- Cross-device sync / real backend
