# Stake Engine Game Development Skills

A collection of reusable AI-agent skills for building, testing, reviewing, and preparing Stake Engine games for approval.

## Included skills

- `stake-engine-game-development` — end-to-end game development workflow
- `stake-engine-game-setup` — scaffold math and frontend projects
- `stake-math-requirements` — RTP, simulations, volatility, and math validation
- `stake-frontend-requirements` — UI, mobile, popout, assets, and interaction checks
- `stake-rgs-requirements` — RGS authentication, betting, currencies, and languages
- `stake-replay-requirements` — replay-mode implementation and testing
- `stake-engine-approval` — approval readiness and hard-gate checks
- `stake-game-disclaimer` — required rules/info disclaimer
- `stake-jurisdiction-requirements` — Stake.US social-mode compliance
- `stake-game-tile` — game tile asset requirements
- `stake-quality-rankings` — quality-rating and review guidance

## Install globally

Install all skills for supported agent integrations, including Codex and Claude:

```bash
npx skills add https://github.com/Mohitkoley/stake-engine-game-development \
  --all --full-depth -g
```

The repository is public, so no GitHub authentication is required for installation.

## Update

Run the install command again to refresh the globally installed skills:

```bash
npx skills add https://github.com/Mohitkoley/stake-engine-game-development \
  --all --full-depth -g
```

## Use

For Codex or Claude, invoke the main workflow skill with:

```text
$stake-engine-game-development
```

The skills follow the official Stake Engine documentation:

https://stake-engine.com/docs
