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
