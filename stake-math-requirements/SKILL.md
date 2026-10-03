---
name: stake-math-requirements
description: "Use this skill when validating the math model of a Stake Engine game for approval — RTP, max win, hit-rate, volatility, and simulation criteria. Triggers on: Stake Engine math approval, RTP range, max win achievable, hit-rate, volatility, standard deviation, number of simulations, zero-weight payouts, math verification."
---

# Stake Engine — Math Verification

> Summary statistics and hit-rate tables are analyzed to ensure the game adheres to industry standards for chance-based casino games and is not misleading. Source: https://stake-engine.com/docs/approval/math-requirements

**Related skills:** stake-engine-approval

## Summary Statistics

- **Mode cost** must be correctly represented in the game rules for each mode.
- **RTP must be within 90.0%–98.0%.** For multiple modes, all must fall within a **0.5%** variation of each other (e.g. a base game at 97% RTP requires other modes to be 96.5%–97.5%).
- **Max win** must match the description in the game rules for each mode.
- **Max win must be realistically obtainable** — typically more frequent than **1 in 10,000,000** (the checklist cites 1 in 20,000,000 or more frequent), depending on payout amount.
- **Simulation count:** for slot-type games run **100,000–1,000,000 simulations** to ensure outcome diversity and avoid repeated results in a single session.
- **Paying portion:** a reasonable portion of simulations should yield paying results (e.g. 90,000 non-paying out of 100,000 may be grounds for rejection).
- **No single dominant outcome:** the hit-rate of the most likely single simulation should not be overwhelmingly dominant if results are visually expected to be varied.

## Other Considerations

- **Non-zero win hit-rate** should align with industry standards — **<1 in 20 bets** (around 3–8 is typical for base), or more frequent.
- **BASE modes (1x cost):** standard deviation should be within industry norms for reasonable slot volatility.
- **Non-zero weight payouts:** list how many there are; zero-weight payouts should not dominate the provided simulations.
- **No win-range gaps:** inspect hit-rates across win ranges so intermediate wins exist between small payouts and the maximum (avoid unobtainable expected win amounts).

## Quick gate

RTP ∈ [90%, 98%] · all modes within 0.5% · max win obtainable & matches rules · non-zero hit-rate roughly 1-in-3 to 1-in-20 · 100k–1M sims with varied, mostly-paying-enough outcomes.
