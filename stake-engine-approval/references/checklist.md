# Stake Engine — Full Reviewer Approval Checklist

The complete checklist reviewers work through. Source: `static/mockchecklist.json` in StakeEngine/docs (https://stake-engine.com/docs/approval/checklist). All items must be verified before submitting for review.

## PreChecks
- [ ] Game authenticates with RGS successfully on game launch
- [ ] Clicking on the bet button sends a successful play request to RGS
- [ ] Game title is unique and does not contain terms such as Megaways, Xways
- [ ] Game assets and imagery do not contain offensive, discriminatory, or inappropriate content

## Game Thumbnail
- [ ] Game tile is generally bright and does not clash with the Stake background (beware dark edges)
- [ ] Background image is bright and appropriate for the game
- [ ] Foreground image is appropriate and the key focus area is correctly filled
- [ ] Gradient is a similar colour to the background
- [ ] Game title fits within the inner guidelines (not too close to the edges)

## Math Requirements
- [ ] RTP 90% → 98%
- [ ] All modes have an RTP within 0.5% of each other (a 97% game must keep other modes 96.5%–97.5%)
- [ ] Advertised max win is achievable (hit-rate 1 in 20,000,000 or more frequent)
- [ ] Reasonable >0-win hit-rate (typically around 3–8, not >20 for base)

## RGS Requirements
### Bet Levels
- [ ] All bet levels respected — USD $0.10 → $1,000, defaults to USD $1.00
- [ ] All bet levels respected — JPY ¥10 → ¥150,000, defaults to ¥100
- [ ] All bet levels respected — MXN MX1 → MX15,000, defaults to MX10
- [ ] Refreshing mid-spin preserves the selected bet amount (does not revert to default)
### RGS URL
- [ ] Game uses the `rgs_url` query parameter to determine which server it calls (test: change the URL and confirm the game calls it)

## Frontend Requirements
### Game Rules
- [ ] Payout information per symbol is clearly communicated
- [ ] Max multiplier and RTP displayed (per mode if multiple modes)
- [ ] Win combinations displayed (which lines pay, cluster sizes, # symbols for scatter pays, prize payouts, etc.)
- [ ] Description and cost for each available game mode
- [ ] Free-games trigger conditions stated, including re-trigger conditions (e.g. 2 Scatters award +5 spins, 3 Scatters award +10 spins)
- [ ] Disclaimer present (malfunction voids pays/plays; internet required; reload to finish rounds; RTP over many spins; illustrative only; TM and © 2025 Stake Engine)
### Responsive Checks
- [ ] Functions correctly on Desktop
- [ ] Functions correctly on Mobile
- [ ] Functions correctly on Popout S/M
- [ ] Confirmation shows when switching to bet-modes with >2x cost (a 50x bonus mode cannot activate from a single button)
### Auto Play
- [ ] Confirmation step exists — autoplay cannot start from a single click
### Sounds / Music
- [ ] UI option to disable sounds
- [ ] Spacebar bound to the bet button
- [ ] 10 wins per mode checked against Game Rules — displayed win matches payout

## Jurisdiction Requirements
### Stake.US (social-game compliance)
- [ ] Bet button does not say "bet"
- [ ] Game Info contains no restricted word
- [ ] Bet amount field is not labeled "bet amount"
- [ ] Auto-bet feature is not labeled "AutoBET" and popups don't contain "bet"
- [ ] Bonus Buy label does not contain "BUY"; confirmation step does not include "buy" or "bet"
- [ ] Insufficient-funds error has no restricted words
- [ ] Supports SC and GC, and displays these values without a `$` prefix
- [ ] Replay window contains no restricted words

## Replay Support
- [ ] Supports replay URLs — loads and plays the desired event
- [ ] Supports all optional parameters (currency, language, amount)
- [ ] Allows replaying the "event" at the end of replay
- [ ] UI clearly displays bet cost incl. any multiplier and "real" bet cost (e.g. BONUS 1 USD, 250 USD REAL COST)

## Final Approval Checklist
- [ ] Bet-level templates applied (Stake.US must use a template with the `us_` prefix)
- [ ] Both Front and Math requests set to Approved & Active
- [ ] Game appears in `stake-engine-game-approved` (+ `stake-engine-us-game-approved`) channels
- [ ] Approval request closed once the game has the rocket-ship emoji (live on Stake)
