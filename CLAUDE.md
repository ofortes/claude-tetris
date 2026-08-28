# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build step, no package.json — just static files.

## Running

Open `index.html` directly in a browser, or serve it locally (required if testing features that need a server context):

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, or test tooling in this repo — changes to `game.js` are verified by reloading the page in a browser and playing.

## Architecture

Three files, no modules:

- `index.html` — DOM structure: main `<canvas id="board">` (300×600, 10×20 cells at `BLOCK=30`px), a side panel (score/lines/level/next-piece canvas/controls), and a pause/game-over overlay.
- `style.css` — dark/retro arcade theme.
- `game.js` — all game logic, plain script (not a module), executes top-to-bottom and ends with `init()` which kicks off the game loop.

### Core model (`game.js`)

- `board`: `ROWS × COLS` matrix; each cell is `0` (empty) or `1–7` (color index of a locked piece).
- `PIECES`: the 7 tetrominoes as square matrices; `current`/`next` pieces are `{ type, shape, x, y }`.
- `rotateCW(shape)`: rotates via transpose + row reverse. `tryRotate()` applies this to `current` and attempts wall kicks (`kicks = [0, -1, 1, -2, 2]` column offsets) before giving up.
- `collide(shape, ox, oy)`: bounds + overlap check against `board`, used for movement, rotation, and ghost-piece projection.
- `lockPiece()` → `merge()` (bake piece into `board`) → `clearLines()` (bottom-up scan, splice + unshift empty row) → `spawn()` (promote `next` to `current`, generate new `next`; if the new piece immediately collides, `endGame()` fires).
- `loop(ts)`: driven by `requestAnimationFrame`; accumulates `dt` and drops the piece by one row once `dropAccum >= dropInterval`.
- Scoring: `LINE_SCORES = [0, 100, 300, 500, 800]` × `level`; hard drop = 2 pts/row, soft drop = 1 pt/row. Level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- Ghost piece: `ghostY()` projects `current` straight down; drawn at `globalAlpha = 0.2`.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).

## Conventions

- README and in-repo comments are written in Spanish; keep new documentation consistent with that unless told otherwise.
- No framework/module system — keep additions as plain functions/globals in `game.js` consistent with the existing style, rather than introducing bundlers or ES modules.
