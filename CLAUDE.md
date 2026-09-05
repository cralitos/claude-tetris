# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris. No build step, no package manager, no external dependencies — three files (`index.html`, `style.css`, `game.js`) that run directly in a browser.

## Running the game

There is no build/lint/test tooling. To run:

```bash
start index.html       # Windows: open directly
# or serve it (needed if browser blocks local file access for canvas/scripts)
python3 -m http.server 8000
npx serve .
```

Then open `http://localhost:8000`. There are no automated tests to run.

## Architecture

All game logic lives in `game.js` (single file, no modules). Key pieces, in the order the code depends on them:

- **Board model**: `board` is a `ROWS × COLS` matrix (20×10). Each cell is `0` (empty) or an integer `1–7` indexing into `COLORS`/`PIECES` for that piece's color.
- **Pieces**: `PIECES` defines all 7 tetrominoes as square matrices. Rotation (`rotateCW`) is transpose + reverse, not precomputed rotation states.
- **Collision** (`collide`): the single source of truth for whether a shape can occupy a position — checks board bounds and existing filled cells. Used by movement, rotation, ghost-piece projection, and spawn-collision (game over) checks alike.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns before giving up on the rotation.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and drops the piece one row once `dropInterval` is exceeded. Pausing/resuming cancels/restarts this loop via `animId` rather than tracking a paused-time offset.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears completed rows (shifting from the bottom up, re-checking the same row index after a splice), then spawns the next piece.
- **Scoring/leveling**: `LINE_SCORES` table (`[0,100,300,500,800]`) multiplied by `level`; hard drop adds 2 points/row dropped, soft drop adds 1 point/row. Level increases every 10 cleared lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering**: `draw()` clears and redraws the whole board each frame (no dirty-rect tracking) — grid lines, locked blocks, the ghost piece (`globalAlpha = 0.2`, computed via `ghostY()`), then the current piece on top. `drawNext()` renders the preview piece to a separate canvas/context.
- All game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) is module-level `let` — there's no encapsulating class/object. `init()` resets all of it and is also the restart-button handler.

If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, also update the `<canvas id="board">` `width`/`height` attributes in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).

The README (`README.md`) is in Spanish and documents controls, mechanics, and tunable constants in more detail — consult it for gameplay/UX intent before changing behavior.
