# chess-game Planning Document

> **Summary**: 브라우저에서 바로 실행 가능한 싱글 파일 체스 게임 웹사이트 — AI 상대와 혼자 즐기는 완전한 체스 경험
>
> **Project**: test-claude-chess
> **Version**: 1.0.0
> **Author**: User
> **Date**: 2026-03-13
> **Status**: Draft

---

## Executive Summary

| Perspective            | Content                                                                                                         |
| ---------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Problem**            | 설치나 회원가입 없이 바로 체스를 즐길 수 있는 가볍고 완성도 높은 솔로 체스 게임이 필요하다.                     |
| **Solution**           | 단일 HTML 파일로 완성되는 체스 게임 — 외부 의존성 없이 브라우저에서 즉시 실행 가능하며 AI 상대와 대전한다.      |
| **Function/UX Effect** | 완전한 체스 규칙(캐슬링·앙파상·프로모션), 3단계 AI 난이도, 이동 기록·실행 취소·캡처 표시로 몰입감 있는 UX 제공. |
| **Core Value**         | 파일 하나로 언제 어디서든 체스를 즐길 수 있는 Zero-setup 솔로 체스 게임.                                        |

---

## 1. Overview

### 1.1 Purpose

설치·빌드·로그인 없이 HTML 파일 하나를 열면 즉시 AI 상대와 체스 게임을 즐길 수 있는 웹페이지를 제공한다.

### 1.2 Background

- 기존 온라인 체스 서비스는 회원가입, 광고, 외부 스크립트 로딩 등으로 즉각적인 플레이가 어렵다.
- 단일 파일 체스 게임은 오프라인에서도 동작하며, 공유·배포가 매우 단순하다.
- 이미 `chess.html` 파일이 프로젝트에 존재하며, 이를 PDCA 방식으로 체계적으로 개선·완성한다.

### 1.3 Related Documents

- 기존 구현: `chess.html` (현재 프로젝트 루트)
- 참고 구현: `index.html` (초기 레퍼런스)
- 아키텍처 가이드: `CLAUDE.md`

---

## 2. Scope

### 2.1 In Scope

- [x] 완전한 체스 규칙 엔진 (모든 기물 이동·캡처)
- [x] 특수 이동: 캐슬링 (킹사이드/퀸사이드), 앙파상, 폰 프로모션 (Q/R/B/N 선택)
- [x] 체크 / 체크메이트 / 스테일메이트 감지 및 게임 종료 모달
- [x] AI 상대 — 미니맥스 + 알파-베타 가지치기 + PST 평가
- [x] 3단계 난이도 선택 (Easy depth-1 / Medium depth-2 / Hard depth-3)
- [x] 대수 표기법 이동 기록 패널 (캐슬링 O-O/O-O-O 포함)
- [x] 실행 취소 (Undo) — 플레이어 수 + AI 응수 한 쌍
- [x] 캡처된 기물 표시 (흰/검 분리)
- [x] 보드 플립 (흑 시점 전환)
- [x] AI Thinking bar 애니메이션
- [x] Linear 다크 테마 UI
- [x] 반응형 레이아웃 (모바일 세로 스택)

### 2.2 Out of Scope

- 온라인 멀티플레이어 (WebSocket/WebRTC)
- 게임 저장/불러오기 (localStorage 영속성)
- PGN 내보내기/가져오기
- 50수 규칙·3회 반복 무승부 자동 감지
- AI vs AI 모드
- 타이머/시계 기능
- 사운드 효과

---

## 3. Requirements

### 3.1 Functional Requirements

| ID    | Requirement                                                     | Priority | Status |
| ----- | --------------------------------------------------------------- | -------- | ------ |
| FR-01 | 폰·나이트·비숍·룩·퀸·킹 모든 기물 이동 구현                     | High     | Done   |
| FR-02 | 캐슬링: 킹/룩 미이동, 경유 칸 공격 없음, 체크 중 불가 조건 검증 | High     | Done   |
| FR-03 | 앙파상: 직전 수에만 유효, 다음 수 소멸                          | High     | Done   |
| FR-04 | 폰 프로모션: 플레이어는 Q/R/B/N 선택 모달, AI는 자동 퀸         | High     | Done   |
| FR-05 | 체크/체크메이트/스테일메이트 감지 및 게임 종료 모달 표시        | High     | Done   |
| FR-06 | AI: 미니맥스 + 알파-베타 + PST, 난이도 3단계 (depth 1/2/3)      | High     | Done   |
| FR-07 | 이동 기록: 대수 표기법, 스크롤 가능 패널                        | Medium   | Done   |
| FR-08 | Undo: 플레이어+AI 응수 한 쌍 되돌리기, aiThinking 중 비활성화   | Medium   | Done   |
| FR-09 | 캡처 기물 표시: 흰/검 각각 별도 카드                            | Medium   | Done   |
| FR-10 | 보드 플립: 흑 시점 전환, 레이블 재생성                          | Low      | Done   |
| FR-11 | AI Thinking 진행 바 애니메이션                                  | Low      | Done   |
| FR-12 | New Game 버튼으로 언제든 재시작 가능                            | High     | Done   |

### 3.2 Non-Functional Requirements

| Category        | Criteria                                        | Measurement Method           |
| --------------- | ----------------------------------------------- | ---------------------------- |
| Performance     | AI Hard(depth-3) 응답 < 3초                     | 브라우저 콘솔 타이밍         |
| Compatibility   | Chrome/Firefox/Safari 최신 버전 동작            | 수동 브라우저 테스트         |
| Portability     | 단일 HTML 파일, 외부 의존성 없음                | 파일 크기·네트워크 요청 확인 |
| Usability       | 클릭 2회 이내로 이동 가능 (선택→이동)           | UX 수동 검증                 |
| Maintainability | CSS 커스텀 프로퍼티 기반 테마, 순수 함수형 엔진 | 코드 리뷰                    |

---

## 4. Success Criteria

### 4.1 Definition of Done

- [x] 모든 FR-01~FR-12 기능 구현 완료
- [ ] 브라우저 3종 (Chrome/Firefox/Safari) 동작 확인
- [ ] 체크메이트·스테일메이트·캐슬링·앙파상·프로모션 시나리오 수동 검증
- [ ] `chess.html` 파일 단독으로 오프라인 실행 가능 확인

### 4.2 Quality Criteria

- [ ] `chess.html` 파일 하나로 모든 기능 동작
- [ ] 콘솔 에러 없음
- [ ] 불법 이동(체크 노출) 허용 안 됨
- [ ] AI가 항상 유효한 수를 두거나 게임 종료 처리

---

## 5. Risks and Mitigation

| Risk                           | Impact | Likelihood | Mitigation                                                                      |
| ------------------------------ | ------ | ---------- | ------------------------------------------------------------------------------- |
| `inCheck` ↔ 캐슬링 무한재귀    | High   | Medium     | `attacksSquare` / `attackSquares` 함수 분리, `inCheck`는 `attacksSquare`만 호출 |
| 앙파상 소멸 타이밍 오류        | Medium | Medium     | `applyMove`에서 매 수 시작 시 `s.ep = null` 먼저 초기화                         |
| PST 미러링 오류로 AI 평가 왜곡 | Medium | Low        | 검은 말 PST 인덱스 `idx(7-row(i), col(i))`로 계산 확인                          |
| Hard AI depth-3 UI 블로킹      | Medium | Low        | `setTimeout` 내부에서 AI 실행, Thinking bar로 사용자 피드백                     |
| Undo 중 AI 타이밍 경합         | Low    | Low        | `aiThinking === true`면 Undo 무시                                               |

---

## 6. Architecture Considerations

### 6.1 Project Level Selection

| Level          | Characteristics                               | Recommended For          | Selected |
| -------------- | --------------------------------------------- | ------------------------ | :------: |
| **Starter**    | Simple structure, 단일 파일, 외부 의존성 없음 | Static sites, 게임, 도구 |    ☑     |
| **Dynamic**    | Feature-based modules, BaaS integration       | Web apps with backend    |    ☐     |
| **Enterprise** | Strict layer separation, microservices        | High-traffic systems     |    ☐     |

> **선택: Starter** — 단일 `chess.html` 파일로 완성되는 정적 게임. 빌드 도구·패키지 매니저 불필요.

### 6.2 Key Architectural Decisions

| Decision    | Options                 | Selected             | Rationale                              |
| ----------- | ----------------------- | -------------------- | -------------------------------------- |
| 파일 구조   | 단일 파일 / 멀티 파일   | 단일 HTML            | Zero-setup 배포, 오프라인 동작         |
| 엔진 패턴   | 클래스 기반 / 순수 함수 | 순수 함수            | 테스트·디버그 용이, 상태 클로닝 단순화 |
| 보드 표현   | 2D Array / flat Array   | flat Array(64)       | 인덱스 연산 단순, 캐싱 친화적          |
| AI 알고리즘 | Random / Minimax / MCTS | Minimax + α-β + PST  | 합리적 강도, 구현 복잡도 균형          |
| 스타일링    | 인라인 / `<style>`      | `<style>` 태그       | 단일 파일 내 CSS 커스텀 프로퍼티 활용  |
| 외부 폰트   | 없음 / Google Fonts     | Google Fonts (Inter) | 가독성, CDN 단일 요청                  |

### 6.3 Clean Architecture Approach

```
Selected Level: Starter (Single File)

chess.html 내부 구조:
┌─────────────────────────────────────────────────────┐
│ <style>                                             │
│   CSS Custom Properties (테마 변수)                  │
│   Layout, Board, Piece, Modal, Responsive           │
├─────────────────────────────────────────────────────┤
│ <body>                                              │
│   .panel (좌측 컨트롤 패널)                           │
│   .board-wrap (보드 + thinking bar)                 │
│   Modals (게임 종료, 프로모션)                        │
├─────────────────────────────────────────────────────┤
│ <script>                                            │
│   Constants  : SYM, VAL, PST                        │
│   State      : makeState, setupBoard, cloneState    │
│   Helpers    : row, col, idx, clr, typ, opp         │
│   Engine     : attackSquares, attacksSquare,        │
│                inCheck, applyMove, pseudoMoves,     │
│                legalMoves, allLegalMoves,           │
│                isCheckmate, isStalemate             │
│   AI         : evaluate, minimax, bestMove          │
│   Notation   : toNotation                           │
│   UI         : buildBoard, renderAll, renderSquares │
│                renderStatus, renderHistory,         │
│                renderCaptured, setThinking          │
│   Game Flow  : newGame, saveSnap, restoreSnap,      │
│                onSqClick, doMove, scheduleAI,       │
│                showEndModal, showPromo              │
│   Init       : buildBoard() + newGame()             │
└─────────────────────────────────────────────────────┘
```

---

## 7. Convention Prerequisites

### 7.1 Existing Project Conventions

- [x] `CLAUDE.md` 아키텍처 및 코딩 규칙 문서 존재
- [ ] `docs/01-plan/conventions.md` — 해당 없음 (단일 파일 프로젝트)
- [ ] ESLint / Prettier — 해당 없음 (빌드 도구 없음)
- [ ] TypeScript — 해당 없음 (순수 JS)

### 7.2 Conventions to Define/Verify

| Category           | Current State | Rule                                               | Priority |
| ------------------ | ------------- | -------------------------------------------------- | :------: |
| **보드 인덱스**    | 정의됨        | index 0 = a8, index 63 = h1 (flat Array)           |   High   |
| **말 인코딩**      | 정의됨        | 2글자 문자열: `'wK'`, `'bP'`                       |   High   |
| **함수 호출 계층** | 정의됨        | `inCheck` → `attacksSquare` only (무한재귀 방지)   |   High   |
| **상태 변이**      | 정의됨        | `applyMove`만 state 직접 변이, 나머지는 clone 사용 |   High   |
| **ep 초기화**      | 정의됨        | `applyMove` 시작 시 `s.ep = null` 먼저 실행        |  Medium  |

### 7.3 Environment Variables Needed

해당 없음 — 단일 정적 HTML 파일, 서버·API·환경변수 없음.

### 7.4 Pipeline Integration

단일 파일 Starter 프로젝트로 파이프라인 단계는 적용하지 않음.

---

## 8. Next Steps

1. [ ] `/pdca design chess-game` — 상세 설계 문서 작성
2. [ ] `/pdca analyze chess-game` — 현재 `chess.html` 구현 갭 분석
3. [ ] 브라우저 3종 수동 QA 테스트
4. [ ] `/pdca report chess-game` — 완성 보고서 작성

---

## Version History

| Version | Date       | Changes       | Author |
| ------- | ---------- | ------------- | ------ |
| 0.1     | 2026-03-13 | Initial draft | User   |
