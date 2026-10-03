# Math ↔ Frontend Contract

How the Phaser frontend consumes the math-sdk output. Source of truth: Stake Engine RGS docs (`rgs_docs/data_format.md`, `rgs_docs/RGS.md`) and the actual math-sdk `library/` output.

## Files the math run produces (under `<game>/library/`)

| Path | Purpose |
|------|---------|
| `publish_files/index.json` | Lists modes. **Upload this dir to the ACP.** |
| `publish_files/lookUpTable_<mode>_0.csv` | `id,probabilityWeight,payoutMultiplier` per simulation (uint64). |
| `publish_files/books_<mode>.jsonl.zst` | zstd JSON-lines, one object per simulation. |
| `books/books_<mode>.json` | **Uncompressed** books array — the dev RGS reads this (no zstd lib needed). |

> **Two wiring facts the math run must satisfy:**
> - **`index.json` points the `weights` at `lookUpTable_<mode>_0.csv`** (the *published*
>   lookup), which the **optimizer** writes. If you use the custom weighter instead of
>   the optimizer, the weighter must write `_0` itself — the base
>   `lookup_tables/lookUpTable_<mode>.csv` (uniform weight 1) is not what the RGS reads.
> - The uncompressed `books/books_<mode>.json` must be a **JSON array**. The SDK's own
>   uncompressed output is `.jsonl` *lines* (a different format), and Python's
>   `.jsonl.zst` frames can defeat fzstd — so produce the array explicitly (see
>   [custom-game-recipe.md](custom-game-recipe.md)).

### index.json
```json
{
  "modes": [
    { "name": "base",  "cost": 1.0,   "events": "books_base.jsonl.zst",  "weights": "lookUpTable_base_0.csv" },
    { "name": "bonus", "cost": 100.0, "events": "books_bonus.jsonl.zst", "weights": "lookUpTable_bonus_0.csv" }
  ]
}
```

### lookup table CSV (no header)
```
0,60856,0
1,1000000,67
2,60856,0
```
Columns: `id`, `probabilityWeight` (relative weight for selection), `payoutMultiplier`. The CSV `payoutMultiplier` must match the book's for that id (RGS hashes them).

### book object (one simulation)
```json
{ "id": 1, "events": [ { "index": 0, "type": "reveal", ... }, ... ], "payoutMultiplier": 1150 }
```
Required keys: `id`, `events`, `payoutMultiplier`. Extra keys (e.g. `criteria`, `baseGameWins`) are allowed and ignored by the contract.

## Scales & money (critical)

- **`payoutMultiplier` = multiplier × 100.** `1150` = 11.5×, `3980` = 39.8×, `0` = loss. `win = floor(baseBet * payoutMultiplier / 100)`.
- **Money = integer, 6 decimal places.** `1_000_000` = 1.0; a $1 bet = `1000000`. Currency affects display only, never logic.
- **Debit** on `/play` = `baseBetAmount × modeCost`. The win is credited on `/end-round`, not on `/play`.

## How the dev RGS emulates a round (matches prod semantics)

1. `/wallet/authenticate` → balance + `config` (minBet, maxBet, stepBet, defaultBetLevel, betLevels, jurisdiction) + active `round` (or null).
2. `/wallet/play` with `{ amount, mode, sessionID }`:
   - validate `amount` ∈ [minBet, maxBet] and divisible by `stepBet`;
   - debit `amount × modeCost`;
   - weighted-random pick an `id` using the lookup `probabilityWeight` column;
   - load that book; `payout = floor(amount × book.payoutMultiplier / 100)`;
   - return `round = { state: book.events, payoutMultiplier, payout, mode, amount, active: true }`.
3. Frontend replays `round.state` events (animation only).
4. `/wallet/end-round` → credit `payout`, mark round inactive, return new balance.

In dev, the frontend's `rgs_url` defaults to the local dev-server origin, so `client.ts` is byte-identical in dev and prod.

## Replay support

A replay URL re-runs a specific past round. The dev RGS supports `?replay=<mode>:<id>` to force a specific book id (and honors optional `currency`/`lang`/`amount`). See stake-replay-requirements. UI must show real bet cost incl. multiplier (e.g. `BONUS 1 USD, 250 USD REAL COST`).
