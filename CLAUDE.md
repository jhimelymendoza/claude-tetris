# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly in a browser:

```bash
open index.html          # macOS
python3 -m http.server 8000  # then visit http://localhost:8000
```

## Architecture

Three files; no framework, no bundler, no dependencies.

- **`index.html`** — DOM structure: a `<canvas id="board">` (300×600 px) for the play field, a `<canvas id="next-canvas">` (120×120 px) for the next-piece preview, a HUD aside (`#score`, `#lines`, `#level`), and a single `#overlay` div reused for both PAUSE and GAME OVER states.
- **`style.css`** — Dark/retro arcade theme. Flexbox layout, monospace font, `backdrop-filter` on overlays.
- **`game.js`** — All game logic (~305 lines, strict mode, no classes).

### game.js internals

**State** lives in module-level `let` variables: `board` (ROWS×COLS matrix of color indices 0–7), `current` and `next` (piece objects `{type, shape, x, y}`), plus `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `lastTime`, `animId`.

**Key functions and their roles:**

| Function | Role |
|---|---|
| `init()` | Full reset; called on load and restart |
| `loop(ts)` | `requestAnimationFrame` tick; accumulates `dropAccum`, triggers auto-drop or `lockPiece()`, then calls `draw()` |
| `collide(shape, ox, oy)` | Boundary and overlap check against `board` |
| `rotateCW(shape)` | Transpose + reverse — produces a clockwise-rotated matrix |
| `tryRotate()` | Applies `rotateCW`; retries with wall-kick offsets `[0,−1,1,−2,2]` |
| `lockPiece()` | `merge()` → `clearLines()` → `spawn()` |
| `clearLines()` | Bottom-up scan; splices full rows and unshifts empty row; updates score/level/`dropInterval` |
| `ghostY()` | Projects `current.y` downward until collision; used by `draw()` and `hardDrop()` |
| `draw()` | Clears canvas, draws grid, board cells, ghost (α 0.2), current piece |
| `drawBlock(ctx, x, y, colorIndex, size, alpha)` | Shared block renderer for both canvases |

**Speed formula:** `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms.

**Scoring:** `LINE_SCORES = [0, 100, 300, 500, 800]` × level for line clears; +2 pts/cell for hard drop; +1 pt/row for soft drop.

### Tunable constants in game.js

`COLS` (10), `ROWS` (20), `BLOCK` (30 px), `COLORS` (7 piece colors), `LINE_SCORES`. If `COLS`, `ROWS`, or `BLOCK` change, the canvas `width`/`height` attributes in `index.html` must be updated to match (`COLS×BLOCK` and `ROWS×BLOCK`).
