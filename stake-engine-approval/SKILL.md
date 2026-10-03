---
name: stake-engine-approval
description: "Use this skill when preparing a game for submission/approval on Stake Engine, or when checking whether a game meets Stake Engine launch requirements. Covers the key restrictions (stateless, no jackpots/gamble/cashout, IP/originality, content suitability), the review process, post-release change limits, and links to the per-area approval skills. Triggers on: Stake Engine approval, submit game, publish game, go live on Stake, approval process, approval checklist, can my game be approved, rejection reasons."
---

# Stake Engine — Game Approval

> Before a game can go live on the Stake platform it must pass a structured approval process covering math validation, frontend quality, RGS integration, jurisdiction compliance, and replay support. Approval is at the reviewer's discretion and can take as little as 24 hours. Source: https://stake-engine.com/docs/approval (StakeEngine/docs repo).

**Related skills:** stake-math-requirements, stake-rgs-requirements, stake-frontend-requirements, stake-game-disclaimer, stake-game-tile, stake-quality-rankings, stake-jurisdiction-requirements, stake-replay-requirements
**Full reviewer checklist:** [references/checklist.md](references/checklist.md)

## Key Restrictions (hard gates — violating any is grounds for rejection)

- **Stateless only.** Each bet must be independent of previous outcomes. Games **cannot** include jackpots, gamble features, continuation, or early cashout options.
- **IP / copyright.** Team names, game titles, and assets must comply with intellectual-property/copyright law. Infringement is grounds for rejection.
- **Originality.** Games must be original designs. Pre-purchased or licensed games that already exist on other third-party websites are not permitted.
- **No Stake™ / Kick™ branding.** Game assets cannot include material with Stake™ / Kick™ branding or themes.
- **Suitability.** Reviewer discretion applies. Games deemed offensive, explicit, in poor taste, or of insufficient quality may be rejected.
- **No appeal to minors.** Games that promote, encourage, or are likely to appeal to underage persons are not permitted — including artistic depictions of children or child-like characters in any gambling context.
- **Unique title.** The game title must be unique and must **not** contain terms such as `Megaways`, `Xways`, etc.

## Review Process

When a game is submitted, Stake Engine technical support reviews it for **functionality, clarity, communication, and technical performance**. Submit only when the game is **finalized and ready for publication**. Approval requests must include a short blurb describing the game theme and mechanics (used for promotional material and the game description tag).

A game is independently scored by **3 anonymous reviewers**; a minimum average of **1 star** is required for publication — see stake-quality-rankings.

## Post-Release Changes

Once approved, **only minor visual fixes are permitted.** Changes to the underlying **math model**, adding **new game modes**, or modifying **gameplay mechanics** are **not** allowed after approval. Design the math and modes to be final before submitting.

## Final Sign-off (operational)

- Game has bet-level templates applied (Stake.US must use a template with the `us_` prefix).
- Both Front-end and Math requests set to **Approved** & **Active**.
- Game has appeared in the `stake-engine-game-approved` (+ `stake-engine-us-game-approved`) channels.
- Approval request is closed once the game shows the rocket-ship emoji (live on Stake).

## How to use this skill

When a user is getting a Stake Engine game ready, walk the relevant per-area skill(s) above and the full [checklist](references/checklist.md). Flag any **hard gate** violations first (stateless, IP, originality, content) — those block approval regardless of polish.
