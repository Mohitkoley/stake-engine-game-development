---
name: stake-game-disclaimer
description: "Use this skill when adding or reviewing the legal disclaimer required in a Stake Engine game's rules/info popup. Provides the official template text, the required points it must cover, and placement rules. Triggers on: Stake Engine disclaimer, malfunction voids all wins, rules popup legal text, info screen disclaimer, RGS settles winnings wording."
---

# Stake Engine — Game Disclaimer

> Every game must include a legal disclaimer in its rules or information popup. Games submitted **without** a disclaimer in the rules/info popup **will not pass approval.** Source: https://stake-engine.com/docs/approval/disclaimer

**Related skills:** stake-engine-approval, stake-frontend-requirements, stake-jurisdiction-requirements

## Why

Stake Engine uses **pre-calculated game results** — the RGS determines all outcomes before they are displayed; the frontend is purely an animation layer. The disclaimer must make clear that:
- Payouts come from the **server response**, not from visual events in the browser.
- The display is **illustrative only** and does not represent a physical device.
- Network issues may interrupt a round, but it can be resumed by reloading.

## Official Template

> Malfunction voids all wins and plays. A consistent internet connection is required. In the event of a disconnection, reload the game to finish any uncompleted rounds. The expected return is calculated over many plays. The game display is not representative of any physical device and is for illustrative purposes only. Winnings are settled according to the amount received from the Remote Game Server and not from events within the web browser. TM and © 2025 Stake Engine.

You may write your own wording, but the approval team verifies that **all required points** below are covered. When in doubt, use the template.

## Required Points

| Point | Must convey |
|-------|-------------|
| Malfunction clause | Malfunctions void all wins and plays |
| Internet requirement | A stable connection is required |
| Disconnection recovery | Reload the game to finish uncompleted rounds |
| Expected return | RTP is calculated over many plays, not per session |
| Display accuracy | The display is illustrative, not a physical device |
| Payout source | Winnings are determined by the RGS, not browser events |
| Copyright | Appropriate trademark and copyright notice |

## Placement

Accessible from the game's **rules** or **information** screen (typically the `i` or `?` button). It need not be shown on every screen, but must be **easily reachable at all times** during gameplay.

> Note: on Stake.US (social) the disclaimer wording must avoid restricted words — see stake-jurisdiction-requirements.
