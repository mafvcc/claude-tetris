# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly or serve statically:

```bash
open index.html                  # macOS
python3 -m http.server 8000      # then http://localhost:8000
npx serve .
```

## Architecture

Single-page, zero-dependency game in three files:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600px), sidebar panel (score/lines/level/next-piece preview), and a shared overlay div used for both PAUSE and GAME OVER states.
- **`style.css`** — Dark/retro theme. No CSS framework.
- **`game.js`** — All game logic (~300 lines, `'use strict'`, no modules).

### game.js internals

- **Board state**: `board` is a `ROWS×COLS` array; cells hold `0` (empty) or a color index (1–7).
- **Piece objects**: `{ type, shape, x, y }` where `shape` is a 2D matrix. Rotation uses `rotateCW` (transpose + row-reverse).
- **Collision**: `collide(shape, ox, oy)` — checks bounds and board overlap.
- **Wall kicks**: `tryRotate` tries offsets `[0, -1, 1, -2, 2]` before discarding a rotation.
- **Game loop**: `requestAnimationFrame`-based; `dropAccum` accumulates elapsed ms, triggers `lockPiece()` when ≥ `dropInterval`.
- **Line clear**: `clearLines` splices full rows and unshifts empty ones; updates score, level, and `dropInterval = max(100, 1000 − (level−1)×90)`.
- **Ghost piece**: `ghostY()` projects the current piece straight down; drawn at `globalAlpha = 0.2`.
- **Scoring**: `LINE_SCORES = [0,100,300,500,800]` × level; hard drop +2/cell, soft drop +1/row.
- **Entry point**: `init()` resets all state and starts the loop. Called once on load and wired to the restart button.

### Key constants to tune

| Constant | Default | Note |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Also update canvas `width`/`height` in `index.html` |
| `BLOCK` | 30 | Pixel size per cell |
| `COLORS` | 7-color array | Index matches piece type |
| `LINE_SCORES` | `[0,100,300,500,800]` | Points per 1–4 lines cleared |
