---
name: chess_architecture
description: Core architecture decisions, function responsibilities, and known gotchas in the single-file chess engine
type: project
---

Two files coexist in the repo: `index.html` (original) and `chess.html` (new implementation per spec). Both are fully functional single-file chess games.

**Why:** User requested a new `chess.html` with a different color palette and slightly different UI layout from `index.html`.

**How to apply:** When editing either file, treat them as independent implementations. Do not merge them.

## Critical architecture invariant: recursion safety

`attacksSquare(s, sq, target)` — called by `inCheck()` only. Must never call `pseudoMoves`, `legalMoves`, or `inCheck`. It is a pure geometric + blocker check with no board-state recursion.

`attackSquares(s, sq)` — returns list of target squares for normal moves (including en passant for pawns, excluding castling for kings). Called by `pseudoMoves`.

`pseudoMoves(s, sq)` — calls `attackSquares` + appends castling moves. Castling check internally calls `applyMove` on a clone then `inCheck` — this is safe because `inCheck` only calls `attacksSquare`, not `pseudoMoves`.

`legalMoves(s, sq)` — filters `pseudoMoves` by cloning state, applying each move, checking `inCheck`. Safe.

## Board encoding

- Flat `Array(64)`, index 0 = a8 (top-left), index 63 = h1 (bottom-right)
- `row(i) = (i/8)|0`, `col(i) = i%8`, `idx(r,c) = r*8+c`
- Pieces: 2-char strings `'wK'`, `'bP'`, etc.
- `s.ep`: en passant target square index (the square the capturing pawn lands on), or `null`

## applyMove ep handling

Reset `s.ep = null` FIRST, then set if double pawn push. This ensures ep is never stale.

## Color palette difference between files

- `index.html`: `--accent: #7c5cfc`, `--sq-light: #2e2e40`, `--sq-dark: #1a1a26`
- `chess.html`: `--accent: #7c6af7`, `--sq-light: #3d3452`, `--sq-dark: #1e1a2e`

## Piece move animation (chess.html)

Added in chess.html: `animatePiece(from, to, duration, onComplete)` — clones the `.piece` element to `document.body` as `position:fixed`, slides it via CSS `transform: translate(dx,dy)` with `cubic-bezier`, then calls `onComplete` in `transitionend`. Uses a global `animating` flag (same pattern as `aiThinking`) to block user input during animation.

Key design decisions:
- `doMove` / `scheduleAI` now defer `applyMove` + `renderAll` until after animation completes (inside `finish` callback). This means board state is mutated only after the piece visually arrives.
- Castling animates king first (220ms player / 280ms AI), then rook (180ms / 200ms), each as a separate `animatePiece` call chained via callback.
- `isCastling(s, from, to)` checks king type + column distance 2. Must be called BEFORE `applyMove` (state still has original king position).
- `getCastlingRook(color, toCol)` returns `[rookFrom, rookTo]` logical board indices for white/black kingside (toCol=6) and queenside (toCol=2/3).
