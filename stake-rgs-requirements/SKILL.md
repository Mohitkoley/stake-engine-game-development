---
name: stake-rgs-requirements
description: "Use this skill when integrating or validating a Stake Engine game against the Remote Game Server (RGS) for approval — authentication, bet levels, XSS/static-file policy, rgs_url parameter, and currency/language support. Triggers on: RGS communication, authenticate response, bet levels, minStep, rgs_url, XSS policy, static files, supported currencies, languages, play request fails."
---

# Stake Engine — RGS Communication

> Session authentication and bet transactions are handled exclusively through the Stake Engine RGS. The RGS manages session token generation, `play/` responses, and optional parameters like supported currencies and languages. Source: https://stake-engine.com/docs/approval/rgs-requirements

**Related skills:** stake-engine-approval, stake-jurisdiction-requirements

## RGS Authentication & Bet Levels

- The `authenticate` HTTP response returns **default bet levels**, **supported bet levels** for the session currency, and **min/max bet amounts**. The frontend **must respect these values**.
  - Example failure: default bet size 1 unit but the session uses JPY (min bet 10 units) → the `play/` request will fail.
- **Bet increments** must reflect allowed values within `authenticate/config/minStep`.
- **Min and max bet levels** must be available for selection as dictated by the RGS.
- Bet-level test cases (from the checklist): USD $0.10→$1,000 (default $1.00); JPY ¥10→¥150,000 (default ¥100); MXN MX1→MX15,000 (default MX10).
- **Refreshing mid-spin must preserve the selected bet amount** — it must not revert to the default.

## Cross-Site Scripting (XSS) — static files only

- Stake Engine enforces a **strict XSS policy.** The game build must consist **only of static files** and cannot reach external sources.
- Common failure: downloading **fonts from external servers** (logs console errors). Load all images and fonts from the **Stake Engine CDN** instead.

## RGS URL

- The game **must** use the **`rgs_url` query parameter** to determine which server to call. Test: change `rgs_url` and confirm the game calls that URL.

## Currency and Language

- **English (`en`) is the only required language.** If only English is supported, on-screen text must **not corrupt** when other language parameters are passed.
- Reference lists: https://stake-engine.com/docs/reference/languages and https://stake-engine.com/docs/reference/currencies
