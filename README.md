# RC Buffet

Personal market desk — live watchlist, accounts, alerts and the morning brief,
installed to the iPhone home screen. No API keys, no paid services, no app store.

**This repo is deliberately flat.** Every file sits at the top level. The only
folder is `.github/workflows/`, which GitHub requires for the scheduled data
refresh. Do not move files into subfolders — the app looks for them at the root.

## Files

| File | What it does |
|---|---|
| `index.html` | The entire app — UI, tabs, alert engine, portfolio math |
| `sw.js` | Service worker: caches the app so it opens instantly and works offline |
| `manifest.webmanifest` | Makes it installable with a real home-screen icon |
| `fetch_data.py` | Pulls quotes and news. Runs on GitHub's servers, not your phone |
| `quotes.json` / `news.json` | Written by the workflow. Read by the app |
| `icon-*.png` | App icons |
| `.github/workflows/update-data.yml` | The schedule that runs `fetch_data.py` |

## How it works

Browsers block direct calls to market data providers (CORS), and keyless
providers that allow it are unreliable. So the fetching happens on **GitHub's
servers**: a scheduled Action runs `fetch_data.py` every 10 minutes during
market hours and commits `quotes.json` / `news.json` back here. GitHub Pages
serves the app and that data; your phone reads it.

The app also tries a live price call through a public proxy each time it opens.
Badge reads **LIVE** when that works, **SNAPSHOT · 4 min ago** when it doesn't —
so you always know how fresh the number in front of you is.

**Your share counts never leave your phone.** They live in browser local storage,
not in this repo. That's why this repo can safely be public.

## Setup

1. **Pages** — Settings → Pages → Source *Deploy from a branch* → `main` / `(root)` → Save.
   Your URL appears at the top after a minute.
2. **Permissions** — Settings → Actions → General → Workflow permissions →
   **Read and write permissions** → Save.
3. **First data** — Actions tab → *Update RC Buffet data* → Run workflow.
4. **Install** — open the Pages URL in **Safari** on iPhone → Share → Add to Home Screen.
   (Chrome cannot do this on iOS. On Android: Chrome → menu → Install app.)

Then open the **Holdings** tab and enter your share counts. Until you do, totals
show `—`.

## Alert thresholds

Single-day move: broad ETFs **2%** · sector/thematic **4%** · single names **5%** ·
LLY **5%** · CBRS and HIVE **8%**.

Slower: **8%** (funds) or **12%** (single names) below the 30-day high, or up
**10%** over five sessions.

Each alert states the ticker, size of the move, your position value in dollars,
the top related headline, and whether it looks company-specific or broad-market.
That last call is a heuristic — it compares the move against VOO — not a verified
attribution.

Thresholds live in the constants near the top of the `<script>` block in
`index.html` (`BROAD_ETFS`, `THEMATIC`, `DAY_OVERRIDE`, `FIVE_SESSION_RUN`).

## Changing tickers

Two places, and they must match: the `ACCOUNTS` object in `index.html`, and
`HOLDINGS_TICKERS` / `WATCH_ONLY` in `fetch_data.py`.

## Limitations

- Not tick-by-tick. Scheduled Actions run every 10 minutes and can be delayed a
  few more when GitHub is busy. Monitoring dashboard, not a trading terminal.
- Free public data breaks sometimes. Yahoo is primary, Stooq is the fallback
  (end-of-day only — those cards show an `EOD` badge). If everything fails, the
  last good data stays up with an honest timestamp.
- Mutual funds (FSPGX, FSELX, FZROX, FGRIX) only price once daily. That's the
  funds, not the app.
- Not financial advice. Buy Ideas is a research checklist.

## Troubleshooting

**"No market data yet"** — the Action hasn't succeeded. Check the Actions tab;
the usual cause is step 2 above.

**Old version keeps loading after a change** — the service worker cached it.
Bump `SHELL = "rcb-shell-v4"` to `v5` in `sw.js`.

**Share counts vanished** — iOS clears local storage for web apps unused ~7 days.
Use **Export holdings** in the Holdings tab now and then for a JSON backup.
