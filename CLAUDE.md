# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step — open `index.html` directly or serve with any static server:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
open index.html               # macOS direct open
```

## Architecture

Three files, no dependencies:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600px), aside panel with score/lines/level/next-piece preview (`<canvas id="next-canvas">`), and a hidden overlay for PAUSE/GAME OVER states.
- **`style.css`** — Dark/retro arcade theme. Overlay uses `backdrop-filter`.
- **`game.js`** — All game logic (~305 lines, `'use strict'`, ES6+).

### game.js internals

**State** (module-level `let` vars): `board` (2D array, `ROWS×COLS`, 0=empty / 1–7=piece color index), `current`/`next` (piece objects `{type, shape, x, y}`), `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `lastTime`, `animId`.

**Key functions:**

- `collide(shape, ox, oy)` — bounds + overlap check; used everywhere before moving/rotating.
- `rotateCW(shape)` — transpose + reverse for clockwise rotation.
- `tryRotate()` — applies rotation with wall kicks `[0, -1, 1, -2, 2]` column offsets.
- `ghostY()` — projects `current` straight down; result used for ghost rendering and hard drop.
- `lockPiece()` → `merge()` → `clearLines()` → `spawn()` — the piece landing sequence.
- `loop(ts)` — `requestAnimationFrame` loop; accumulates `dropAccum`, triggers auto-drop or `lockPiece` when `dropAccum >= dropInterval`.
- `init()` — full reset; also called by the restart button.

**Speed formula:** `dropInterval = max(100, 1000 − (level − 1) × 90)` ms. Level increments every 10 lines.

**Scoring:** `LINE_SCORES = [0, 100, 300, 500, 800]` × level; soft drop +1/row, hard drop +2/cell.

### Tunable constants (top of game.js)

`COLS`, `ROWS`, `BLOCK` (px per cell), `COLORS` (array indexed 1–7), `LINE_SCORES`. If `COLS`/`ROWS`/`BLOCK` change, update `<canvas>` `width`/`height` in `index.html` to match (`COLS×BLOCK` × `ROWS×BLOCK`).
