# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript + HTML5 Canvas + CSS. No dependencies, no build step, no tests, no linter. The README (in Spanish) and UI text are in Spanish; keep new user-facing strings in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder (e.g. `python3 -m http.server`) and visit it. There are no build, lint, or test commands.

## Architecture

Three files, all loaded by `index.html` (`game.js` is a plain `<script>`, not a module):

- `index.html` — fixed DOM the script depends on by id: `board` (300×600 canvas = `COLS*BLOCK` × `ROWS*BLOCK`), `next-canvas`, `score`/`lines`/`level`, and `overlay`/`overlay-title`/`overlay-score`/`restart-btn`, and `skin-select`. Changing board size means updating both the canvas attributes and the `COLS`/`ROWS`/`BLOCK` constants in `game.js`.
- `style.css` — layout and theme only.
- `game.js` — all game logic, using module-level mutable globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...) declared on one line and (re)initialized in `init()`.

Key conventions in `game.js`:
- Pieces and board cells use integer type indices 1–7, which index into both `PIECES` (shape matrices) and `COLORS`; `0` is empty.
- Game loop is `requestAnimationFrame` → `loop(ts)`, which accumulates `dropAccum` against `dropInterval` for gravity and redraws every frame. Pause/game-over/restart all work by `cancelAnimationFrame(animId)` and restarting the loop; `init()` is also the restart handler.
- Piece lifecycle: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next`, generates a new one, and calls `endGame()` if the new piece collides immediately.
- `clearLines()` owns level/speed progression (level = lines/10 + 1; `dropInterval = max(100, 1000 - (level-1)*90)`) and line scoring (`LINE_SCORES[n] * level`). Soft drop adds 1 point per cell, hard drop 2 per cell.
- Rotation (`tryRotate`) is clockwise only, with horizontal wall kicks tried in order `[0, -1, 1, -2, 2]`.
- Keyboard input is a single `keydown` listener at the bottom of the file; it ignores input when paused or game over.

- Skins: `SKINS` (retro/neon/pastel/pixel) in `game.js` each provide `colors`, `bg`, `grid` and a `drawBlock`; the global `drawBlock` delegates to `SKINS[currentSkin]`. The choice is stored in `localStorage` key `tetris.skin`; changing it redraws immediately without reloading.
