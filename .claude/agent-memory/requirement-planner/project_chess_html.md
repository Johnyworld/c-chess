---
name: project_chess_html
description: Chess game project context — index.html is fully implemented, user wants chess.html as a new deliverable
type: project
---

index.html is a complete, production-quality chess implementation (1192 lines). It includes: full chess rules, castling/en-passant/promotion, minimax AI with alpha-beta pruning, PST evaluation, move history, undo, captured pieces display, board flip, difficulty levels (Easy/Medium/Hard = depth 1/2/3), Linear dark theme UI.

User's first request (2026-03-13): create chess.html as a new single-file implementation following the same architecture. The request is for an implementation plan, not the code itself.

**Why:** User wants a standalone chess.html separate from the existing index.html, likely for comparison or as a clean deliverable.
**How to apply:** When planning chess.html, treat index.html as the reference implementation. The plan should map directly to patterns already proven in index.html rather than inventing new approaches.
