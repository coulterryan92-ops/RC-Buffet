# RC Buffet

A personal market desk that lives on your phone's home screen. Live watchlist,
your three accounts, threshold alerts, the morning brief, and buy ideas — one app.

No API keys. No paid services. No app store.

---

## How it works (30 seconds)

Your phone can't call market data providers directly — browsers block it (CORS),
and the ones that don't block it want an API key. So the fetching happens on
**GitHub's servers** instead:

1. A GitHub Action wakes up every 10 minutes during market hours.
2. It pulls quotes, 30-day history and news headlines.
3. It commits them to `data/*.json` in this repo.
4. GitHub Pages serves the app + that data. Your phone reads it.

The app also tries a direct live price call through a public proxy each time you
open it. If that works, the badge at the top right says **LIVE**. If it doesn't,
it says **SNAPSHOT · 4 min ago** so you always know exactly how fresh the number
in front of you is.

**Your share counts never leave your phone.** They're stored in your browser's
local storage, not in this repo. That's why the repo can safely be public.

---

## Setup — about 10 minutes, once

### 1. Put the files in a repo

In **GitHub Desktop**: `File → New Repository…`

- **Name:** `rc-buffet`
- **Local path:** wherever you keep projects
- Leave the rest at defaults, click **Create Repository**

Then copy everything from this folder into the repository folder it just made —
`index.html`, `sw.js`, `manifest.webmanifest`, `README.md`, and the `data`,
`icons`, `scripts`, and `.github` folders. (`.github` starts with a dot, so it
may be hidden — on Mac press `Cmd + Shift + .` in Finder to show hidden files.)

Back in GitHub Desktop you'll see all the files listed. Type a summary like
`initial commit`, click **Commit to main**, then **Publish repository**.

> **Important: uncheck "Keep this code private."**
> Free GitHub Pages and free unlimited Actions minutes both require a public
> repo. Nothing personal is in here — only ticker symbols and public price data.
> Your share counts and account values stay on your phone.

### 2. Turn on GitHub Pages

On github.com, open your `rc-buffet` repo:

`Settings → Pages → Build and deployment`

- **Source:** Deploy from a branch
- **Branch:** `main`, folder `/ (root)`
- **Save**

Wait a minute, then refresh. Your URL appears at the top:

```
https://<your-username>.github.io/rc-buffet/
```

### 3. Let the robot write to the repo

`Settings → Actions → General → Workflow permissions`

- Select **Read and write permissions**
- **Save**

Without this the data refresh runs but can't commit what it fetched.

### 4. Fetch data for the first time

`Actions` tab → **Update RC Buffet data** (left sidebar) → **Run workflow** →
leave mode as `all` → **Run workflow**.

If GitHub shows a banner saying workflows are disabled, click the button to
enable them first. The run takes about a minute.

From then on it runs on its own — every 10 minutes for prices during market
hours, plus full news pulls before the open, midday, and after the close.

### 5. Add it to your home screen

On your iPhone, open the Pages URL **in Safari** (not Chrome — only Safari can
install web apps on iOS):

**Share button → Add to Home Screen → Add**

You now have an RC Buffet icon. It opens fullscreen with no browser chrome,
exactly like a native app.

On Android: open in Chrome → menu → **Install app**.

---

## Using it

| Tab | What's there |
|---|---|
| **All Accounts** | Watchlist, broad market, indexes, every position sorted by size |
| **Individual / Roth / Rollover** | Per-account cards; the header total switches with the tab |
| **Alerts** | Everything that crossed a threshold, with your dollar exposure and likely cause |
| **News & Macro** | Fired-alert banner, macro, watchlist headlines, FDA/peptide/telehealth, market buzz |
| **Buy Ideas** | Diversification gaps — what you *don't* own — plus a concentration check |
| **Holdings** | Enter your share counts here |

**First thing to do:** open **Holdings** and type in your share counts. Until you
do, the header shows `—` and alerts say "no share count entered."

**Hide $** in the header blurs every dollar figure so you can show someone the
dashboard without showing your balances. It's pure CSS and persists across tabs.

### Alert thresholds

Single-day move:

- Broad ETFs (VOO, SCHG, SCHD, QQQM, SPMO, MGK, VUG, FZROX, FSPGX, FGRIX) — **2%**
- Sector / thematic (SMH, FSELX, BOTZ, AIPO, WQTM) — **4%**
- Single names — **5%**
- LLY — **5%** · CBRS and HIVE — **8%**

Slower-moving:

- **8%** (funds) or **12%** (single names) below the 30-day high
- Up **10%** or more over the last five sessions

Every alert states the ticker, the size of the move, your position value in
dollars, the top related headline, and whether it looks company-specific or
broad-market. The broad-vs-specific call is a heuristic — it compares the move
against VOO — not a verified attribution.

To change any threshold, edit the constants near the top of the `<script>` block
in `index.html` (`BROAD_ETFS`, `THEMATIC`, `DAY_OVERRIDE`, `FIVE_SESSION_RUN`).

---

## Changing your tickers

Two places, and they need to match:

1. `index.html` — the `ACCOUNTS` object at the top of the script
2. `scripts/fetch_data.py` — `HOLDINGS_TICKERS` and `WATCH_ONLY`

Commit and push. GitHub Pages redeploys in under a minute.

---

## Honest limitations

- **It isn't tick-by-tick.** Scheduled GitHub Actions run every 10 minutes and
  can be delayed a few more minutes when GitHub is busy. The **LIVE** badge only
  appears when a public CORS proxy happens to answer. Treat this as a monitoring
  dashboard, not a trading terminal.
- **Free public data breaks sometimes.** If Yahoo changes an endpoint, the
  fetcher falls back to Stooq (end-of-day only — those cards show an `EOD`
  badge), and if everything fails the last good data stays on screen with an
  honest timestamp. It never shows you a stale price pretending to be current.
- **Mutual funds price once a day.** FSPGX, FSELX, FZROX and FGRIX will only ever
  move after the close. That's the funds, not the app.
- **Not financial advice.** Buy Ideas is a research checklist, nothing more.

---

## Troubleshooting

**Blank page / "No market data yet"** — the Action hasn't run successfully.
Check the Actions tab for a red X. The most common cause is step 3 (workflow
write permissions).

**Icon didn't appear on the home screen** — you used Chrome on iOS. Only Safari
can do this on iPhone.

**Prices look a day old** — normal outside market hours, and normal for the
mutual funds. Check the badge in the header for the real timestamp.

**Old version keeps loading after you push a change** — the service worker
cached it. Close the app fully (swipe up), reopen, and give it a few seconds.
To force it, bump `SHELL = "rcb-shell-v3"` to `v4` in `sw.js`.

**Share counts disappeared** — iOS clears local storage for web apps that go
unused for about 7 days. Use **Export holdings** in the Holdings tab
occasionally; it copies a JSON backup you can paste into **Import holdings**.
