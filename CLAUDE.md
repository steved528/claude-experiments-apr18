# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Two single-file browser games. No build step, no dependencies, no package manager. Both files are fully self-contained HTML with inline CSS and JS.

## Running

```
open index.html   # tic-tac-toe
open game.html    # top-down shooter
```

---

## index.html — Tic-Tac-Toe

- **CSS**: Dark retro-terminal theme. CSS variables in `:root` control all colors/glows. Cell borders (not pseudo-elements) draw the 3×3 grid.
- **JS**: `board[]` (9 cells), `current` player, `scores` object. `init()` resets board; `initFull()` also resets scores. Win detection iterates `WINS` constant (8 lines). `renderBoard()` rebuilds all cells from scratch each turn.
- Winning cells get class `winning` → CSS glow animation. Ghost preview re-rendered each `renderBoard()` call with current player's color.

---

## game.html — Top-Down Shooter ("SECTOR ZERO")

### Game states
`MENU → PLAYING → LEVEL_COMPLETE → PLAYING` (next level), or `PLAYING → GAME_OVER → MENU`. Stored as `let gameState`.

### Game loop
`requestAnimationFrame` → `update(dt)` → `render()`. `dt` capped at 50ms to prevent teleport on tab switch. `totalTime` increments every frame regardless of state (used for menu/blink animations).

### Core systems

**Player** (`player` object): arrow keys / WASD movement, mouse aim via `Math.atan2`, hold-click to fire. Mouse coords transformed with `getBoundingClientRect()` + scale factors to handle CSS canvas scaling. Invincibility frames (`iframes`) prevent multi-hit per enemy contact.

**Enemies** — two types, both in the `enemies[]` array:
- `grunt` — red square, moves directly toward player, walking leg animation
- `zigzagger` — purple diamond, adds sinusoidal lateral offset to movement, faster but lower health

**Spawner**: `buildWaveSchedule()` flattens wave group configs into a flat `[{type, spawnAt}]` array sorted by time. Each frame, `spawnTime` increments and entries are popped when due. `spawnerDone` flag set when schedule empties; wave advances only when `spawnerDone && enemies.length === 0` after a 1.8s pause.

**Bullets**: simple array, iterated **backwards** when splicing. Collision vs enemies uses `circlesOverlap()` (avoids `sqrt`). The `outer:` label on the bullet loop allows `continue outer` to skip to the next bullet after a hit.

**Particles**: 10 square/diamond fragments per enemy death, velocity + alpha fade.

### Level config
`LEVELS[]` array — 3 entries. Each has `waves[]` (array of groups), and `stats` per enemy type (speed, health, dmg, score, radius). `startLevel(n)` resets position/arrays but carries player health over for tension. Health resets only on `startGame()`.

### Render order (each frame)
`clearRect` → `drawGrid` → `particles` → `bullets` → `enemies` → `drawPlayer` → `drawVignette` → (restore shake translate) → `drawHUD` → overlay screen if applicable.

HUD and overlay screens are drawn **outside** the shake `ctx.translate` so they don't jitter.

### Shared conventions (both files)
- Fonts: `Bebas Neue` (display headings) + `Share Tech Mono` (mono/body) from Google Fonts
- Palette: `#050508` bg · `#00f5c4` cyan · `#ff4e6a` red · `#b44ff7` purple · `#c0c0d0` text · `#2a2a4a` grid lines
- Scanlines: fixed `<div>` overlay using `repeating-linear-gradient`
- Canvas logical size: 800×600, CSS-scaled with `min()` + `aspect-ratio` to fit viewport
