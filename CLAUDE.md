# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file vanilla HTML/CSS/JS chess game — no build step, no dependencies, no package manager. Open `index.html` directly in a browser to run.

## Architecture

Everything lives in `index.html` in three sections:

**CSS (Linear dark-theme design)**
- CSS custom properties on `:root` define the full color palette (`--accent`, `--sq-light`, `--sq-dark`, etc.)
- Board squares use `.light` / `.dark` base classes, with state classes layered on top: `selected`, `hint`, `hint-cap`, `last-from`, `last-to`, `in-check`
- Piece visibility: white pieces use `color: #f0eeff` + black drop-shadow filter; black pieces use `color: #c8b8ff` + black drop-shadow

**Chess Engine (pure functions, no classes)**
- Board is a flat `Array(64)` — index 0 = a8 (top-left), index 63 = h1 (bottom-right)
- Pieces encoded as 2-char strings: `'wK'`, `'bP'`, etc.
- Key separation: `attacksSquare(s, sq, target)` is the non-recursive attack detector used by `inCheck()`. This avoids infinite recursion that would occur if castling logic called `inCheck` which called pseudo-moves which called `inCheck` again.
- `pseudoMoves(s, sq)` calls `attackSquares` for basic moves, then appends castling moves for kings
- `legalMoves(s, sq)` filters pseudo-moves by cloning state and checking `inCheck` after each move
- AI uses minimax with alpha-beta pruning (`minimax`) + piece-square tables (`PST`)

**UI**
- `buildBoard()` creates 64 `.sq` divs with `data-sq` = logical board index; call this on init and on flip
- `renderAll()` = `renderSquares()` + `renderStatus()` + `renderHistory()` + `renderCaptured()`
- `snapshots[]` array holds cloned states for undo; `doMove()` and `scheduleAI()` each call `saveSnap()` before mutating state
- `histPairs[]` tracks move history as `{num, white, black}` pairs for algebraic notation display
- AI runs in `setTimeout` inside `scheduleAI()` to avoid blocking the UI thread
