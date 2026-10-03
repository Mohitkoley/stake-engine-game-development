# Math Package Scaffold (math-sdk game)

How to build the **math** half from scratch, inside a cloned `math-sdk/` (see [environment-setup.md](environment-setup.md)). The frontend consumes its `library/` output per [math-contract.md](math-contract.md).

> **Non-reel / event-stream game?** (watch a run/match/draw resolve — no reels or
> paytable). The samples are reel-first; follow [custom-game-recipe.md](custom-game-recipe.md)
> instead — it has the minimal `run_spin`, an internals cheat-sheet, a ready-made
> custom weighter, and the dev-RGS book/format steps. The rest of this file is for
> reel/board games.

## Create the game package

The SDK ships a `games/template/` and several samples (`games/0_0_lines`, `0_0_cluster`, `0_0_ways`, `0_0_scatter`, `fifty_fifty`, …). Start from the one whose mechanic is closest, copy it to `games/<GameName>/`, then edit. A game package contains:

```
math-sdk/games/<GameName>/
  game_config.py        # game_id, bet modes + costs, RTP/wincap, reels/paths
  gamestate.py          # per-simulation state machine: plays one round, emits events
  game_events.py        # event constructors (the event stream the frontend animates)
  game_executables.py   # reusable round operations
  game_calculations.py  # win/eval math
  game_override.py       # hooks to override base SDK behavior
  game_optimization.py   # optimizer setup (conditions/criteria for weighting)
  run.py                # the pipeline entry point (sims → optimize → configs → checks)
  reels/                # CSV strips, if a reel-based game
```

## Critical gotcha: `game_id` must equal the folder name

In `game_config.py`, `game_id` (or `self.game_id`) **must exactly match the folder name**, including case. The SDK builds all `library/` paths from `game_id`; a mismatch writes books to a differently-cased path and the optimizer/analysis can't find them.

## Define the game

- **Bet modes** = the discrete levers (e.g. one mode per quote, or base/bonus). Each has a `cost`. No sliders, stateless, no jackpot/gamble/cashout (stake-engine-approval, stake-math-requirements).
- **Events**: `gamestate.py` emits a list of events per simulation via `game_events.py`. Each event needs a `type` and whatever fields the frontend handler reads. These event `type`s are the contract the Phaser `ReplayEngine` keys on.
- **Payout**: every round resolves to one capped `payoutMultiplier` (integer = multiplier × 100). Wincap is the max-win cap.

## Run the pipeline

`run.py` orchestrates it (mirrors the SDK samples):
```python
run_conditions = {"run_sims": True, "run_optimization": True, "run_analysis": True, "run_format_checks": True}
num_sim_args = {"<mode>": int(1e5)}   # 100k–1M for slot-type variety (stake-math-requirements)
```
Invoke with the game on the path:
```bash
cd math-sdk
PYTHONPATH=games/<GameName> env/bin/python games/<GameName>/run.py
```

Windows PowerShell:

```powershell
$env:PYTHONPATH = "games/<GameName>"
& .\env\Scripts\python.exe games\<GameName>\run.py
```
This writes `games/<GameName>/library/` → `publish_files/index.json`, `lookUpTable_<mode>_0.csv`, `books_<mode>.jsonl.zst`, and the uncompressed `books/books_<mode>.json` used by the dev RGS.

### Optimizer choice
- **Standard/official optimizer** (Rust, `optimization_program`): use it when the win distribution is tractable — it produces the certified-style weights Stake expects.
- **Custom weighter fallback**: for an **emergent heavy-tail** (a big max win that arises from mechanics, not a forced jackpot), the Rust optimizer can be very slow. A small custom script that sets lookup-table weights so the weighted mean payout = `RTP × 100 × cost` (winning books uniform, zero-win books up-weighted to pull the mean to target) is instant and preserves the natural shape. A ready-made exponential-tilt version is in [custom-game-recipe.md](custom-game-recipe.md). Document which you used and why. For Stake certification, the official optimizer is the one that counts — run it for the final build even if you iterate with the custom weighter.

> **Two traps when weighting cost>1 modes / emergent tails:**
> - **Per-unit RTP & cost.** The debit is `amount × cost` but the win is on `amount`,
>   so per-unit RTP = `mean(multiplier) / cost`. Target the weighted-mean multiplier
>   at **`RTP × cost`** (a cost-3 mode at 96% needs mean multiplier `2.88`), or the
>   optimizer hits the wrong number.
> - **Forced-wincap hang.** Don't add a `win_criteria=wincap` distribution for an
>   emergent tail — `create_books` resamples until it produces that exact payout and
>   **loops forever** if the wincap isn't mechanically reachable yet. Use a single
>   natural distribution; only force criteria the engine can actually hit.

## Verify against approval targets

After the run, confirm (stake-math-requirements):
- RTP within **90–98%**; all modes within **0.5%** of each other.
- Max win matches the rules and is **obtainable** (≈ 1 in 10–20M or more frequent).
- Non-zero hit-rate roughly 1-in-3 to 1-in-20; no win-range gaps; outcomes varied (100k–1M sims).

Report RTP, hit-rate, max win, and volatility per mode.
