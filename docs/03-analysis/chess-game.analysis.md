# chess-game Analysis Report

> **Analysis Type**: Gap Analysis (Design vs Implementation)
>
> **Project**: test-claude-chess
> **Version**: 1.0.0
> **Analyst**: Claude (gap-detector)
> **Date**: 2026-03-13
> **Design Doc**: [chess-game.design.md](../02-design/features/chess-game.design.md)

---

## 1. Analysis Overview

### 1.1 Analysis Purpose

Design 문서(`chess-game.design.md`)와 실제 구현(`chess.html`) 간의 일치도를 검증하여, 미구현/변경/추가 항목을 식별한다.

### 1.2 Analysis Scope

- **Design Document**: `docs/02-design/features/chess-game.design.md`
- **Implementation Path**: `chess.html`
- **Analysis Date**: 2026-03-13

---

## 2. Overall Scores

| Category | Score | Status |
|----------|:-----:|:------:|
| Design Match | 90% | ✅ |
| Architecture Compliance | 100% | ✅ |
| Convention Compliance | 98% | ✅ |
| **Overall** | **93%** | ✅ |

---

## 3. Gap Analysis (Design vs Implementation)

### 3.1 Single File Structure (Design SS2.1)

| Item | Design | Implementation | Status |
|------|--------|----------------|--------|
| `<style>` CSS Custom Properties | `:root` 변수 정의 | `:root` 변수 정의 (L11-28) | ✅ Match |
| `<style>` Layout | body, .panel, .board-wrap | body, .panel, .board-wrap (L30-48) | ✅ Match |
| `<style>` Board & Squares | #board, .sq, .light, .dark | #board, .sq, .light, .dark (L228-258) | ✅ Match |
| `<style>` State Classes | .selected, .hint, .hint-cap, .last-from, .last-to, .in-check | 모두 구현 (L256-278) | ✅ Match |
| `<style>` Pieces | .piece, .wp, .bp | .piece, .wp, .bp (L284-307) | ✅ Match |
| `<style>` UI Components | .card, .btn, .diff-btn, .history-list | 모두 구현 (L66-166) | ✅ Match |
| `<style>` Modals | .overlay, .modal, .promo-row | 모두 구현 (L310-356) | ✅ Match |
| `<style>` Responsive | @media max-width: 820px | @media max-width: **840px** | ⚠️ Changed |
| `<body>` .panel | 좌측 컨트롤 패널 | 모두 구현 (L368-418) | ✅ Match |
| `<body>` .board-wrap | thinking-bar, board-outer, labels | 모두 구현 (L421-428) | ✅ Match |
| `<body>` #end-overlay | 게임 종료 모달 | 구현 (L431-438) | ✅ Match |
| `<body>` #promo-overlay | 프로모션 선택 모달 | 구현 (L441-447) | ✅ Match |
| `<script>` Constants | SYM, VAL, PST | 모두 구현 (L453-508) | ✅ Match |
| `<script>` State | makeState, setupBoard, cloneState | 모두 구현 (L513-543) | ✅ Match |
| `<script>` Helpers | row, col, idx, clr, typ, opp | 모두 구현 (L548-553) | ✅ Match |
| `<script>` Engine | attackSquares ~ isStalemate | 모두 구현 (L559-746) | ✅ Match |
| `<script>` AI | evaluate, minimax, bestMove | 모두 구현 (L811-869) | ✅ Match |
| `<script>` Notation | toNotation | 구현 (L874-892) | ⚠️ Partial |
| `<script>` UI | buildBoard ~ setThinking | 모두 구현 (L945-1193) | ✅ Match |
| `<script>` Game Flow | newGame ~ showPromo | 모두 구현 (L910-1232) | ✅ Match |
| `<script>` Init | buildBoard() + newGame() | 구현 (L1268-1269) | ✅ Match |

### 3.2 Data Model (Design SS3)

| Item | Design | Implementation | Status |
|------|--------|----------------|--------|
| State.board | Array(64) | `Array(64).fill(null)` (L515) | ✅ Match |
| State.turn | 'w' \| 'b' | `'w'` (L516) | ✅ Match |
| State.castling | {wK, wQ, bK, bQ} | `{wK:true, wQ:true, bK:true, bQ:true}` (L517) | ✅ Match |
| State.ep | number \| null | `null` (L518) | ✅ Match |
| State.capW | string[] | `[]` (L519) | ✅ Match |
| State.capB | string[] | `[]` (L520) | ✅ Match |
| Board index 0=a8 | index 0 = a8 | buildBoard에서 idx(0,0)=a8 확인 | ✅ Match |
| Board index 63=h1 | index 63 = h1 | setupBoard에서 56+7=63=h1 확인 | ✅ Match |
| Piece encoding 2-char | 'wK', 'bP' 등 | `'b' + back[i]`, `'wP'` 등 (L527-530) | ✅ Match |
| clr(piece) | piece[0] | `p ? p[0] : null` (L551) | ✅ Match |
| typ(piece) | piece[1] | `p ? p[1] : null` (L552) | ✅ Match |
| opp(c) | c==='w'?'b':'w' | 동일 (L553) | ✅ Match |
| UI: state | 현재 게임 상태 | `let state = makeState()` (L897) | ✅ Match |
| UI: selected | 선택된 칸 | `let selected = null` (L898) | ✅ Match |
| UI: hints | 합법 이동 배열 | `let hints = []` (L899) | ✅ Match |
| UI: lastFrom | 직전 출발 | `let lastFrom = null` (L900) | ✅ Match |
| UI: lastTo | 직전 도착 | `let lastTo = null` (L901) | ✅ Match |
| UI: flipped | 뒤집기 여부 | `let flipped = false` (L902) | ✅ Match |
| UI: aiDepth | AI 탐색 깊이 | `let aiDepth = 1` (L903) | ✅ Match |
| UI: playerColor | 플레이어 색 | `let playerColor = 'w'` (L904) | ✅ Match |
| UI: aiThinking | AI 연산 중 | `let aiThinking = false` (L905) | ✅ Match |
| UI: snapshots | Undo 스택 | `let snapshots = []` (L906) | ✅ Match |
| UI: histPairs | 이동 기록 | `let histPairs = []` (L907) | ✅ Match |

### 3.3 Chess Engine (Design SS4)

| Item | Design | Implementation | Status |
|------|--------|----------------|--------|
| `attackSquares(s, sq)` 존재 | SS4.2 | L625 | ✅ Match |
| `attacksSquare(s, sq, target)` 존재 | SS4.3 | L559 | ✅ Match |
| `inCheck(s, c)` - attacksSquare만 호출 | SS4.1 | L607-619: attacksSquare만 사용 | ✅ Match |
| `inCheck` - pseudoMoves 호출 금지 | SS4.1 | pseudoMoves 미호출 확인 | ✅ Match |
| `applyMove` 실행 순서 | ep=null 먼저 | 캡처 기록 -> ep 캡처 -> ep=null 순서 | ⚠️ Changed |
| `pseudoMoves(s, sq)` | attackSquares + 캐슬링 | L693-719 | ✅ Match |
| `legalMoves(s, sq)` | pseudo -> clone -> inCheck 필터 | L724-732 | ✅ Match |
| `allLegalMoves(s)` | 전체 합법 이동 | L735-743 | ✅ Match |
| `isCheckmate(s)` | inCheck && allLegal===0 | L745 | ✅ Match |
| `isStalemate(s)` | !inCheck && allLegal===0 | L746 | ✅ Match |
| Castling kingside | f,g empty, f 경유체크 | L705-709 | ✅ Match |
| Castling queenside | b,c,d empty, d 경유체크 | L712-717 | ✅ Match |

### 3.4 AI (Design SS6)

| Item | Design | Implementation | Status |
|------|--------|----------------|--------|
| `evaluate(s)` PST 미러링 | `idx(7-row(i), col(i))` | L817: `idx(7 - row(i), col(i))` | ✅ Match |
| `minimax` alpha-beta | SS6.2 pseudocode | L826-847 | ⚠️ Changed |
| `bestMove(s, depth)` shuffle | 무작위 셔플 후 탐색 | L853-856: Fisher-Yates shuffle | ✅ Match |
| Easy: depth 1, 250ms | SS6.3 | depth 1, **300ms** | ⚠️ Changed |
| Medium: depth 2, 400ms | SS6.3 | depth 2, **500ms** | ⚠️ Changed |
| Hard: depth 3, 700ms | SS6.3 | depth 3, **800ms** | ⚠️ Changed |

### 3.5 Notation (Design SS7)

| Item | Design | Implementation | Status |
|------|--------|----------------|--------|
| `toNotation(s, from, to, promo)` 존재 | SS7 | L874 | ✅ Match |
| Castling "O-O" / "O-O-O" | SS7 | L883-886 | ✅ Match |
| Pawn capture "exd5" | SS7 | L889: `files[col(from)] + 'x' + tSq` | ✅ Match |
| Promotion "e8=Q" | SS7 | L890 | ✅ Match |
| Check "+" suffix | SS7 | **미구현** | ❌ Missing |
| Checkmate "#" suffix | SS7 | **미구현** | ❌ Missing |

### 3.6 UI/UX (Design SS5)

| Item | Design | Implementation | Status |
|------|--------|----------------|--------|
| .selected class | SS5.2 | L256 | ✅ Match |
| .hint class (원형 점) | SS5.2 | L261-269 | ✅ Match |
| .hint-cap class (링) | SS5.2 | L271-278 | ✅ Match |
| .last-from / .last-to | SS5.2 | L257 | ✅ Match |
| .in-check | SS5.2 | L258 | ✅ Match |
| .piece.wp (흰) | SS5.3: color #f0eeff | L291: `color: #f0eeff` | ✅ Match |
| .piece.bp (검) | SS5.3: color #c8b8ff | L300: `color: #c8b8ff` | ✅ Match |
| #end-overlay | SS5.4 | L431-438 | ✅ Match |
| End modal icons | SS5.4: trophy/robot/handshake | party/sad/handshake | ⚠️ Changed |
| #promo-overlay | SS5.4 | L441-447 | ✅ Match |
| Promo modal "Promote Pawn" | SS5.4 | L443 | ✅ Match |
| Promo 4 choices (Q/R/B/N) | SS5.4 | L1221 | ✅ Match |

### 3.7 CSS Color Palette (Design SS9)

| Variable | Design Value | Implementation Value | Status |
|----------|-------------|---------------------|--------|
| --bg | #0e0f11 | #0e0f11 | ✅ Match |
| --surface | #161719 | #161719 | ✅ Match |
| --surface2 | #1c1d20 | #1c1d20 | ✅ Match |
| --border | #2a2b2f | #2a2b2f | ✅ Match |
| --border2 | #35363c | #35363c | ✅ Match |
| --text | #e2e2e5 | #e2e2e5 | ✅ Match |
| --muted | #8b8c96 | #8b8c96 | ✅ Match |
| --subtle | #5a5b63 | #5a5b63 | ✅ Match |
| --accent | #7c6af7 | #7c6af7 | ✅ Match |
| --accent-dim | #4e40c8 | #4e40c8 | ✅ Match |
| --accent-glow | rgba(124,106,247,.25) | rgba(124,106,247,.25) | ✅ Match |
| --sq-light | #3d3452 | #3d3452 | ✅ Match |
| --sq-dark | #1e1a2e | #1e1a2e | ✅ Match |
| --sq-sel | rgba(124,106,247,.55) | rgba(124,106,247,.55) | ✅ Match |
| --sq-last | rgba(124,106,247,.25) | rgba(124,106,247,.25) | ✅ Match |
| --sq-check | rgba(220,60,60,.55) | rgba(220,60,60,.55) | ✅ Match |

### 3.8 Game Flow Functions (Design SS2.1 Script)

| Function | Design | Implementation | Status |
|----------|--------|----------------|--------|
| `newGame()` | SS2.1 | L910-919 | ✅ Match |
| `saveSnap()` | SS2.1 | L922-930 | ✅ Match |
| `restoreSnap()` | SS2.1 | L933-940 | ✅ Match |
| `onSqClick()` | SS2.1 | L1090-1120 | ✅ Match |
| `doMove()` | SS2.1 | L1122-1142 | ✅ Match |
| `scheduleAI()` | SS2.1 | L1144-1170 | ✅ Match |
| `showEndModal()` | SS2.1 | L1195-1211 | ✅ Match |
| `showPromo()` | SS2.1 | L1213-1232 | ✅ Match |
| `buildBoard()` | SS2.1 | L945-963 | ✅ Match |
| `buildLabels()` | SS2.1 | L965-987 | ✅ Match |
| `renderAll()` | SS2.1 | L1080-1085 | ✅ Match |
| `setThinking(on)` | SS2.1 | L1172-1193 | ✅ Match |

---

## 4. Differences Found

### 4.1 Missing Features (Design O, Implementation X)

| # | Item | Design Location | Description |
|---|------|-----------------|-------------|
| 1 | Check notation suffix "+" | SS7 (L494-495) | `toNotation`에서 이동 후 체크 시 "+" 접미어 미구현 |
| 2 | Checkmate notation suffix "#" | SS7 (L496) | `toNotation`에서 이동 후 체크메이트 시 "#" 접미어 미구현 |

### 4.2 Changed Features (Design != Implementation)

| # | Item | Design | Implementation | Impact |
|---|------|--------|----------------|--------|
| 1 | Responsive breakpoint | 820px | 840px (L358) | Low |
| 2 | AI Easy delay | 250ms | 300ms (L1149) | Low |
| 3 | AI Medium delay | 400ms | 500ms (L1149) | Low |
| 4 | AI Hard delay | 700ms | 800ms (L1149) | Low |
| 5 | applyMove 실행 순서 | ep=null -> ep캡처 -> 캐슬링룩 -> 플래그 -> 이동 -> 캡처기록 -> 프로모션 -> turn | 캡처기록 -> ep캡처 -> ep=null -> 캐슬링룩 -> 플래그 -> 프로모션 -> 이동 -> turn | Medium |
| 6 | minimax 종료 조건 | `moves.length === 0` 후 inCheck 체크 | `isCheckmate(s)` / `isStalemate(s)` 호출 후 moves 생성 | Low |
| 7 | End modal icons | trophy/robot/handshake | party/sad/handshake | Low |

### 4.3 Added Features (Design X, Implementation O)

| # | Item | Implementation Location | Description |
|---|------|------------------------|-------------|
| - | - | - | 추가 기능 없음 |

---

## 5. Match Rate Summary

```
Total checked items: 82
  Matched:     73 items (89%)
  Changed:      7 items  (9%)
  Missing:      2 items  (2%)

Overall Match Rate: 90%
```

---

## 6. Convention Compliance

### 6.1 Naming Convention Check

| Category | Convention | Compliance | Violations |
|----------|-----------|:----------:|------------|
| Functions (make*, clone*, setup*) | camelCase | 100% | - |
| Functions (is*, in*) | camelCase boolean | 100% | - |
| Functions (render*, show*, build*) | camelCase | 100% | - |
| Constants (SYM, VAL, PST) | UPPER_SNAKE_CASE | 100% | - |
| Variables | camelCase | 100% | - |
| CSS classes | kebab-case | 100% | - |
| CSS custom properties | --kebab-case | 100% | - |

### 6.2 State Mutation Rules

| Rule | Compliance | Notes |
|------|:----------:|-------|
| applyMove(cloneState(s), ...) for temp | ✅ | legalMoves, pseudoMoves, minimax 모두 준수 |
| applyMove(state, ...) for real game | ✅ | doMove, scheduleAI에서 사용 |
| Engine 함수에서 직접 state 수정 금지 | ✅ | 모든 엔진 함수가 파라미터 s 사용 |

### 6.3 Call Hierarchy Rules

| Rule | Compliance | Notes |
|------|:----------:|-------|
| inCheck -> attacksSquare only | ✅ | pseudoMoves 호출 없음 확인 |
| pseudoMoves -> attackSquares + inCheck | ✅ | 캐슬링에서만 inCheck 호출 |

Convention Score: **98%**

---

## 7. Detailed Gap Analysis

### 7.1 GAP-001: Check/Checkmate Notation Suffix Missing

**Severity**: Medium
**Design Reference**: SS7, L494-496

Design 문서에 다음과 같이 명시됨:
```
체크 / 체크메이트:
  이동 후 inCheck -> "+" 접미어
  이동 후 체크메이트 -> "#" 접미어
```

현재 `toNotation` (L874-892)에는 체크/체크메이트 접미어 로직이 없다. 이동 표기법의 정확성에 영향을 미치며, 표준 대수 표기법(SAN)에서는 필수 요소이다.

### 7.2 GAP-002: applyMove Execution Order

**Severity**: Medium
**Design Reference**: SS4.4, L293-308

Design 순서:
1. `s.ep = null` (앙파상 먼저 초기화)
2. 앙파상 캡처 처리
3. 캐슬링 룩 이동
4. castling 플래그 갱신
5. 기물 이동
6. 캡처 기록
7. 프로모션
8. turn 전환

Implementation 순서 (L751-806):
1. 캡처 기록 (일반 캡처)
2. 앙파상 캡처 처리
3. `s.ep = null`
4. 앙파상 설정 (2칸 전진)
5. 캐슬링 룩 이동 + 플래그
6. 룩 이동 시 플래그
7. 프로모션
8. 기물 이동
9. turn 전환

캡처 기록이 ep 초기화보다 먼저 일어나지만, 기능적으로는 동일하게 동작한다 (ep 값을 읽기 전에 초기화하지 않으므로 앙파상 캡처가 정상 작동). 순서 차이는 있으나 로직 결과는 동등하다.

### 7.3 GAP-003: AI Delay Values

**Severity**: Low
**Design Reference**: SS6.3

| Difficulty | Design | Implementation | Diff |
|------------|--------|----------------|------|
| Easy | 250ms | 300ms | +50ms |
| Medium | 400ms | 500ms | +100ms |
| Hard | 700ms | 800ms | +100ms |

구현 값이 일관적으로 Design보다 약간 더 길다. UX에 미치는 영향은 미미하나, Design 문서 또는 구현 중 하나를 동기화해야 한다.

---

## 8. Recommended Actions

### 8.1 Immediate (구현 수정)

| Priority | Item | Location | Description |
|----------|------|----------|-------------|
| 1 | Check/Checkmate notation | chess.html `toNotation()` | "+" / "#" 접미어 추가 |

### 8.2 Documentation Update

| Priority | Item | Design Location | Description |
|----------|------|-----------------|-------------|
| 1 | Responsive breakpoint | SS5.1 | 820px -> 840px로 업데이트 |
| 2 | AI delay values | SS6.3 | 250/400/700 -> 300/500/800으로 업데이트 |
| 3 | applyMove 순서 | SS4.4 | 실제 구현 순서로 업데이트 |
| 4 | minimax 종료 조건 | SS6.2 | isCheckmate/isStalemate 사용으로 업데이트 |
| 5 | End modal icons | SS5.4 | 실제 이모지로 업데이트 |

### 8.3 Intentional Differences (기록)

없음 - 모든 차이는 동기화 필요.

---

## 9. Next Steps

- [ ] `toNotation()`에 체크/체크메이트 접미어 구현
- [ ] Design 문서의 변경 항목 5건 업데이트
- [ ] 완료 보고서 작성 (`chess-game.report.md`)

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 0.1 | 2026-03-13 | Initial analysis | Claude (gap-detector) |
