---
name: stake-engine-game-development
description: Build, test, and review Stake Engine games with compliant static math files, frontend behavior, RGS integration, replay support, and approval readiness. Use for new Stake Engine games, math packages, PixiJS/Svelte frontends, RGS debugging, or approval preparation.
metadata:
  short-description: Build and validate Stake Engine games
---

# Stake Engine game development

Use the official documentation as the source of truth. The documentation is a live site, so browse `https://stake-engine.com/docs` before relying on a detail that may have changed. If a local Stake Engine skill conflicts with the current official docs, follow the official docs and call out the discrepancy.

## Operating model

Stake Engine games are stateless from the player's perspective: the RGS selects a precomputed outcome and the frontend renders its events. Keep game math deterministic and publishable as static files; do not put outcome selection or payout authority in browser code.

Before coding, establish the game concept, unique title, modes, base cost, feature rules, max win, target RTP, supported languages/currencies, replay behavior, and the output contract between math events and frontend book-event handlers.

Reject or redesign concepts that require jackpots, gamble features, continuation/state across bets, early cashout, copied/licensed third-party games, Stake/Kick branding, or child-like characters in gambling content. Treat originality, suitability, and IP clearance as hard gates.

## Recommended implementation shape

For math, start from the closest sample in the Math SDK or its template. Keep reusable behavior in `src/` and game-specific behavior in the game directory. A typical game contains:

```text
game/
├── library/                 # generated books, configs, forces, lookup tables
├── reels/
├── readme.txt
├── run.py
├── game_config.py
├── game_executables.py
├── game_calculations.py
├── game_events.py
├── game_override.py
└── gamestate.py
```

`GameConfig` defines symbols, board dimensions, win type, paytable, reels, special symbols, win cap, RTP, and `BetMode` objects. `run_spin()` should reset state, draw/evaluate the board, update the win manager, emit ordered events, run features when applicable, finalize the win, validate distribution criteria, and imprint the book. Seed simulations with the simulation number for reproducibility.

For frontend work, use the PixiJS/Svelte architecture when available. Set required Svelte contexts at the app entry point, use layout context for responsive sizing, XState for betting/resume/autobet states, and the event emitter to connect book-event handlers to components. Keep handlers small and sequential: complex events should be decomposed into atomic events that can be tested independently in Storybook.

## Math publication contract

For each mode, publish an `index.json` entry containing the mode name, cost multiplier, compressed events filename, and lookup-table filename. Lookup CSV rows are:

```text
simulation number, probability/weight, payout multiplier
```

Every event record must be a JSON object with at least `id`, `events`, and `payoutMultiplier`; game logic is currently stored as zstandard-compressed JSON Lines (`.jsonl.zst`). The payout multiplier in the CSV must exactly match the corresponding book.

Use distribution criteria such as zero win, base-game win, feature/free-game, and max-win to guarantee useful outcome coverage. Run small uncompressed simulations while debugging, then run production-scale simulations (normally at least 100,000 per mode) with analysis and optimization enabled. Inspect RTP, hit-rate, standard deviation, win-range gaps, non-zero weights, and repeated/over-dominant outcomes.

Current approval math gates from the official docs include:

- RTP 90.0%–96.7% for every mode, with no more than 0.5% variation between modes;
- a base mode at 1.0× cost and base standard deviation at least 0.6;
- maximum payout multiplier no greater than 500,000× and cost multiplier no greater than 2,000×;
- a non-zero win at least once per 50 spins;
- a viable bet template after exposure and cost limits are applied;
- no single events file over 4.2 GB and no mode over 10,000,000 events.

Do not hard-code a conflicting RTP rule from memory; verify the current approval page before finalizing math.

## RGS and frontend contract

On load, read `sessionID`, `lang`, `device`, and `rgs_url` from the game URL. Call `POST {rgs_url}/wallet/authenticate` before other wallet endpoints. Respect the returned balance, currency, `minBet`, `maxBet`, `stepBet`, default bet, bet levels, jurisdiction flags, and any active round. Use `/wallet/play` to start a round and `/wallet/end-round` when the round is complete; use `/wallet/balance` for refreshes and `/bet/event` for resumable in-progress state where required.

Amounts use six decimal places as integer values: 1 USD is `1000000`. Never hard-code the RGS URL. A game that only supports English must remain uncorrupted when other language values are passed.

The frontend must:

- display balance, bet amount, final win, game rules, paytable, RTP, max win, mode costs, special-symbol values, feature triggers, and a UI guide;
- expose every bet level returned by authentication and preserve the selected bet after refreshes;
- support mobile and mini-player layouts without distortion;
- include a sound-off option and map the spacebar to the bet button;
- require explicit confirmation for autoplay and modes costing more than 2×;
- load images and fonts from the Engine CDN/static package only, with no external runtime requests;
- use ordered animation/event playback and keep payout display synchronized with the final RGS multiplier.

## Replay mode

Replay is mandatory for new games. Detect `replay=true`; parse `game`, `version`, `mode`, `event`, `rgs_url`, and optional currency/amount/language/device/social values; call `GET {rgs_url}/bet/replay/{game}/{version}/{mode}/{event}`; and render the returned state without session authentication or betting. Show loading, Play, Play Again, final payout, and error states. Disable normal bet controls and prevent transition from replay into live play.

## Approval readiness

Before submission, test normal wins, losses, big wins, max wins, every mode, replay events, currencies, languages, mobile, mini-player, sound toggle, keyboard controls, refresh/resume, and network errors. Check the browser network console for errors or leaked game information.

Include the required rules/info disclaimer, covering malfunction voids wins and plays, connection/reload recovery, RTP being calculated over many plays, illustrative display, RGS-settled winnings, and the current Engine trademark/copyright notice. Use the current official template rather than an old copied year.

Use unique, original visual/audio assets. Prepare the game tile background, transparent foreground, and provider logo; keep background plus foreground under the documented combined size limit.

Approval is for a specific frontend and math version. Treat math, modes, and mechanics as final before submission because post-release changes are generally limited to minor visual fixes.

## Useful references

- [Official docs map and current requirements](references/official-docs.md)
- [Stake Engine documentation](https://stake-engine.com/docs)
- [RGS documentation](https://stake-engine.com/docs/rgs)
- [Math documentation](https://stake-engine.com/docs/math)
- [Frontend documentation](https://stake-engine.com/docs/front-end)
- [Approval guidelines](https://stake-engine.com/docs/approval-guidelines)
