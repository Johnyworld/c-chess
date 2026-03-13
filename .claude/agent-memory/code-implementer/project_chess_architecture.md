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

## Rendering: Canvas (chess.html)

chess.html uses `<canvas id="board">` with `SQ = 72` (px per square), total `576×576px`.

4-layer render order in `renderCanvas()`:
1. `drawBoard(ctx)` — fills each square with hardcoded hex (`'#1e1a2e'` dark, `'#3d3452'` light)
2. `drawOverlays(ctx)` — last-move, selected, hint dots/rings, in-check king highlight
3. `drawPieces(ctx)` — all static pieces (skipping `animFrom` set)
4. `drawAnimatingPiece(x, y, piece)` — called during rAF loop, draws moving piece on top

Helper layout:
- `sqX(i)`, `sqY(i)`: top-left pixel of square `i`, flipped-aware
- `sqCenterX/Y(i)`: center pixel
- `fillSq(ctx, i, color)`: fill whole square with color
- `pixelToSq(x, y)`: canvas-relative click → logical board index (flipped-aware)

`animFrom` is a `Set<number>` of board indices currently being animated. `drawPieces` skips any index in this set. For castling, king and rook are added/deleted as their animations start/complete.

## Piece move animation (chess.html)

`animatePiece(from, to, duration, onComplete)` — requestAnimationFrame loop, ease-out-quad easing (`1-(1-p)²`). Calls `renderCanvas()` each frame then `drawAnimatingPiece` on top.

Key design decisions:
- `animFrom.add(from)` at start; `animFrom.delete(from)` at completion. `animating = false` only when `animFrom.size === 0`.
- `doMove` / `scheduleAI` defer `applyMove` + `renderAll` until after animation completes (inside `finish` callback). Board state mutated only after piece visually arrives.
- Castling: king first (220ms player / 280ms AI), then rook (180ms / 200ms), chained via callback. Both can be in `animFrom` simultaneously during the brief transition between animations.
- `isCastling(s, from, to)` must be called BEFORE `applyMove` (state still has original king position).
- `getCastlingRook(color, toCol)` returns `[rookFrom, rookTo]` for white/black kingside (toCol=6) and queenside (toCol=2/3).
- Click handled by `onCanvasClick` (mousedown), uses `pixelToSq` to convert event coords to board index. Blocked when `animating || aiThinking`.
