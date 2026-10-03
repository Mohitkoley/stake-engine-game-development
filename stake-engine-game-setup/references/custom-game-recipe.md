# Custom (non-reel) Game Recipe

Use this when the game is **not** a reel/board game — an event-stream game where
each simulation runs some custom logic and emits a list of events the frontend
animates ("watch a run / match / draw resolve"). The SDK samples and the
`template` are reel-first; this recipe is the minimal path that avoids
reverse-engineering the engine. (For reel games, stick with the samples.)

## The minimal `gamestate.py`

A custom game does **not** need reels, a paytable, or the optimizer. It needs a
`run_spin` that emits events and reports one capped payout multiplier. Mirror
`games/fifty_fifty/` (the only non-reel sample), generalized:

```python
import random
from game_override import GameStateOverride

class GameState(GameStateOverride):
    def run_spin(self, sim, simulation_seed=None):
        self.reset_seed(sim)            # seeds the GLOBAL `random` module
        self.repeat = True
        while self.repeat:
            self.reset_book()
            mode = str(self.betmode)    # current bet-mode name, e.g. "party2"

            def emit(event):
                event = dict(event)
                event["index"] = len(self.book.events)   # EVERY event needs index
                self.book.add_event(event)

            result = my_engine.run(roll, emit)           # your logic; the only RNG
            self.win_manager.update_spinwin(result.multiplier)   # RAW multiplier units
            self.win_manager.update_gametype_wins(self.gametype) # REQUIRED (see asserts)
            self.evaluate_finalwin()    # sets payoutMultiplier; appends a "finalWin" event
        self.imprint_wins()

    def run_freespin(self):
        pass
```

`game_config.py` for such a game is small: `num_reels = 0`, empty `paytable`,
and one `BetMode` per discrete lever, each with a **single natural distribution**
(no forcing):

```python
distributions=[Distribution(criteria="allwins", quota=1.0,
    conditions={"reel_weights": {}, "force_wincap": False, "force_freegame": False})]
```

`game_calculations.py` / `game_executables.py` can be one-line `pass` subclasses;
`game_events.py` is just `from src.events.events import *`.

## math-sdk internals cheat-sheet (the gotchas that bite while writing the above)

- **Scale:** `book.payoutMultiplier = final_win × 100`. `update_spinwin()` takes the
  **raw** multiplier (e.g. `3.25`), not ×100.
- **Every event needs `index = len(self.book.events)`** or the frontend contract breaks.
- **`evaluate_finalwin()` already appends a `finalWin` event** (type `"finalWin"`,
  amount ×100). If your engine *also* emits `finalWin`, they collide — don't.
- **`update_final_win()` asserts** `basegame_wins + freegame_wins == running_bet_win`.
  You must call `update_gametype_wins(self.gametype)` after `update_spinwin`, or it raises.
- **`reset_seed(sim)` seeds the global `random` module** — pass `random` itself as
  your engine's RNG for per-sim reproducibility.
- **`self.betmode`** is the active mode name inside `run_spin`.
- **Determinism:** keep all per-sim randomness in one place (roll/seed generation);
  let the run itself be deterministic. Reproducible and far easier to certify.

## Wincap & forced criteria — the hang trap

For an **emergent heavy tail** (big win arises from mechanics, not a forced
jackpot), do **NOT** add a `win_criteria=wincap` distribution: `create_books`
resamples until it produces that exact payout, and if the wincap isn't
mechanically reachable yet it **loops forever**. Use a single natural
distribution and let the weighter/optimizer shape frequencies. Only add a forced
criteria once you've confirmed the engine can actually hit it.

## Lookup tables: base vs published `_0`

`run_sims` + `generate_configs` write the **base** lookup
`library/lookup_tables/lookUpTable_<mode>.csv` with **uniform weight 1**. The RGS
(and dev RGS) read the **published** `library/publish_files/lookUpTable_<mode>_0.csv`,
which `index.json` references. The official optimizer produces `_0`. **If you use
the custom weighter instead, the weighter must write `_0` itself** — otherwise the
`_0` file is missing/stale and the dev RGS can't load.

## Per-unit RTP with mode cost

The debit is `amount × cost` but the win is `amount × payoutMultiplier / 100`, so
per-unit RTP = `mean(multiplier) / cost`. For a mode to **read** RTP `R` after
analysis divides by cost, target **weighted-mean multiplier = `R × cost`** (e.g. a
cost-3 mode at 96% needs mean multiplier `2.88`, not `0.96`). The weighter/optimizer
target must use `R × cost`, not `R`.

## Ready-made custom weighter (`tools/weight_books.py`)

Game-agnostic. Reads the base lookup, applies a single exponential tilt
`w_i ∝ exp(α·multiplier_i)`, and bisects `α` so the weighted-mean multiplier hits
`R × cost` (max-entropy reweighting — least-distorting fit, preserves the tail
shape). Writes `_0` and reports the stats. This is the fast, reliable fallback the
math-scaffold mentions; run the official optimizer for the final certified build.

```python
import csv, math, os
WEIGHT_SCALE = 1_000_000

def _read_base(path):
    ids, pm = [], []
    for row in csv.reader(open(path, newline="")):
        if row: ids.append(int(row[0])); pm.append(int(row[2]))
    return ids, pm

def _mean(mults, a, ref):
    num = den = 0.0
    for m in mults:
        w = math.exp(max(-700.0, min(700.0, a * (m - ref))))   # clamp: bisection visits big |a|
        den += w; num += w * m
    return num / den if den else 0.0

def _solve_alpha(mults, target, ref):
    if target <= min(mults) + 1e-9: return -1e9
    if target >= max(mults) - 1e-9: return 1e9   # unreachable without a fatter tail
    lo, hi = -200.0, 200.0
    for _ in range(200):
        mid = (lo + hi) / 2
        if _mean(mults, mid, ref) < target: lo = mid
        else: hi = mid
    return (lo + hi) / 2

def weight_mode(library_path, mode, cost, rtp):
    base = os.path.join(library_path, "lookup_tables", f"lookUpTable_{mode}.csv")
    out  = os.path.join(library_path, "publish_files", f"lookUpTable_{mode}_0.csv")
    ids, pm = _read_base(base)
    mults = [p / 100.0 for p in pm]
    ref = max(mults) if mults else 0.0
    a = _solve_alpha(mults, rtp * cost, ref)            # per-unit RTP: target = rtp * cost
    raw = [math.exp(max(-700.0, min(700.0, a * (m - ref)))) for m in mults]
    wmax = max(raw) or 1.0
    weights = [max(1, round(WEIGHT_SCALE * (w / wmax))) for w in raw]
    with open(out, "w", newline="") as f:
        for i, w, p in zip(ids, weights, pm): csv.writer(f).writerow([i, w, p])
    tot = sum(weights)
    mean_pm = sum(p * w for p, w in zip(pm, weights)) / tot
    hit = sum(w for p, w in zip(pm, weights) if p > 0) / tot
    print(f"[{mode}] per-unit RTP={mean_pm/100/cost*100:.3f}%  hit-rate=1 in {1/hit:.1f}  max={max(pm)/100:.2f}x")
```

## Uncompressed books for the dev RGS

With `compression=True` the SDK writes only `publish_files/books_<mode>.jsonl.zst`,
and Python's zstd frames omit a content-size header (fzstd can choke). The dev RGS
prefers an uncompressed **`.json` array** at `library/books/books_<mode>.json`
(note: the SDK's own uncompressed output is `.jsonl` *lines*, not a `.json`
array — different format). Emit the array yourself:

```python
import io, json, os, zstandard as zstd
def write_uncompressed_books(library_path, modes):
    pub = os.path.join(library_path, "publish_files")
    out = os.path.join(library_path, "books"); os.makedirs(out, exist_ok=True)
    for m in modes:
        with open(os.path.join(pub, f"books_{m}.jsonl.zst"), "rb") as fh:
            text = io.TextIOWrapper(zstd.ZstdDecompressor().stream_reader(fh), encoding="utf-8").read()
        books = [json.loads(l) for l in text.strip().split("\n") if l]
        json.dump(books, open(os.path.join(out, f"books_{m}.json"), "w"))
```

Keep dev sim counts modest (**1e4–1e5**): long event streams make these arrays
large (hundreds of MB at 2e4 for deep games), which slows writing and the dev
RGS's startup parse. Pair with the lazy-per-mode dev RGS (see frontend-scaffold).
```
