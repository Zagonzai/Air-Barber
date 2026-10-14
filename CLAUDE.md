# Boston Chair Count

A mobile-first tracker for a month-long barbering trip to Boston. The one job of the app: show clearly whether the money made cutting hair covers everything the trip costs — money already spent plus money still to be spent.

## What it tracks
- **Trip setup:** first/last day, flight, lodging per night (currently $45/night, negotiable — staying at a friend's Airbnb), nights (auto from dates, overridable), other one-time costs (chair rent, supplies), estimated food and transport per day.
- **Daily log:** date, number of cuts, money made, transport spent, food spent. One entry per date; saving an existing date updates it.

## Core math (in `analyze()` in index.html)
- Fixed costs = flight + (lodging rate × nights) + other
- Spent so far = fixed + logged food + logged transport
- Still to spend = remaining unlogged trip days (today onward) × (estimated daily food + transport; falls back to logged averages if estimates are blank)
- Break-even goal = spent so far + still to spend
- Gap = goal − earned. Also derived: avg per cut, avg per day, cuts needed to break even, $ and cuts needed per remaining day, projected end-of-trip net at current pace, savings per $5/night lodging reduction.

## Current state
- Single self-contained `index.html` (vanilla HTML/CSS/JS, no build step). Fonts from Google Fonts (Big Shoulders Display + Figtree).
- Data persists in `localStorage` (key `boston-cut-tracker-v1`). The original claude.ai version also synced via a `window.claude` db capability; that code path is guarded and simply does nothing outside claude.ai — it can be removed.
- Design: barber-pole progress bar (red/white/blue stripes) is the signature element; light and dark themes via CSS tokens on `:root`.

## Owner notes
- Owner communicates mostly by voice transcription — interpret requests generously.
- Keep it mobile-first; it's used on a phone at the end of each day.

## Ideas not yet built
- Cross-device sync / real backend
- Export to CSV
- Installable PWA (manifest + service worker) for home-screen use
