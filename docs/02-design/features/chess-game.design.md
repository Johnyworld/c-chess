# chess-game Design Document

> **Summary**: 단일 `chess.html` 파일 안에서 동작하는 체스 엔진·AI·UI의 상세 설계
>
> **Project**: test-claude-chess
> **Version**: 1.0.0
> **Author**: User
> **Date**: 2026-03-13
> **Status**: Draft
> **Planning Doc**: [chess-game.plan.md](../01-plan/features/chess-game.plan.md)

### Pipeline References

| Phase   | Document           | Status            |
| ------- | ------------------ | ----------------- |
| Phase 1 | Schema Definition  | N/A (단일 파일)   |
| Phase 2 | Coding Conventions | N/A (단일 파일)   |
| Phase 3 | Mockup             | N/A (구현 완료)   |
| Phase 4 | API Spec           | N/A (백엔드 없음) |

---

## 1. Overview

### 1.1 Design Goals

- **완전성**: 표준 체스 규칙(FIDE)을 완벽히 구현 — 불법 이동 차단, 체크메이트·스테일메이트 정확 감지
- **단순성**: 단일 파일, 외부 의존성 없음 — 브라우저에서 파일만 열면 즉시 실행
- **안전성**: `inCheck` ↔ 캐슬링 무한재귀 방지를 위한 명확한 함수 호출 계층 유지
- **반응성**: AI 연산을 `setTimeout`으로 격리, UI 블로킹 없음

### 1.2 Design Principles

- **순수 함수 우선**: 엔진 함수들은 상태를 직접 변이하지 않고 clone 사용 (단, `applyMove` 제외)
- **단일 책임**: 각 함수는 하나의 역할만 수행 (이동 생성 / 이동 적용 / 렌더링 분리)
- **호출 계층 엄수**: `inCheck` → `attacksSquare` only (캐슬링 포함 `pseudoMoves` 호출 절대 금지)
- **상태 불변성 우선**: `cloneState` 후 `applyMove`로 가상 이동 검증

---

## 2. Architecture

### 2.1 단일 파일 구조도

```
chess.html
├── <head>
│   └── Google Fonts (Inter) link
├── <style>
│   ├── CSS Custom Properties (:root)
│   ├── Layout (body, .panel, .board-wrap)
│   ├── Board Canvas (#board — canvas 요소)
│   ├── UI Components (.card, .btn, .diff-btn, .history-list)
│   ├── Modals (.overlay, .modal, .promo-row)
│   └── Responsive (@media max-width: 820px)
├── <body>
│   ├── .panel (좌측 컨트롤)
│   │   ├── .logo
│   │   ├── .card "Status" (#dot, #status-text, #turn-icon, #turn-label)
│   │   ├── .card "Difficulty" (.diff-btn ×3)
│   │   ├── .card "Actions" (#btn-new, #btn-undo, #btn-flip)
│   │   ├── .card "Captured by White" (#cap-w)
│   │   ├── .card "Captured by Black" (#cap-b)
│   │   └── .card "Move History" (#history)
│   ├── .board-wrap
│   │   ├── .thinking-bar#tbar > .thinking-fill#tfill
│   │   └── .board-outer
│   │       ├── .rank-labels#rlabels
│   │       ├── <canvas id="board">  ← DOM div 대신 Canvas 사용
│   │       └── .file-labels#flabels
│   ├── #end-overlay (게임 종료 모달)
│   └── #promo-overlay (프로모션 선택 모달)
└── <script>
    ├── [Constants]   SYM, VAL, PST, SQ (칸 크기 상수)
    ├── [State]       makeState, setupBoard, cloneState
    ├── [Helpers]     row, col, idx, clr, typ, opp
    ├── [Engine]      attackSquares, attacksSquare, inCheck
    │                 applyMove, pseudoMoves, legalMoves
    │                 allLegalMoves, isCheckmate, isStalemate
    ├── [AI]          evaluate, minimax, bestMove
    ├── [Notation]    toNotation
    ├── [Canvas UI]   initCanvas, renderAll
    │                 drawBoard, drawPieces, drawOverlays
    │                 drawAnimatingPiece, pixelToSq
    │                 renderStatus, renderHistory, renderCaptured
    │                 setThinking
    ├── [Animation]   animatePiece, isCastling, getCastlingRook
    ├── [Game Flow]   newGame, saveSnap, restoreSnap
    │                 onCanvasClick, doMove, scheduleAI
    │                 showEndModal, showPromo
    └── [Init]        initCanvas() + newGame()
```

### 2.2 데이터 흐름

```
사용자 클릭 (Canvas mousedown)
    │
    ▼
onCanvasClick(e)
    │  pixelToSq(x, y) → sq
    │
    ├─ [말 선택] → legalMoves(state, sq) → hints 저장 → renderAll()
    │
    └─ [이동 실행] ──┬─ [프로모션?] → showPromo() → 선택 후 doMove()
                    │
                    └─ doMove(from, to, promo)
                            │
                            ├─ saveSnap()
                            ├─ toNotation() → histPairs 갱신
                            ├─ animatePiece(from, to, 180ms, finish)
                            │       └─ requestAnimationFrame 루프
                            │            └─ drawAnimatingPiece (ease-out-quad)
                            └─ finish():
                                    ├─ applyMove(state, from, to, promo)
                                    ├─ renderAll()
                                    ├─ [게임 종료?] → showEndModal()
                                    └─ [AI 차례?] → scheduleAI()
                                                        │
                                                        ├─ setThinking(true)
                                                        ├─ setTimeout(fn, delay)
                                                        │       │
                                                        │       ├─ bestMove(state, depth)
                                                        │       │       └─ minimax() 재귀
                                                        │       ├─ saveSnap()
                                                        │       └─ animatePiece(from, to, 280ms, finish)
                                                        └─ setThinking(false)
```

### 2.3 함수 의존성

| 함수            | 의존하는 함수                                                    | 주의사항                            |
| --------------- | ---------------------------------------------------------------- | ----------------------------------- |
| `inCheck`       | `attacksSquare`                                                  | `pseudoMoves` 절대 호출 금지        |
| `pseudoMoves`   | `attackSquares`, `inCheck`                                       | 캐슬링 로직에서 `inCheck` 호출 가능 |
| `legalMoves`    | `pseudoMoves`, `cloneState`, `applyMove`, `inCheck`              |                                     |
| `allLegalMoves` | `legalMoves`                                                     |                                     |
| `minimax`       | `allLegalMoves`, `cloneState`, `applyMove`, `evaluate`           | 재귀 함수                           |
| `bestMove`      | `allLegalMoves`, `minimax`                                       |                                     |
| `doMove`        | `saveSnap`, `toNotation`, `applyMove`, `renderAll`, `scheduleAI` |                                     |
| `scheduleAI`    | `bestMove`, `saveSnap`, `applyMove`, `renderAll`                 | setTimeout 내부 실행                |

---

## 3. 데이터 모델

### 3.1 게임 상태 (State Object)

```js
// makeState() 반환값
{
  board: Array(64),      // null 또는 'wK', 'bP' 등 2글자 문자열
  turn: 'w' | 'b',      // 현재 차례
  castling: {
    wK: boolean,         // 흰 킹사이드 캐슬링 가능 여부
    wQ: boolean,         // 흰 퀸사이드 캐슬링 가능 여부
    bK: boolean,         // 검 킹사이드 캐슬링 가능 여부
    bQ: boolean,         // 검 퀸사이드 캐슬링 가능 여부
  },
  ep: number | null,     // 앙파상 가능한 타깃 칸 인덱스 (없으면 null)
  capW: string[],        // 흰이 잡은 기물 배열 ('bP', 'bR' 등)
  capB: string[],        // 검이 잡은 기물 배열 ('wP', 'wN' 등)
}
```

### 3.2 보드 인덱스 매핑

```
Board Index Layout (flat Array(64)):
  a    b    c    d    e    f    g    h
┌────┬────┬────┬────┬────┬────┬────┬────┐
│  0 │  1 │  2 │  3 │  4 │  5 │  6 │  7 │  rank 8
├────┼────┼────┼────┼────┼────┼────┼────┤
│  8 │  9 │ 10 │ 11 │ 12 │ 13 │ 14 │ 15 │  rank 7
├────┼────┼────┼────┼────┼────┼────┼────┤
│ 16 │ 17 │ 18 │ 19 │ 20 │ 21 │ 22 │ 23 │  rank 6
├────┼────┼────┼────┼────┼────┼────┼────┤
│ 24 │ 25 │ 26 │ 27 │ 28 │ 29 │ 30 │ 31 │  rank 5
├────┼────┼────┼────┼────┼────┼────┼────┤
│ 32 │ 33 │ 34 │ 35 │ 36 │ 37 │ 38 │ 39 │  rank 4
├────┼────┼────┼────┼────┼────┼────┼────┤
│ 40 │ 41 │ 42 │ 43 │ 44 │ 45 │ 46 │ 47 │  rank 3
├────┼────┼────┼────┼────┼────┼────┼────┤
│ 48 │ 49 │ 50 │ 51 │ 52 │ 53 │ 54 │ 55 │  rank 2
├────┼────┼────┼────┼────┼────┼────┼────┤
│ 56 │ 57 │ 58 │ 59 │ 60 │ 61 │ 62 │ 63 │  rank 1
└────┴────┴────┴────┴────┴────┴────┴────┘

index 0 = a8 (top-left)
index 63 = h1 (bottom-right)

Helper functions:
  row(i) = Math.floor(i / 8)   // 0 = 8th rank, 7 = 1st rank
  col(i) = i % 8               // 0 = a-file, 7 = h-file
  idx(r, c) = r * 8 + c
```

### 3.3 기물 인코딩

```js
// 2글자 문자열: [색][타입]
'wK' = 흰 킹    'bK' = 검 킹
'wQ' = 흰 퀸    'bQ' = 검 퀸
'wR' = 흰 룩    'bR' = 검 룩
'wB' = 흰 비숍  'bB' = 검 비숍
'wN' = 흰 나이트 'bN' = 검 나이트
'wP' = 흰 폰    'bP' = 검 폰

// 헬퍼
clr(piece) = piece[0]   // 'w' | 'b'
typ(piece) = piece[1]   // 'K' | 'Q' | 'R' | 'B' | 'N' | 'P'
opp(c) = c === 'w' ? 'b' : 'w'
```

### 3.4 UI 상태 변수

```js
let state; // 현재 게임 상태 (State Object)
let selected; // 선택된 칸 인덱스 (null = 없음)
let hints; // 합법 이동 가능 칸 인덱스 배열
let lastFrom; // 직전 이동 출발 칸
let lastTo; // 직전 이동 도착 칸
let flipped; // 보드 뒤집기 여부 (boolean)
let aiDepth; // AI 탐색 깊이 (1 | 2 | 3)
let playerColor; // 플레이어 색 ('w')
let aiThinking; // AI 연산 중 여부 (boolean)
let snapshots; // Undo용 스냅샷 스택 (배열)
let histPairs; // 이동 기록 [{num, white, black}]
```

---

## 4. 체스 엔진 설계

### 4.1 함수 호출 계층 (핵심 아키텍처 규칙)

```
                    ┌─────────────────────────────────┐
                    │  CALL HIERARCHY (절대 준수)      │
                    └─────────────────────────────────┘

legalMoves(s, sq)
  └─ pseudoMoves(s, sq)
       ├─ attackSquares(s, sq)      ← 일반 이동 (캐슬링 제외)
       │    └─ (이동 생성 로직)
       └─ [킹이면] 캐슬링 체크
            └─ inCheck(s, c)        ← 캐슬링 경유 칸 검증
                 └─ attacksSquare(s, enemy, kingPos)  ← 공격 여부만

inCheck(s, c)
  └─ attacksSquare(s, sq, target)   ← 절대로 pseudoMoves 호출 금지!
```

### 4.2 `attackSquares(s, sq)` 설계

일반 이동 목록 반환 (캐슬링 제외, 앙파상 포함).

```
입력: state, 출발 칸 인덱스
출력: 이동 가능 칸 인덱스 배열

내부 헬퍼:
  tryPush(nr, nc): 보드 범위 내, 아군 말 없는 칸만 추가
  slide(dr, dc): 슬라이딩 이동 (빈 칸 계속, 적 말 캡처 후 중단)

폰 이동:
  dir = (clr==='w') ? -1 : +1   (흰=위로, 검=아래로)
  startRow = (clr==='w') ? 6 : 1
  - 1칸 전진: (r+dir, c) 비어있을 때
  - 2칸 전진: (r+2*dir, c) r===startRow && 경유 칸 비어있을 때
  - 대각 캡처: (r+dir, c±1) 적 말 있거나 s.ep === idx(r+dir, c±1)

나이트: 8방향 L자 tryPush
비숍: 4 대각선 slide
룩: 4 직선 slide
퀸: 8방향 slide
킹: 1칸 8방향 tryPush (캐슬링 제외)
```

### 4.3 `attacksSquare(s, sq, target)` 설계

`sq`의 기물이 `target` 칸을 공격하는지 bool 반환.
**`inCheck`에서만 호출**, 재귀 없음.

```
폰:   dr === dir && |dc| === 1  (전진 방향 대각선만)
나이트: |dr|+|dc| === 3 && |dr| >= 1 && |dc| >= 1
킹:   |dr| <= 1 && |dc| <= 1
비숍/퀸(대각): |dr| === |dc|, 경로 상 장애물 없음
룩/퀸(직선):  dr===0 || dc===0, 경로 상 장애물 없음
```

### 4.4 `applyMove(s, from, to, promo)` 설계

State를 직접 변이 (clone 없이).

```
실행 순서 (반드시 준수):
1. s.ep = null                           ← 앙파상 먼저 초기화
2. 앙파상 캡처 처리
   - 이동 기물이 폰 && to === 이전 s.ep → 경유 칸의 적 폰 제거
3. 앙파상 설정
   - 폰이 2칸 전진 → s.ep = idx((r1+r2)/2, c1)
4. 캐슬링 룩 이동
   - 킹이 2칸 이동 → 룩을 반대편으로 이동
5. castling 플래그 갱신
   - 킹 이동 → 양쪽 false
   - a1/h1/a8/h8 룩 이동 → 해당 side false
6. 기물 이동: s.board[to] = s.board[from]; s.board[from] = null
7. 캡처 기록: 적 말이 있었으면 capW 또는 capB에 추가
8. 프로모션: r2===0 || r2===7 && 폰 → c+'Q' (또는 선택한 기물)
9. s.turn = opp(s.turn)
```

### 4.5 캐슬링 조건 설계

```
킹사이드 (O-O):
  조건: castling[c+'K'] === true
        board[f-file] === null && board[g-file] === null
        !inCheck(s, c)                    ← 현재 체크 아님
        !inCheck(after king→f, c)         ← 경유 칸(f) 안전
        → g-file 추가 (legalMoves가 체크 필터 처리)

퀸사이드 (O-O-O):
  조건: castling[c+'Q'] === true
        board[d], board[c], board[b] === null
        !inCheck(s, c)
        !inCheck(after king→d, c)         ← 경유 칸(d) 안전
        → c-file 추가

인덱스 참조:
  흰: e1=60, f1=61, g1=62, d1=59, c1=58, b1=57, h1=63, a1=56
  검: e8=4,  f8=5,  g8=6,  d8=3,  c8=2,  b8=1,  h8=7,  a8=0
```

---

## 5. UI/UX 설계

### 5.1 화면 레이아웃

```
┌──────────────────────────────────────────────────────────┐
│  body (flex, row, gap:24px, justify:center, align:start) │
│                                                          │
│  ┌─────────────┐    ┌──────────────────────────────┐    │
│  │   .panel    │    │       .board-wrap             │    │
│  │  (236px)    │    │                               │    │
│  │             │    │  ┌─ thinking-bar ──────────┐  │    │
│  │ ♟ Chess     │    │  │ [████████░░░░░░░░░░░░░] │  │    │
│  │             │    │  └──────────────────────────┘  │    │
│  │ [Status]    │    │                               │    │
│  │  ● Active   │    │  8 ┌──┬──┬──┬──┬──┬──┬──┬──┐ │    │
│  │  White turn │    │  7 │  │  │  │  │  │  │  │  │ │    │
│  │             │    │  6 │  │  │  │  │  │  │  │  │ │    │
│  │ [Difficulty]│    │  5 │  │  │  │  │  │  │  │  │ │    │
│  │ Easy Med Hard    │  4 │  │  │  │  │  │  │  │  │ │    │
│  │             │    │  3 │  │  │  │  │  │  │  │  │ │    │
│  │ [Actions]   │    │  2 │  │  │  │  │  │  │  │  │ │    │
│  │ New Game    │    │  1 └──┴──┴──┴──┴──┴──┴──┴──┘ │    │
│  │ Undo        │    │     a  b  c  d  e  f  g  h   │    │
│  │ Flip Board  │    │                               │    │
│  │             │    └──────────────────────────────┘    │
│  │ [Cap White] │                                        │
│  │ [Cap Black] │                                        │
│  │ [History]   │                                        │
│  └─────────────┘                                        │
└──────────────────────────────────────────────────────────┘

모바일 (@media max-width:820px):
  body → flex-direction: column
  .panel → 전체 너비, flex-wrap: wrap
```

### 5.2 보드 칸 상태 클래스

| 클래스       | 조건                            | 시각 효과             |
| ------------ | ------------------------------- | --------------------- |
| `.light`     | 밝은 칸 (기본)                  | `--sq-light` 배경     |
| `.dark`      | 어두운 칸 (기본)                | `--sq-dark` 배경      |
| `.selected`  | 선택된 기물 위치                | `--sq-sel` 오버레이   |
| `.hint`      | 이동 가능 빈 칸                 | `::after` 원형 점     |
| `.hint-cap`  | 이동 가능 캡처 칸 (앙파상 포함) | `::after` 링          |
| `.last-from` | 직전 이동 출발                  | `--sq-last` 오버레이  |
| `.last-to`   | 직전 이동 도착                  | `--sq-last` 오버레이  |
| `.in-check`  | 체크 중인 킹 위치               | `--sq-check` 오버레이 |

### 5.3 Canvas 보드 렌더링 (FR-13)

체스판은 `<canvas id="board">` 요소로 렌더링한다. DOM div 방식을 사용하지 않는다.

```
const SQ = 72;          // 칸 크기 (px), 보드 전체 = SQ * 8 = 576px
canvas.width = canvas.height = SQ * 8;

─── 렌더링 레이어 순서 ───────────────────────────────────
1. drawBoard(ctx)       — 64개 칸 배경색 (light/dark)
2. drawOverlays(ctx)    — selected / hint / hint-cap / last-from / last-to / in-check
3. drawPieces(ctx)      — 정지 기물 (애니메이션 중인 기물 제외)
4. drawAnimatingPiece(ctx, piece, x, y)  — 이동 중 기물 (최상위 레이어)
──────────────────────────────────────────────────────────

drawBoard(ctx):
  for i in 0..63:
    r, c = row(i), col(i)
    ctx.fillStyle = (r + c) % 2 === 0 ? SQ_LIGHT : SQ_DARK
    ctx.fillRect(c * SQ, r * SQ, SQ, SQ)

drawPieces(ctx):
  ctx.font = '${SQ * 0.72}px serif'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  for i in 0..63:
    if animating && i === animFrom: continue  // 이동 중 기물 제외
    piece = state.board[i]
    if !piece: continue
    ctx.fillStyle = clr(piece) === 'w' ? '#f0eeff' : '#c8b8ff'
    ctx.shadowColor = '#000'
    ctx.shadowBlur = 3
    ctx.fillText(SYM[piece], c*SQ + SQ/2, r*SQ + SQ/2)

drawOverlays(ctx):
  // selected 칸: rgba(124,106,247,.55)
  // hint 칸 (빈 칸): 원형 점 (반지름 SQ*0.15)
  // hint-cap 칸: 링 (외곽 SQ*0.48, 두께 SQ*0.1)
  // last-from / last-to: rgba(124,106,247,.25)
  // in-check 킹: rgba(220,60,60,.55)

pixelToSq(x, y):  // Canvas 클릭 좌표 → 보드 인덱스
  c = Math.floor(x / SQ)
  r = Math.floor(y / SQ)
  if flipped: r = 7 - r, c = 7 - c
  return r * 8 + c
```

**기물 유니코드 심볼:**

```js
const SYM = {
  wK:'♔', wQ:'♕', wR:'♖', wB:'♗', wN:'♘', wP:'♙',
  bK:'♚', bQ:'♛', bR:'♜', bB:'♝', bN:'♞', bP:'♟'
}
```

### 5.4 모달 설계

**게임 종료 모달 (#end-overlay)**

```
.overlay (position:fixed, full-screen, backdrop)
  └─ .modal
       ├─ #end-icon  (🏆 / 🤖 / 🤝)
       ├─ #end-title ("You Win!" / "AI Wins" / "Draw")
       ├─ #end-sub   (세부 메시지)
       └─ #end-btn   ("New Game" → newGame())
```

**프로모션 선택 모달 (#promo-overlay)**

```
.overlay
  └─ .modal
       ├─ .modal-title ("Promote Pawn")
       ├─ .modal-sub   ("Choose a piece")
       └─ .promo-row#promo-row
            └─ .promo-btn ×4 (Q/R/B/N)
                 클릭 시: 모달 닫기 + doMove(from, to, piece)
```

### 5.5 기물 이동 애니메이션 설계 (FR-14)

모든 기물은 이동 시 출발 칸에서 도착 칸으로 부드럽게 슬라이드한다.

**애니메이션 흐름:**

```
doMove(from, to, promo) 또는 scheduleAI():
  1. animFrom = from  (drawPieces에서 해당 기물 제외)
  2. animatePiece(from, to, duration, finish)
  3. finish():
       applyMove(state, from, to, promo)
       animFrom = null
       renderAll()
       animating = false
```

**`animatePiece(from, to, duration, onComplete)` 구현:**

```js
function animatePiece(from, to, duration, onComplete) {
  animating = true;
  animFrom = from;
  const piece = state.board[from];
  const startX = col(from) * SQ + SQ/2,  startY = row(from) * SQ + SQ/2;
  const endX   = col(to)   * SQ + SQ/2,  endY   = row(to)   * SQ + SQ/2;
  const t0 = performance.now();

  function frame(now) {
    const p = Math.min((now - t0) / duration, 1);
    const ease = 1 - (1 - p) * (1 - p);  // ease-out-quad
    const x = startX + (endX - startX) * ease;
    const y = startY + (endY - startY) * ease;

    renderAll({ skipPiece: from });       // 보드 + 오버레이 + 나머지 기물
    drawAnimatingPiece(ctx, piece, x, y); // 이동 중 기물 최상위 렌더

    if (p < 1) requestAnimationFrame(frame);
    else { animating = false; animFrom = null; onComplete(); }
  }
  requestAnimationFrame(frame);
}
```

**easing**: `ease-out-quad` — `1 - (1-p)²` (감속 곡선, 자연스러운 마무리)

**애니메이션 시간:**

| 이동 주체 | 일반 이동 | 캐슬링 킹 | 캐슬링 룩 |
| --------- | --------- | --------- | --------- |
| 플레이어  | 180ms     | 220ms     | 180ms     |
| AI        | 280ms     | 280ms     | 200ms     |

**캐슬링 순차 애니메이션:**

```
킹 애니메이션 완료 콜백 → 룩 애니메이션 시작 → 룩 완료 콜백 → finish()
```

**입력 차단:**

```js
// onCanvasClick 상단, Undo 버튼 핸들러 상단
if (animating || aiThinking) return;
```

**Undo**: 애니메이션 없이 즉시 이전 스냅샷으로 복원 (`restoreSnap()` → `renderAll()`).

---

## 6. AI 설계

### 6.1 평가 함수 `evaluate(s)`

```
score = 0
for i in 0..63:
  piece = s.board[i]
  if piece === null: continue
  t = typ(piece)
  if clr(piece) === 'w':
    score += VAL[t] + PST[t][i]
  else:
    score -= VAL[t] + PST[t][idx(7-row(i), col(i))]  ← 흑 PST 미러링
return score  // 양수 = 흰 유리
```

### 6.2 미니맥스 + 알파-베타

```
minimax(s, depth, alpha, beta, isMax):
  if depth === 0: return evaluate(s)
  moves = allLegalMoves(s)
  if moves.length === 0:
    return inCheck(s, s.turn)
      ? (isMax ? -900000 : 900000)  // 체크메이트
      : 0                            // 스테일메이트

  if isMax:
    best = -Infinity
    for {from, to} in moves:
      tmp = cloneState(s)
      applyMove(tmp, from, to, 'Q')
      val = minimax(tmp, depth-1, alpha, beta, false)
      best = max(best, val)
      alpha = max(alpha, val)
      if beta <= alpha: break  // 가지치기
    return best
  else:
    // 대칭 (최소화)
```

### 6.3 `bestMove(s, depth)` 난이도

| 난이도 | depth | 딜레이 | 특징      |
| ------ | ----- | ------ | --------- |
| Easy   | 1     | 250ms  | 즉각 반응 |
| Medium | 2     | 400ms  | 전술적 수 |
| Hard   | 3     | 700ms  | 전략적 수 |

이동 목록을 **무작위 셔플** 후 탐색 → 같은 점수의 이동 간 다양성 확보.

---

## 7. 이동 표기법 설계 (toNotation)

```
files = 'abcdefgh'
ranks = '87654321'

캐슬링:
  킹사이드 (킹이 g로): "O-O"
  퀸사이드 (킹이 c로): "O-O-O"

폰:
  일반 전진: "e4"
  캡처: "exd5" (출발 파일 + 'x' + 도착)
  프로모션: "e8=Q"

다른 기물:
  일반: "Nf3" (타입 + 도착)
  캡처: "Nxf3"

체크 / 체크메이트:
  이동 후 inCheck → "+" 접미어
  이동 후 체크메이트 → "#" 접미어
```

---

## 8. 오류 처리

### 8.1 엔진 안전장치

| 상황                  | 처리 방법                                       |
| --------------------- | ----------------------------------------------- |
| 킹을 찾지 못함 (버그) | `inCheck` → `true` 반환 (방어적)                |
| AI 이동 없음          | `bestMove` → `null`, `scheduleAI`에서 조기 종료 |
| Undo 스냅샷 없음      | `snapshots.length === 0` 확인 후 무시           |
| AI 연산 중 Undo       | `aiThinking === true` 확인 후 무시              |

### 8.2 UI 안전장치

| 상황                       | 처리 방법                        |
| -------------------------- | -------------------------------- |
| 플레이어 차례 아닐 때 클릭 | `onSqClick` 상단에서 `return`    |
| 불법 이동 클릭             | `hints.includes(i)` 확인 후 무시 |
| 체크메이트 후 클릭         | 게임 종료 모달로 차단            |

---

## 9. CSS 색상 팔레트 설계

```css
:root {
  /* 배경 */
  --bg: #0e0f11;
  --surface: #161719;
  --surface2: #1c1d20;

  /* 경계 */
  --border: #2a2b2f;
  --border2: #35363c;

  /* 텍스트 */
  --text: #e2e2e5;
  --muted: #8b8c96;
  --subtle: #5a5b63;

  /* 강조 (보라 계열) */
  --accent: #7c6af7;
  --accent-dim: #4e40c8;
  --accent-glow: rgba(124, 106, 247, 0.25);

  /* 보드 칸 */
  --sq-light: #3d3452;
  --sq-dark: #1e1a2e;
  --sq-sel: rgba(124, 106, 247, 0.55);
  --sq-last: rgba(124, 106, 247, 0.25);
  --sq-check: rgba(220, 60, 60, 0.55);
}
```

---

## 10. 코딩 컨벤션 (체스 엔진 특화)

### 10.1 함수 네이밍

| 패턴        | 예시              | 의미             |
| ----------- | ----------------- | ---------------- |
| `make*`     | `makeState()`     | 새 객체 생성     |
| `clone*`    | `cloneState()`    | 깊은 복사        |
| `setup*`    | `setupBoard()`    | 초기 설정        |
| `apply*`    | `applyMove()`     | 상태 직접 변이   |
| `is*`       | `isCheckmate()`   | boolean 반환     |
| `in*`       | `inCheck()`       | boolean 반환     |
| `*Moves`    | `legalMoves()`    | 이동 배열 반환   |
| `*Square*`  | `attacksSquare()` | 단일 칸 관련     |
| `render*`   | `renderAll()`     | DOM 업데이트     |
| `show*`     | `showEndModal()`  | 모달 표시        |
| `schedule*` | `scheduleAI()`    | 비동기 작업 등록 |
| `build*`    | `buildBoard()`    | DOM 구조 생성    |

### 10.2 상태 변이 규칙

```
✅ 허용: applyMove(cloneState(s), from, to)  → 임시 상태에 적용
✅ 허용: applyMove(state, from, to)           → 실제 게임 진행
❌ 금지: 엔진 함수에서 직접 state 수정
```

---

## 11. 구현 순서

### 11.1 파일 구조

```
chess.html   (단일 파일, 전체 구현 포함)
```

### 11.2 구현 순서 (의존성 기반)

1. [ ] HTML 뼈대 + CSS 스타일 (`:root`, 레이아웃, `<canvas id="board">`, 모달)
2. [ ] `SYM`, `VAL`, `PST`, `SQ` 상수 정의
3. [ ] `makeState`, `setupBoard`, `cloneState`, 헬퍼 함수
4. [ ] `attackSquares` (일반 이동 생성)
5. [ ] `attacksSquare` (단일 공격 감지)
6. [ ] `inCheck` (체크 감지)
7. [ ] `applyMove` (이동 적용)
8. [ ] `pseudoMoves` (캐슬링 포함)
9. [ ] `legalMoves`, `allLegalMoves`
10. [ ] `isCheckmate`, `isStalemate`
11. [ ] `evaluate`, `minimax`, `bestMove` (AI)
12. [ ] `toNotation` (체크/체크메이트 접미어 포함)
13. [ ] `initCanvas`, `drawBoard`, `drawOverlays`, `drawPieces` (Canvas 렌더링)
14. [ ] `drawAnimatingPiece`, `pixelToSq` (Canvas 애니메이션/입력)
15. [ ] `renderAll`, `renderStatus`, `renderHistory`, `renderCaptured`, `setThinking`
16. [ ] `animatePiece`, `isCastling`, `getCastlingRook` (애니메이션 로직)
17. [ ] `newGame`, `saveSnap`, `restoreSnap` (게임 흐름)
18. [ ] `onCanvasClick`, `doMove`, `scheduleAI` (이벤트)
19. [ ] `showEndModal`, `showPromo` (모달)
20. [ ] Canvas `mousedown` 이벤트 연결 + `initCanvas()` + `newGame()` 초기화

---

## Version History

| Version | Date       | Changes       | Author |
| ------- | ---------- | ------------- | ------ |
| 0.1     | 2026-03-13 | Initial draft | User   |
