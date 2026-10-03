---
name: stake-game-tile
description: "Use this skill when creating or reviewing the game tile/thumbnail for a Stake Engine game using the dashboard Tile Editor. Covers provider logo, background/foreground/gradient/title layers, and the thumbnail imagery/gradient/text rules that cause rejection. Triggers on: Stake Engine game tile, thumbnail, Tile Editor, provider logo, background image too dark, gradient overlay, title text, key focus area, tile rejected."
---

# Stake Engine — Game Tile

> Game tiles are created in the Stake Engine dashboard **Tile Editor** (game page → hover thumbnail → pen icon). You compose the final tile from layers — no separate asset submission. Tiles that don't meet these requirements are rejected. Source: https://stake-engine.com/docs/approval/game-tile

**Related skills:** stake-engine-approval

## Provider Logo (set once)

Configured in **Team Settings → Branding** and applied automatically to all tiles — not submitted per game.
- PNG, JPG, or GIF up to **10 MB**; **square ratio** recommended (shown small).
- Use a **transparent background with padding** so it stays legible at small sizes.
- Displayed publicly on stake.com alongside your tiles.

## Tile Editor Layers

- **Background image** — environmental background showing the game world. High-res PNG/JPG. Use the Lighten/Darken slider; the editor's **Brightness Analysis** warns if too dark.
- **Foreground element** — feature character / key item, high-res **PNG with transparent background**; position/scale within the tile.
- **Gradient** — overlay between background and title area; keeps title text readable.
- **Game title** — title text with configurable font, size, position.

Controls: Snapping, Clip Overflow, Guidelines (safe-zone / key focus), Download, Save.

## Thumbnail Guidelines (rejection criteria)

### Imagery
- **Background must be brighter than the Stake platform** — dark backgrounds blend in and make the tile invisible.
- **Avoid dark colours around the edges** — they blend into the Stake background.
- Background and foreground must be **bright and engaging.**
- **Enlarge characters/symbols to fill the key focus area** (align the focus point to the editor's red box).
- **No wording or multipliers baked into the background or foreground** — text is handled by the title layer.

> Tiles with dark edges, low-contrast backgrounds, or text/multipliers in the imagery **will be rejected.**

### Gradient
- Pick a **prominent colour already present** on the tile so it feels like a natural extension.
- Keep it **light** and use it to **enhance text legibility.**
- **Avoid bright yellows, greens, or blues** as gradient colours (they reduce text legibility); don't let it overpower the imagery.

### Text
- Title text must **fit within the height of the guide** (don't exceed the text area).
- **Fill as much of the text-box width as possible** — small centred text with large margins looks weak at thumbnail size.
- Max of **2 text sizes** per thumbnail (e.g. large game name + smaller subtitle).
- Preview at small sizes before saving.
