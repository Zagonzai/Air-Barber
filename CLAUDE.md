# Boston Chair Count

A mobile-first tracker for a month-long barbering trip to Boston. The one job of the app: show clearly whether the money made cutting hair covers everything the trip costs — money already spent plus money still to be spent.

## What it tracks
- **Trip setup:** first/last day, flight, lodging per night (owner's saved value is $45/night, negotiable — staying at a friend's Airbnb; not a default), nights (auto from dates, overridable), chair rent per week, weeks (auto = ceil(trip days / 7), overridable), other one-time costs (supplies etc.), estimated food and transport per day, optional profit goal (take-home after every cost).
- **Tap counter:** big "+1 Cut" button under the hero for counting cuts live during the day (see Current state).
- **Daily log:** date, optional in/out times, number of cuts, money made, transport spent, food spent. One entry per date; saving an existing date updates it.

## Core math (in `analyze()` in index.html)
- Fixed costs = flight + (lodging rate × nights) + (chair rent per week × weeks) + other
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
- Day entries look like `{cuts, made, transport, food, inAt, outAt}` with times as `"HH:MM"` or `""`. Older entries without `inAt`/`outAt` (and old states without `profitGoal`, `chairRent`, `weeks`) load fine.
- **Draft days:** the tap counter creates today's entry as `{cuts, draft: true}`. `analyze()` skips drafts entirely (not in totals, averages, chart, or CSV rows; a draft today still counts as a remaining day for still-to-spend), so a half-finished day doesn't move the math. Saving the day in Log a day writes a normal entry (no `draft`). Days list shows drafts as "in progress". Restore keeps drafts.
- **Tap counter** (`bumpCuts()` / `renderCounter()`): "+1 Cut" adds to today's entry (creating a draft if none) and saves immediately; "−1" can't go below 0 and deletes a draft that reaches 0. Uses the live date (`today()`), refreshes on `visibilitychange`, updates the Log a day Cuts field when it's showing today, bump/pulse animation + `navigator.vibrate`. Note shown when today is outside the trip dates.
- Trip costs grid order: first/last day, flight/other, lodging per night/nights, chair rent per week/weeks, food/transport estimates, profit goal. Where you stand shows "Chair rent (N weeks × $rate)" when rent is set.
- Hero: barber-pole break-even bar, plus a slim green goal bar (with a marker at the break-even point) when a profit goal is set. Chart draws the break-even line (grey dashed) and the goal line (green dotted).
- **New users start blank:** `defaults()` has every cost field and both trip dates empty (all inputs show a "0" placeholder). Until both dates are set the hero reads "Let's get started / Set your trip dates and costs to get started." and Trip costs stays open (also stays open after saving if dates are still missing). Saved data is merged over the blank defaults untouched; a saved state or backup with no `lodgingRate` key at all (pre-dates this change) keeps the old $45 (`LEGACY_LODGING_RATE`).
- Data persists only in `localStorage` (key `boston-cut-tracker-v1`). No account sync; the old claude.ai sync code was removed. Header reads "Saved on this phone only".
- All paths are relative so it works under the GitHub Pages repo subpath (`/Air-Barber/`).
- Installable on iPhone (Safari → Share → Add to Home Screen) and Android (Chrome → Install app). Works offline after the first load.
- **Backup section** (bottom of the page):
  - Export CSV — one row per logged day (date, in at, out at, hours, cuts, money made, transport, food, day net), a Total row, then a "Trip summary" block (dates, costs, fixed costs, earned, spent, still to spend, break-even goal, gap, profit goal, goal target, hours, $/hour, cuts/hour).
  - Download backup — the full `state` object as JSON.
  - Restore backup — validates/sanitizes the JSON (`cleanBackup()`), confirms, then replaces `state`. Missing fields (times, profit goal) default to blank.
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
