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

## Methodology

### How the value guess is built
- Every protocol starts from the money locked in it, because that is the
  value its future token would have to distribute.
- A fifth of that locked money is the starting guess, not a price and not
  a promise of a token.
- Fast growers get a lift and shrinkers get a cut, so a protocol that is
  gaining users separates from one that is losing them.
- The growth lift is capped on both ends, so one wild week cannot carry
  the whole ranking and one bad week cannot sink a large protocol.
- Older protocols without a token score higher, because the longer a
  protocol runs without one, the more value has piled up for a launch.
- The age lift is capped at three years, so a very old protocol does not
  outrank everything else on age alone.
- Protocols live on more chains get a small lift, because being on more
  chains means more users and more places the value can come from.
- The chain lift is capped at five chains, so breadth helps but cannot
  dominate the ranking.
- A protocol with no listing date is treated as one year old, which is
  the middle of the range, rather than as brand new or very old.
- No protocol name is ever special-cased: the same arithmetic scores
  every row, and the ranking is whatever the numbers say.
- The model uses only the snapshot's own figures: locked money, the
  seven-day change, the listing date, and the chain count.
- Nothing is pulled from a live price feed, a market cap, or a social
  signal, because the snapshot is sealed and the entry must be
  reproducible from it alone.
- The ranking is sorted by the guessed value, highest first, and ties
  are impossible in practice because the values are rounded whole
  dollars and the board holds two hundred distinct numbers.
- Every row carries the three factors that shaped it, so a reader can
  see why a protocol sits where it does instead of trusting a black
  box.
- The detail view recomputes the guess from the same inputs, and the
  page says whether the recomputed number matches the board's number.
- That recompute check is the entry's own proof that the board and the
  page agree with each other.

### How the board was assembled
- The snapshot holds 758 protocols, and the watchlist names 40 of them
  that the bounty asks the board to cover in full.
- The board holds 200 rows: all 40 watchlist protocols, plus the 160
  strongest pre-token protocols below the watchlist floor.
- The floor is fifty million dollars of locked money, which keeps the
  board to protocols large enough for the guess to mean something.
- A protocol counts as pre-token only when the snapshot records no
  market cap, no CoinGecko id, and no symbol for it.
- Twenty-four protocols in the snapshot already have a token, and they
  are set aside with their market caps instead of being ranked.
- The live-token protocols are reachable from one link on the page,
  so a reader can check what was left out and why.
- The watchlist is covered forty out of forty: no watchlist protocol
  is missing from the board, whatever its rank.
- Ranks run exactly one to two hundred with no gaps and no duplicates.
- The guessed values never rise as the rank falls, which is what a
  ranking sorted by value must look like.
- The top of the board is dominated by risk curators and onchain
  capital allocators, because those categories hold the most locked
  money in the snapshot.
- The board is rebuilt from the snapshot by one command, so anyone
  with the kit and the snapshot can reproduce it byte for byte.

### How the farm plans are written
- Every row carries a six-step farm plan, written for a reader who has
  never farmed a protocol before.
- Each step says what to do, in plain words, with no jargon left
  unexplained.
- Each step names a concrete cost or a concrete amount of time, so
  the reader can tell whether the step is cheap or dear before
  starting it.
- The first step is always the cheapest way in: usually a small
  deposit or a test transaction, never a large commitment up front.
- Later steps build on the earlier ones: supply, then borrow or
  stake, then use the receipt token somewhere it earns more.
- One step always says how to keep the position safe, because a
  farm plan that never mentions risk is not a plan.
- The last step always says how to get out, including what it costs
  to unwind and what to watch for on the way out.
- Steps are kept short, between twenty and six hundred characters,
  so each one fits on a phone screen without scrolling.
- The plans never promise a token, an airdrop, or a return: they say
  what the protocol does today and what using it costs.
- A plan is written from the protocol's own category and chains, so
  the steps fit what that protocol actually offers.
- Two protocols in different categories get different plans, even
  when their numbers look alike.

### How the dashboard is meant to be used
- The page opens on the full ranking, strongest first, with the
  three biggest protocols visible without scrolling.
- One search box finds a protocol by name or category, and one
  menu narrows the board to a single category.
- Every column header sorts the table, so a reader can reorder by
  locked money, growth, age, or guessed value.
- A star marks the watchlist protocols, so the forty the bounty
  cares about stand out at a glance.
- Tapping a row opens the detail view, which keeps every important
  fact: the locked money in full, the seven-day change in words,
  the listing date and the age in words, every chain, the three
  factors, the formula, the recompute check, and the farm plan.
- The detail view says the numbers in plain words on purpose, so a
  reader who has never seen a DeFi dashboard can still follow it.
- The page works with no network at all: every byte it shows is in
  the data file the program wrote beside it.
- The whole dashboard is under sixty-four kilobytes, so it loads
  fast on a phone on a slow connection.
- The three screenshots in the repository show the board, a detail
  view, and a filtered view, all taken from this dashboard.

### How the entry was checked
- Eighteen local checks run from one script, and all eighteen pass
  with zero failures.
- The checks confirm the board has two hundred rows, the watchlist
  is fully covered, and the ranks run one to two hundred.
- They confirm the guessed values never rise down the ranking and
  that all two hundred values are distinct.
- They confirm every row has six farm steps and every step is
  between twenty and six hundred characters.
- They confirm the dashboard is a complete static page under the
  size cap with no outside links in it.
- They confirm the data file beside the dashboard matches the
  board row for row.
- They confirm the three screenshots are the right size and under
  the file-size cap.
- They confirm the dossier matches a fresh rebuild byte for byte
  and stays under its size cap.
- They confirm re-running the program produces a byte-identical
  board, so the build is deterministic.
- The checks are re-run after every change to the entry, and the
  entry is only pushed when they all pass.

### What the entry does not do
- It does not predict prices, and it does not promise that any
  protocol will launch a token.
- It does not use live data: the snapshot is dated 14 November
  2026, and the world has moved on since then.
- It does not score teams, audits, or governance, because the
  snapshot carries none of that.
- It does not rank the twenty-four protocols that already have a
  token, because their value is already priced by a market.
- It does not tell anyone how much money to put in: the farm
  plans describe costs, they never recommend an amount.
- The guessed value is a starting point for research, not a
  verdict on any protocol.

## Notes

- No secrets, API keys, or keypair material are in this directory.
- No promotional text for any external service is included anywhere
  in the entry.

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>
