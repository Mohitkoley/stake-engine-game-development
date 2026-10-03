---
name: stake-frontend-requirements
description: "Use this skill when building or reviewing the frontend of a Stake Engine game for approval — UI components, rules/paytable display, mobile and popout (mini-player) support, autoplay/sound/spacebar behavior, and asset/CDN rules. Triggers on: Stake Engine frontend approval, UI components, paytable, game rules display, mobile view, popout mini-player, autoplay confirmation, disable sounds, spacebar bet, fastplay, unique assets."
---

# Stake Engine — Frontend & Communication

> Frontend checks review in-game performance and display to ensure the game is free of visual bugs, has industry-standard UI components, and behaves as described in the rules. Source: https://stake-engine.com/docs/approval/frontend-requirements

**Related skills:** stake-engine-approval, stake-game-disclaimer, stake-jurisdiction-requirements

## Game Display

- **Unique assets only.** Backgrounds, symbols, and/or animations shipped with the `web-sdk` sample games **will not be approved**. Audio and visual assets must be original.
- Free of **visual bugs** — no broken or missing assets/animations.
- **Popout (mini-player) support:** the game must render in Stake's small "mini-player" modal without the active game board being visibly distorted.
- **Mobile support** for commonly used devices, with all UI functionality usable during screen scaling.
- **All images and fonts must load from the Stake Engine CDN** (ties into the RGS static-file/XSS policy).

## Rules & Paytable

- Game information accessible from the UI, including a **detailed description of all game rules.**
- If multiple modes exist, describe the **cost of each bet** and the actions being purchased.
- **RTP** of the game (and each mode) must be clearly communicated.
- **Max win amount** for each mode must be clearly displayed.
- **Payout amounts for all symbol combinations** must be presented.
- **Special symbols** (cash prizes, multipliers): list all obtainable values.
- **Feature modes** (e.g. Scatter-triggered): describe how to access them (e.g. "3 Scatters award 10 free spins; 4 Scatters award 15 spins…").

## UI Components

- Include a **UI guide** briefly describing the UI buttons' functionality.
- Player must be able to **change bet size**, using **all bet-levels** returned in the RGS `auth/` response.
- The player's **current balance** must be displayed.
- **Final win amounts** clearly shown for non-zero payouts; if an outcome has multiple winning actions, the payout must **incrementally update** to the final multiplier.
- UI must include an option to **disable sounds.**
- **Spacebar must be mapped to the bet button.**
- **Autoplay** (if present) requires the player to confirm — games may **not** auto-place consecutive bets from one click. (Likewise, switching to a bet-mode costing **>2x** requires a confirmation step.)

## Other Checks

- **Network tab:** no errors and no game information being logged.
- **Playtest** to verify behavior matches the rules (validate payout combinations — e.g. check 10 wins per mode).
- Tested with **various currencies and languages.**
- **Fastplay** (if present): win amounts, winning symbol combinations, and pop-up info must remain legible.
