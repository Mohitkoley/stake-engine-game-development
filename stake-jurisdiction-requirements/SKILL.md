---
name: stake-jurisdiction-requirements
description: "Use this skill when making a Stake Engine game compliant for specific jurisdictions, especially Stake.US social-game requirements: restricted/banned wording and SC/GC currency display. Triggers on: Stake.US, social game compliance, restricted words, cannot say bet, bonus buy wording, SC GC sweepstakes coins, gold coins, us_ template, jurisdiction requirements."
---

# Stake Engine — Jurisdiction Requirements

> Some jurisdictions require wording and currency adjustments. The main case is **Stake.US**, which runs as a **social game** and forbids gambling terminology. Source: Stake Engine approval checklist (https://stake-engine.com/docs/approval/checklist), Jurisdiction Requirements section.

**Related skills:** stake-engine-approval, stake-frontend-requirements, stake-game-disclaimer

## Stake.US — Social-game wording (restricted words)

The game must be compliant with the required translations for a social game. In particular:

- The **bet button does not say "bet."**
- **Game Info** contains **no restricted word.**
- The **bet amount field is not labeled "bet amount."**
- The **autobet feature is not labeled "AutoBET"**, and any popups do **not contain the word "bet."**
- The **Bonus Buy label does not contain "BUY"**, and its **confirmation step does not include "buy" or "bet."**
- The **insufficient-funds error** has **no restricted words.**
- The **Replay window** contains **no restricted words.**

> Treat words like **bet / buy** (and their compounds) as banned across all visible UI text, popups, errors, and replay views in the Stake.US build. Audit the disclaimer text too — see stake-game-disclaimer.

## Stake.US — Currencies (SC / GC)

- The game must **support SC (Sweeps Coins) and GC (Gold Coins).**
- It must **display these values without a `$` sign prefix.**

## Operational

- Apply **bet-level templates**; **Stake.US must use a template with the `us_` prefix.**
- Approved Stake.US games appear in the `stake-engine-us-game-approved` channel.
