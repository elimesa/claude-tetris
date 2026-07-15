# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A vanilla-JS implementation of Tetris using the HTML5 Canvas 2D API. No build step, no dependencies, no package manager, no tests. Three source files: `index.html` (DOM + two canvases), `style.css` (dark arcade theme), `game.js` (all game logic, ~300 lines).

UI strings shown to the player are in Spanish (e.g. `GAME OVER` overlay shows `Puntuación:`, controls list uses `mover`/`rotar`); keep new user-facing text consistent with that.

## Running

No install or compile. Open `index.html` directly, or serve statically (recommended so paths resolve cleanly):

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

There is no lint or test tooling — verify changes by playing the game in a browser.

## Architecture (game.js)

Everything hangs off module-level mutable state declared on line ~43 (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars). `init()` resets all of it and is also the restart handler.

Key representations:
- **Board**: `ROWS × COLS` array of ints. `0` = empty; `1–7` = a color/piece-type index into `COLORS` and `PIECES` (both are 1-indexed with a `null` at index 0).
- **Piece**: `{ type, shape, x, y }` where `shape` is a small square matrix of those same indices. Rotation is transpose-then-reverse-rows (`rotateCW`), producing a brand-new matrix each time.

Core interactions to know before editing:
- `collide(shape, x, y)` is the single source of truth for legality — movement, rotation, drops, ghost projection, and game-over detection all call it. Cells above the board (`ny < 0`) are intentionally allowed so pieces can spawn/rotate partly off the top.
- `tryRotate()` implements basic wall kicks: it tries horizontal offsets `[0, -1, 1, -2, 2]` and applies the first non-colliding one, else abandons the rotation.
- **Game loop** (`loop`) is `requestAnimationFrame`-driven and time-accumulator based (`dropAccum` vs `dropInterval`), so it is framerate-independent. Pause works by cancelling the frame and is resumed by restarting the loop with a fresh `lastTime`.
- Locking a piece is always `lockPiece()` → `merge()` (stamp into board) → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current`, generates a new `next`, and calls `endGame()` if the freshly spawned piece already collides.

Scoring / difficulty coupling (`clearLines`): `LINE_SCORES[cleared] * level`; `level = floor(lines/10)+1`; and `dropInterval = max(100, 1000 - (level-1)*90)`. These recompute together — changing one usually means revisiting the others.

## Gotcha: canvas dimensions are hard-coded in HTML

`COLS`, `ROWS`, `BLOCK` live in `game.js`, but the board canvas `width`/`height` (`300 × 600`) are hard-coded in `index.html`. If you change any of those constants you must update `<canvas id="board">` to `COLS*BLOCK × ROWS*BLOCK`, or rendering will be clipped/scaled.
