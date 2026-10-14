# Boston Chair Count 💈

Track a month of cutting hair in Boston against what the trip costs. Log each day's cuts, earnings, food, and transport; the app shows how far you are from breaking even and what it takes to get there.

**Live app:** https://zagonzai.github.io/Air-Barber/

## Features
- Break-even tracker with a barber-pole progress bar, a daily log, and a money-in vs. money-out chart
- Installs to your phone's home screen and works with no signal
- Backup: export to CSV, download a JSON backup, restore from a backup

Data is saved on your phone only (browser storage). Download a backup now and then.

## Install on your phone
- **iPhone:** open the live link in **Safari** → tap **Share** → **Add to Home Screen** → **Add**.
- **Android:** open the live link in **Chrome** → tap **Install app** (or ⋮ menu → **Add to Home screen** / **Install app**).

Open it once while you have signal; after that it works offline.

## Run it locally
Open `index.html` in a browser. No install or build step. (The offline/installable parts need it served over http, e.g. `python3 -m http.server`.)

## Deploy
GitHub Pages: Settings → Pages → Source **Deploy from a branch** → branch `main`, folder `/ (root)`.

When you change app files, bump `CACHE` in `sw.js` so installed copies update.
