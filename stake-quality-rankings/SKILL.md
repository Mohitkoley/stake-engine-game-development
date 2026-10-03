---
name: stake-quality-rankings
description: "Use this skill to understand how Stake Engine scores game quality (0–3 stars), the fractional 3-reviewer rating process, the minimum 1-star approval threshold, the below-threshold resubmission process, and how star tiers affect category placement/visibility. Triggers on: Stake Engine quality ranking, star rating, fractional rating, anonymous reviewers, minimum 1 star, New Releases, Burst Games, Stake Exclusives, resubmit after rejection."
---

# Stake Engine — Game Quality Rankings

> All games get a Quality Ranking from **0 to 3 stars**, determined by a fractional rating process with multiple anonymous reviewers. This affects visibility and positioning eligibility. A minimum average of **1 star** is required to be approved. Source: https://stake-engine.com/docs/approval/quality

**Related skills:** stake-engine-approval

## Rating Process

Each game is evaluated by **3 independent, anonymous reviewers.** Each picks one **fractional score** from this fixed 10-value scale (no arbitrary decimals):

> **0** · 0.33 · 0.67 · **1** · 1.33 · 1.67 · **2** · 2.33 · 2.67 · **3**

1. Three anonymous reviews — each scores quality, creativity, and polish.
2. **Average** the three scores → a single fractional value.
3. **Round** to the nearest whole number → final star tier (0, 1, 2, or 3).

**Critical:** if the average is **below 1.0**, the game gets a **0-star rating and is NOT approved**, regardless of rounding. An average of 0.67 does **not** round up to 1 — it is below the minimum threshold.

Example: 2.33, 2.33, 3.00 → average 2.55 → **rounds to 3 stars.**

## Ranking Tiers & Visibility

| Tier | Meaning | Visibility |
|------|---------|------------|
| **3 stars** | Studio-quality; exceptional creativity, uniqueness, detail | Optimal — eligible for Burst Games, Stake Exclusives, and the *featured* section of New Releases |
| **2 stars** | Considerable creativity/originality; less polish | May appear in Burst Games / Stake Exclusives if driven by popularity; New Releases placement depends on space/demand |
| **1 star** | Lower polish but meets publishing requirements | Published with limited visibility; always at the bottom of New Releases; no promo categories unless exceptional demand |
| **0 stars** | Below 1-star average | **Not published** — does not appear on the platform |

## Minimum Quality Threshold (in effect since March 2026)

Games averaging **below 1 star are not approved.** Process:
1. **Thread closed** and locked.
2. **7-day improvement window** — thread stays locked 7 days.
3. **Resubmission** — after 7 days you can resubmit for a new review cycle.

Not a permanent rejection — the goal is to help raise quality.

## Category Placement (updated weekly)

- **New Releases:** all 1+ star games get a New Release tag. 3★ prioritized/featured; 2★ if space allows; 1★ always at the bottom unless sorted by Newest.
- **Burst Games:** priority to 3★; 2★ may appear if popularity drives demand.
- **Stake Exclusives:** priority to 3★; 2★ may appear if driven by popularity.
