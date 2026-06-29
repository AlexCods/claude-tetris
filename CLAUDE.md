# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly in a browser:

```bash
open index.html          # macOS
python3 -m http.server 8000  # then visit http://localhost:8000
```

## Architecture

Three files, no dependencies, no framework:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600 px) for the game field, `<canvas id="next-canvas">` (120×120 px) for the next-piece preview, a sidebar panel with live score/lines/level displays, and an `#overlay` div that doubles as the pause screen and game-over screen.
- **`style.css`** — Dark retro-arcade theme. The overlay uses `backdrop-filter: blur` and is toggled via `.hidden`.
- **`game.js`** — All game logic (~305 lines). Entry point is `init()`, called on load and on restart.

### game.js internals

**State** (module-level `let` vars): `board` (2-D array ROWS×COLS, `0` = empty, `1–7` = piece color index), `current` and `next` (piece objects `{type, shape, x, y}`), `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `lastTime`, `animId`.

**Key constants** (top of file — change these to tune the game):

| Constant | Default | Notes |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Also update canvas `width`/`height` in HTML if changed |
| `BLOCK` | 30 px | Pixel size of each cell |
| `COLORS` | 7-entry array | Index 0 is `null`; indices 1–7 map to piece types |
| `PIECES` | 7-entry array | Each piece is a 2-D matrix; index 0 is `null` |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |

**Game loop**: `requestAnimationFrame(loop)` accumulates elapsed time in `dropAccum`; when it exceeds `dropInterval` the piece drops one row or locks.

**Piece lifecycle**: `spawn()` → gravity in `loop()` → `lockPiece()` → `merge()` + `clearLines()` → `spawn()`. If the new piece collides immediately on spawn, `endGame()` fires.

**Rotation**: `rotateCW(shape)` (transpose + reverse rows). `tryRotate()` tries the rotation then applies wall kicks `[0, −1, +1, −2, +2]` column offsets until one doesn't collide.

**Rendering**: `draw()` clears the board canvas, draws the grid, the locked blocks, the ghost piece (`globalAlpha = 0.2`), then the active piece. `drawNext()` renders on the separate preview canvas. `drawBlock()` accepts an optional `alpha` for the ghost.

**Scoring**: hard drop +2 per cell fallen; soft drop +1 per row; line clears use `LINE_SCORES[count] * level`. Level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 − (level−1) × 90)` ms.
