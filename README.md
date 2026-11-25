# Token Radar — make.fun bounty token-radar-01

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/trash-station/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/trash-station/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

Build-only entry for account @trash-station. **Being submitted.**

## What this is

A Node 22 program (`main.ts`, standard library only) that reads the sealed
DefiLlama snapshot, scores pre-token protocols, and writes a ranked board
plus a static dashboard. The judge compares dashboards pairwise; this entry
aims to win on a stated, reproducible value model, concrete farm plans,
and a usable page.

## Model — radar-2

Every row is scored by one function of the snapshot's own figures, no
protocol name is special-cased:

```
implied = round( tvl_usd x 0.20
  x (1 + clamp(change_7d, -80, 150) / 100 x 0.75)
  x (0.55 + min(age_days, 1095) / 1095 x 0.45)
  x (1 + min(chains - 1, 4) x 0.03) )
```

- **TVL** — the base. Larger protocols have more value to distribute.
- **7-day growth** — weighted 0.75, so fast growers separate from shrinkers.
- **Age since listed_at** — longer with no token = more accrued value;
  unknown age uses 365 days.
- **Chain count** — breadth: up to +12% for protocols live on 5+ chains.
- Pre-token means no market cap, no CoinGecko id, no symbol on record.
  Live-token protocols are set aside with their market caps, never ranked.

## Files

- `main.ts` — entrypoint: reads `--input <snapshot.json>`, writes
  `board.json`, `data.js`, copies `dashboard/` + `screenshots/`, writes
  `dossier.json` via `vendor/makefun-board.ts`.
- `lib/model.ts` — the model, board builder, farm-plan generator.
- `dashboard/` — static page, no network: `index.html`, `app.css`,
  `app.js` (31,359 bytes total, under the 64 KB cap). Mixed design: the
  token table is the full old table (rank, name, TVL, 7-day change, age,
  implied value with its bar, token status — a star marks the watchlist —
  sortable on every column), while the filter stays the simple new one
  (one search box and one category menu). Tap any row and a detail view
  opens with everything that matters — TVL, 7-day change, listed date and
  age, all chains, the three model factors with the full formula, the
  recomputed value check, and the 6-step farm plan. A single link reaches
  the 24 live-token protocols, kept out of the ranking.
- `vendor/makefun-board.ts` — dossier library, unchanged from the kit.
- `screenshots/` — see the blocker note below.
- `out/` — local build output from the last run (board, data, dashboard,
  screenshots, dossier). Not part of the submission; the grader runs
  `main.ts` itself.

## How to run

```bash
cd ~/workspace/makefun/token-radar-entry
mkdir -p out
MAKEFUN_OUTPUT_DIR=out node main.ts --input ../token-radar-01-kit/input/snapshot.json
```

Requires Node 22.18+ (tested on Node 24.22.1). No installs, no network.

## Verification (local, 18/18 passed)

Run from `~/workspace/makefun`:

```bash
node check-entry.mjs
```

Checks: 200 board rows; all slugs in the snapshot; watchlist 40/40;
ranks exactly 1..200; implied values non-increasing, 200 distinct;
farm steps 6 per row, each 20–600 chars; dashboard has index.html +
css + js, 33,397 bytes, no http/https URLs; `data.js` rows match
`board.json` (model radar-2, 40 watchlist flags); 3 screenshots, PNG,
1440x900, each under 512 KiB; `dossier.json` (708,902 bytes) equals
`buildDossier` output byte-for-byte and is under 1.5 MiB; re-running
`main.ts` produces byte-identical `board.json`.

Top 10 by implied value:

1. steakhouse-financial — $469,103,931
2. gauntlet — $284,800,633
3. veda — $265,939,609
4. usdd — $175,534,083
5. upshift — $99,973,081
6. nonce-capital — $55,774,687
7. onre — $46,403,830
8. opentrade — $42,525,455
9. symbiotic — $42,233,626
10. rockawayx — $41,502,832

## What remains before submission (not done here)

1. **Screenshots — blocker.** No Chrome / Chromium / Firefox binary is
   installed on this machine, and no Playwright cache exists. The three
   screenshots in `screenshots/` are the kit starter's renders of the
   starter dashboard, not re-renders of this improved dashboard. Before
   submitting, open the built dashboard in a browser and re-take the
   three shots (board, a protocol detail view, a filtered view),
   1440x800 minimum short side, each under 512 KiB, then re-run
   `main.ts` so `dossier.json` is rebuilt from them.
2. **GitHub repository.** Create a public repo after the bounty opened
   (2026-11-14), copy this directory in, add `.makefun/owner.json`
   with the connected wallet, commit, push. Do not submit the starter
   unchanged — this build already differs (radar-2 model, farm plans,
   dashboard filters).
3. **Transcripts.** The submission must carry the agent sessions that
   built this entry (bundle via the kit CLI). This build was done in
   this session; bundle it before submitting.
4. **Submit.** Only after 1–3, run the kit's check, then submit with
   the kit CLI. Do not submit from this workspace without that review.

## Notes

- No secrets, API keys, or keypair material are in this directory.
- No promotional text for any external service is included anywhere
  in the entry.

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>
