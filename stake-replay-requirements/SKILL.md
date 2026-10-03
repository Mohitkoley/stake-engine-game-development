---
name: stake-replay-requirements
description: "Use this skill when implementing or reviewing replay support for a Stake Engine game's approval — replay URLs, optional parameters, end-of-replay event replay, and bet-cost display including bonus/multiplier real cost. Triggers on: Stake Engine replay, replay url, replay support, replay parameters currency language amount, real bet cost, bonus cost display."
---

# Stake Engine — Replay Support

> Games must support replaying a specific past round via a replay URL, honoring optional parameters and clearly showing the true bet cost. Source: Stake Engine approval checklist (https://stake-engine.com/docs/approval/checklist), Replay Support section.

**Related skills:** stake-engine-approval, stake-rgs-requirements

## Requirements

- **Replay URLs:** the game loads and plays the **desired event** from a replay URL.
- **Optional parameters:** supports all optional parameters such as **currency, language, and amount.**
- **End-of-replay:** allows **replaying the "event"** again at the end of a replay.
- **Bet-cost clarity:** the UI must clearly display the **bet cost, including any multiplier applied to the bet, and the "real" bet cost.**
  - Example: `BONUS 1 USD, 250 USD REAL COST` — show both the nominal bonus cost and the actual deducted cost.

## Notes

Replay reuses the same RGS contract (see stake-rgs-requirements): the round is pre-calculated, so replay must reproduce the exact event stream for the given round/seed. On Stake.US, the **replay window must contain no restricted words** — see stake-jurisdiction-requirements.
