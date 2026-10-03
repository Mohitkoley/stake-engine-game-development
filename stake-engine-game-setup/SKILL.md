---
name: stake-engine-game-setup
description: "Use this skill to scaffold a new Stake Engine game with a two-folder layout: a Python math package (the 'math' folder, built on the Stake Engine math-sdk) and a Phaser 4 frontend (the 'frontend' folder). The frontend's dev server emulates the RGS by reading the math folder's published output, so running it in VS Code uses real values from the math sims. Triggers on: set up / scaffold / create a new Stake Engine game, new slot game project, Phaser frontend for math-sdk, wire frontend to math output, mock RGS, dev RGS, play math books in the browser."
---

# Stake Engine — Game Setup (Math + Phaser 4 Frontend)

> Scaffolds a Stake Engine game as two side-by-side folders — a **math** package (Python, math-sdk) and a **frontend** (Phaser 4 + Vite + TypeScript). In dev, a Vite plugin emulates the RGS by reading the math folder's published `library/` output (index.json + lookup tables + books), so the frontend you run in VS Code is driven by **real simulation values**. In production the exact same frontend talks to the real RGS via the `rgs_url` query param — nothing changes.

**Related skills:** stake-engine-approval (and the per-area approval skills: stake-math-requirements, stake-rgs-requirements, stake-frontend-requirements, stake-game-disclaimer, stake-game-tile, stake-jurisdiction-requirements, stake-replay-requirements). Build to those guidelines from the start. For Phaser 4 specifics use the Phaser skills (scenes, sprites-and-images, input-keyboard-mouse-touch, tweens, text-and-bitmaptext, audio-and-sound, scale-and-responsive).

**Do not** copy from existing `monkeys-fe` or `DCC` folders — their compliance is unverified. Build fresh from this skill.

## Self-contained: works from an empty folder

This skill assumes **nothing** is pre-installed. Given only a folder containing `.claude/skills/`, it carries everything needed to build a complete game:
- **Part 0 — environment:** clone the math-sdk and set up the Python/Node toolchains → [references/environment-setup.md](references/environment-setup.md).
- **Part 1 — math:** scaffold the math package and run the pipeline → [references/math-scaffold.md](references/math-scaffold.md) + the contract in [references/math-contract.md](references/math-contract.md).
- **Part 2 — frontend:** the full Phaser 4 + Vite + dev-RGS source → [references/frontend-scaffold.md](references/frontend-scaffold.md).

**Boundary — skill vs. prompt:** this skill is the reusable *how* (engine, scaffolds, contract, approval rules). It is game-agnostic. **The specific game's design — its modes, mechanics, payout model, event types — comes from your prompt**, not from this skill. An empty folder + these skills can build *any* Stake game; your prompt names which one and supplies its design brief.

## Target Layout

```
<GameName>/
  math/        # Stake Engine math-sdk game package (Python). Produces library/publish_files/
  frontend/    # Phaser 4 + Vite + TS. Dev RGS reads ../math/library
```

The **math** folder is a normal math-sdk game (it runs inside the cloned `math-sdk/`). Two ways to wire it:
- Create the game at `math-sdk/games/<GameName>/` and point the frontend's `mathDir` at `math-sdk/games/<GameName>/library` (simplest — the SDK runs games from its own `games/` dir).
- Or keep a `<GameName>/math/` folder and symlink it into `math-sdk/games/`.

Either way the frontend only needs the path to the math game's `library/` directory.

## The Math ↔ Frontend Contract (this is the whole bridge)

The math run writes, under `<game>/library/`:
- `publish_files/index.json` — `{ "modes": [{ "name", "cost", "events": "books_<mode>.jsonl.zst", "weights": "lookUpTable_<mode>_0.csv" }] }`
- `publish_files/lookUpTable_<mode>_0.csv` — rows of `id,probabilityWeight,payoutMultiplier` (uint64)
- `publish_files/books_<mode>.jsonl.zst` — zstd-compressed JSON-lines, each `{ "id", "events": [...], "payoutMultiplier" }`
- `books/books_<mode>.json` — the **uncompressed** books array (what the dev RGS reads — no zstd needed)

Key facts the frontend relies on:
- **`payoutMultiplier` is an integer = multiplier × 100.** `1150` → 11.5×, `3980` → 39.8×. Win = `baseBet × payoutMultiplier / 100`.
- **Money is integer with 6 decimal places.** `1_000_000` = 1.0 unit; a $1 bet sends `"1000000"`.
- Outcome is chosen by **weighted-random pick** over the lookup table's `probabilityWeight` column; the win comes from the selected **book's** `payoutMultiplier` (authoritative).
- The frontend is **purely an animation layer**: it replays `round.state` (the book's `events[]`) and never computes payouts itself. This is a hard approval requirement — see stake-game-disclaimer.

Full details: [references/math-contract.md](references/math-contract.md).

## Procedure

0. **Set up the environment** (skip if `math-sdk/` already exists with a working `env/`). Follow [references/environment-setup.md](references/environment-setup.md): clone the math-sdk, bootstrap the Python venv (incl. the no-pip fallback), and install Node 18.18 + pnpm. This is what makes the empty-folder case work.

1. **Create the math game.** Follow [references/math-scaffold.md](references/math-scaffold.md): copy the SDK `template`/sample closest to your mechanic to `math-sdk/games/<GameName>/`, set `game_id` = folder name, define the bet modes + event stream + payout from **your prompt's design brief**. **For a non-reel / event-stream game** (watch a run resolve — no reels or paytable), use [references/custom-game-recipe.md](references/custom-game-recipe.md) instead: the minimal `run_spin`, an internals cheat-sheet, a ready-made custom weighter, and the dev-RGS book steps. Keep modes discrete (no sliders), stateless, no jackpot/gamble/cashout. Run `run.py` to produce `library/publish_files/` and the uncompressed `library/books/books_<mode>.json`. Hit RTP 90–98%, modes within 0.5%, obtainable max win (stake-math-requirements).

2. **Scaffold the frontend.** Create `<GameName>/frontend/` and write every file in [references/frontend-scaffold.md](references/frontend-scaffold.md) verbatim (adjusting names). It includes:
   - `vite/devRgs.ts` — the Vite plugin that emulates `/wallet/authenticate`, `/wallet/play`, `/wallet/end-round`, `/wallet/balance`, `/bet/event` from the math `library/`.
   - `src/rgs/` — the production RGS client (calls `rgs_url`), money helpers, types. Identical code path dev vs prod.
   - `src/game/` — Phaser 4 scenes, a HUD with the required UI (bet selector honoring RGS bet levels, balance, spin, sound toggle, info/disclaimer, spacebar→spin, autoplay-with-confirmation), and the **event replay** engine that turns book events into animations.

3. **Point the dev RGS at the math output.** In `vite.config.ts`, set `mathDir` to the math game's `library` folder (e.g. `../../math-sdk/games/<GameName>/library`).

4. **Run it.** `cd <GameName>/frontend && pnpm install && pnpm dev` (or npm). The page opens with `rgs_url` defaulting to the local dev server; clicking spin calls the dev RGS, which weighted-picks a real book from the math sims and returns its events for the Phaser layer to animate.

5. **Implement the game's event handlers.** Each math event `type` (e.g. `reveal`, `win`, `roomEnter`) maps to an animation in `src/game/events/handlers.ts`. The replay engine awaits each handler so animations sequence correctly.

## Build & Publish

- `pnpm build` → static files in `dist/` with `base: './'` (all asset paths CDN-relative). The build must be **static files only** and load all images/fonts from the CDN — no external requests (stake-rgs-requirements, XSS policy).
- Upload `dist/` to the Stake Engine ACP for the frontend; upload `math/library/publish_files/` for the math. Then follow stake-engine-approval for the review/checklist.

## Guardrails to bake in (from the approval skills)

- Stateless; no jackpot/gamble/continuation/early-cashout. Unique title (no "Megaways"/"Xways"). Original, non-infringing assets; no Stake™/Kick™.
- RTP 90–98%, modes within 0.5%, max win obtainable & matches rules.
- Respect RGS bet levels / `stepBet` / min-max; preserve selected bet on mid-spin refresh; read server from `rgs_url`.
- Disclaimer in the info popup; rules show RTP, max win, paytable, mode costs.
- Spacebar→bet, sound-disable option, autoplay confirmation, >2× mode-switch confirmation.
- If targeting Stake.US, scrub restricted words ("bet"/"buy") and support SC/GC display (stake-jurisdiction-requirements).
- Support replay URLs (stake-replay-requirements).
