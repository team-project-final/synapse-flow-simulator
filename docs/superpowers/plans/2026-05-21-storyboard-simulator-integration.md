# Storyboard #8 → Interactive Simulator Integration Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add 7 interactive scenarios (4 persona journeys + 3 supplement flows) covering all 12 Epics from storyboard v2.0, with a new `ui_action` step type that visualizes user-facing product flows alongside backend service interactions.

**Architecture:** Data-centric extension of the existing single-file app. `data/scenarios.json` gets new categories, screen actors, and scenario data. `index.html` gets ~35 lines of code for `ui_action` rendering support. No new files, no framework changes, no build process.

**Tech Stack:** Vanilla HTML/CSS/JS, JSON data, Tabler Icons, SVG animation

**Design Spec:** `docs/superpowers/specs/2026-05-21-storyboard-simulator-integration-design.md`

**Source Documents:**
- Storyboard: `/tmp/wiki-docs/08_스토리_보드.md` (cloned wiki)
- Sequence Diagrams: `/tmp/wiki-docs/05_화면_흐름_시퀀스_다이어그램.md` (cloned wiki)

---

## File Map

| File | Action | Responsibility |
|---|---|---|
| `index.html` | Modify (lines 17, 137-148, 254-255, 256-269, 344, 393-438, 492-502, 557-591, 714-736) | CSS variable, step type class, color maps, kind order, arch arrow dash, seq arrow dash, detail panel ui_action fields, log panel ui_action dir label |
| `data/scenarios.json` | Modify (add to categories, actors, scenarios arrays) | New categories, screen actors, 7 scenarios with steps and branches |

---

### Task 1: Add CSS and JS support for `ui_action` step type and `screen` kind

**Files:**
- Modify: `index.html:17` (CSS custom properties in `:root`)
- Modify: `index.html:137-148` (step type CSS classes)
- Modify: `index.html:254` (COLOR_MAP in `initColors()`)
- Modify: `index.html:255` (KIND_COLORS in `initColors()`)
- Modify: `index.html:256-269` (TYPE_ARROW in `initColors()`)
- Modify: `index.html:344` (kindOrder array in `renderArchView()`)

- [ ] **Step 1: Add CSS custom properties for ui_action**

In `index.html`, inside the `:root` block (line 17), after `--cyan:#06b6d4;`, add:

```css
--ui-action:#4ade80;
```

The full line 17 becomes:
```css
  --blue:#3b82f6;--teal:#14b8a6;--coral:#f97316;--amber:#f59e0b;--pink:#ec4899;
  --purple:#9333ea;--red:#ef4444;--green:#22c55e;--gray:#6b7280;--cyan:#06b6d4;--ui-action:#4ade80;
```

- [ ] **Step 2: Add step type CSS class for ui_action**

After line 148 (`.type-sync_op{...}`), add:

```css
.type-ui_action{background:rgba(74,222,128,.15);color:var(--ui-action);border:1px solid rgba(74,222,128,.3)}
```

- [ ] **Step 3: Add ui_action to COLOR_MAP**

In `initColors()` (line 254), extend the COLOR_MAP object. The current line ends with `cyan:v('--cyan')`. Change to:

```js
window.COLOR_MAP = {blue:v('--blue'),teal:v('--teal'),coral:v('--coral'),amber:v('--amber'),pink:v('--pink'),purple:v('--purple'),red:v('--red'),green:v('--green'),gray:v('--gray'),cyan:v('--cyan'),'ui-action':v('--ui-action')};
```

- [ ] **Step 4: Add screen kind to KIND_COLORS**

In `initColors()` (line 255), add `screen` to KIND_COLORS. The current line ends with `k8s:v('--blue')`. Change to:

```js
window.KIND_COLORS = {client:'#4a9eff',screen:v('--ui-action'),edge:v('--purple'),datastore:v('--blue'),broker:v('--amber'),external:v('--gray'),ops:v('--amber'),k8s:v('--blue')};
```

- [ ] **Step 5: Add ui_action to TYPE_ARROW**

In `initColors()`, after the `sync_op` entry (line 268), add before the closing `};`:

```js
    ui_action:{color:v('--ui-action'),dash:'6,4',width:2}
```

The last two lines of TYPE_ARROW become:
```js
    sync_op:{color:v('--teal'),dash:'',width:2},
    ui_action:{color:v('--ui-action'),dash:'6,4',width:2}
  };
```

- [ ] **Step 6: Add screen to kindOrder**

In `renderArchView()` (line 344), change the kindOrder array from:

```js
const kindOrder = ['client','edge','service','ops','k8s','broker','datastore','external'];
```

to:

```js
const kindOrder = ['client','screen','edge','service','ops','k8s','broker','datastore','external'];
```

- [ ] **Step 7: Add ui_action handling to logStep()**

In `logStep()` (lines 720-726), after the `kafka` check (line 725), add a new `else if` for `ui_action`:

```js
  else if (type === 'ui_action') { dir = 'evt'; dirLabel = 'UI'; }
```

The full block becomes:
```js
  let dir = 'evt';
  let dirLabel = 'EVT';
  if (type === 'http_request') { dir = 'req'; dirLabel = 'REQ'; }
  else if (type === 'http_response') { dir = 'res'; dirLabel = 'RES'; }
  else if (type === 'error') { dir = 'err'; dirLabel = 'ERR'; }
  else if (type.startsWith('kafka')) { dir = 'evt'; dirLabel = type === 'kafka_publish' ? 'PUB' : 'SUB'; }
  else if (type === 'ui_action') { dir = 'evt'; dirLabel = 'UI'; }
```

- [ ] **Step 8: Verify the changes render correctly**

Open `index.html` in a browser. No scenarios use `ui_action` yet, but verify:
1. No console errors on page load
2. Existing scenarios still work (click "로그인" scenario, press play)
3. Architecture view and sequence view render correctly

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "feat: add ui_action step type and screen kind support

Add CSS custom property --ui-action, step type class .type-ui_action,
COLOR_MAP/KIND_COLORS/TYPE_ARROW entries, screen kind in kindOrder,
and UI log label. Prepares rendering pipeline for storyboard scenarios."
```

---

### Task 2: Add new categories and screen actors to scenarios.json

**Files:**
- Modify: `data/scenarios.json` (categories array, actors array)

- [ ] **Step 1: Add 3 new categories**

In `data/scenarios.json`, after the last category object (`ops`, ending around line 33), add these 3 categories before the closing `]` of the categories array:

```json
    ,
    {
      "id": "journey-learner",
      "label": "학습자 여정",
      "color": "green",
      "description": "P1·P2·P3 페르소나 사용자 여정",
      "scenarioIds": ["journey-p1-note-graph-share", "journey-p2-paper-ai-search", "journey-p3-onboard-routine-subscribe"]
    },
    {
      "id": "journey-admin",
      "label": "관리자 여정",
      "color": "purple",
      "description": "P4 관리자 일과 여정",
      "scenarioIds": ["journey-p4-admin-daily"]
    },
    {
      "id": "supplement",
      "label": "보충 흐름",
      "color": "cyan",
      "description": "게이미피케이션 · 알림 · 데이터 내보내기",
      "scenarioIds": ["supplement-gamification", "supplement-notification", "supplement-data-export"]
    }
```

- [ ] **Step 2: Add 16 screen actors**

In `data/scenarios.json`, after the last actor object (`Operator`, line 75), add these 16 screen actors before the closing `]` of the actors array:

```json
    ,
    { "id": "SCR-AUTH-001",    "label": "로그인 화면",           "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-AUTH-002",    "label": "회원가입 화면",         "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-AUTH-003",    "label": "MFA 설정",              "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-DASH-001",    "label": "대시보드",              "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-NOTE-001",    "label": "노트 목록",             "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-NOTE-002",    "label": "노트 에디터",           "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-GRAPH-001",   "label": "전체 그래프",           "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-CARD-001",    "label": "덱 목록",               "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-CARD-005",    "label": "복습 세션",             "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-SEARCH-001",  "label": "검색",                  "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-BILLING-001", "label": "플랜/결제",             "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-COMM-001",    "label": "커뮤니티 목록",         "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-COMM-004",    "label": "공유 덱 탐색",          "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-GAME-001",    "label": "게이미피케이션 프로필", "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-NOTI-001",    "label": "알림 센터",             "kind": "screen", "color": "green", "icon": "ti-app-window" },
    { "id": "SCR-ADMIN-001",   "label": "관리자 대시보드",       "kind": "screen", "color": "green", "icon": "ti-app-window" }
```

- [ ] **Step 3: Add Stripe actor**

The storyboard scenarios reference Stripe for billing. Check if a Stripe actor exists. If not (it doesn't currently), add after the screen actors:

```json
    ,
    { "id": "Stripe",          "label": "Stripe",                "kind": "external", "color": "gray",  "icon": "ti-credit-card" },
    { "id": "FCM",             "label": "FCM (Push)",            "kind": "external", "color": "gray",  "icon": "ti-bell" },
    { "id": "Elasticsearch",   "label": "Elasticsearch",         "kind": "datastore", "color": "teal", "icon": "ti-search" },
    { "id": "Cron",            "label": "Cron Scheduler",        "kind": "ops",      "color": "amber", "icon": "ti-clock" }
```

Note: `Elasticsearch` may already exist in the actors — check before adding. If it does, skip that line.

- [ ] **Step 4: Verify JSON validity**

```bash
cd C:/workspace/team-project-manager/team-project-final/synapse-flow-simulator
node -e "const d = require('./data/scenarios.json'); console.log('Categories:', d.categories.length, 'Actors:', d.actors.length)"
```

Expected output: `Categories: 7 Actors: XX` (where XX is previous count + 20)

- [ ] **Step 5: Verify in browser**

Open `index.html` in browser. The sidebar should now show 7 category groups. The 3 new categories ("학습자 여정", "관리자 여정", "보충 흐름") should appear at the bottom, empty (no scenarios yet).

- [ ] **Step 6: Commit**

```bash
git add data/scenarios.json
git commit -m "feat: add storyboard categories, screen actors, and external service actors

Add 3 new categories (journey-learner, journey-admin, supplement),
16 screen actors (SCR-*), and 4 external/ops actors (Stripe, FCM,
Elasticsearch, Cron) to prepare for storyboard scenario data."
```

---

### Task 3: Add P1 journey scenario — 김시냅스: 노트→그래프→그룹 공유

**Files:**
- Modify: `data/scenarios.json` (scenarios array)

**Source:** Storyboard §3 P1 Journey Map + §2 Epic 1/2/4/5/6/8 User Stories + Sequence Diagrams §5.2/5.3/5.4/5.7/5.9/5.10

- [ ] **Step 1: Add P1 journey scenario**

In `data/scenarios.json`, at the end of the `scenarios` array (before its closing `]`), add:

```json
    ,
    {
      "id": "journey-p1-note-graph-share",
      "title": "P1 김시냅스: 노트 → 그래프 → 그룹 공유",
      "category": "journey-learner", "priority": "P0", "status": "implemented",
      "description": "개발자 김시냅스의 여정: GitHub OAuth 로그인 → 마크다운 노트 작성 + 위키링크 → 그래프 탐색 → AI 카드 생성 → 스터디 그룹 공유 → Pro 구독. Epic 1, 2, 4, 5, 6, 8을 관통하며 Epic 9(XP)가 자연스럽게 삽입된다.",
      "actors": ["Browser", "SCR-AUTH-001", "SCR-DASH-001", "SCR-NOTE-002", "SCR-GRAPH-001", "SCR-CARD-001", "SCR-COMM-001", "SCR-COMM-004", "SCR-GAME-001", "SCR-BILLING-001", "Gateway", "Platform", "Knowledge", "LearningAI", "Engagement", "LearningCard", "PostgreSQL", "Redis", "Kafka", "ClaudeAPI", "GitHubOAuth", "Stripe", "pgvector"],
      "ui": { "layout": "sequence", "tags": ["OAuth", "마크다운", "위키링크", "AI카드", "그래프", "커뮤니티", "결제"], "estimatedDuration": 5000 },
      "rules": ["§6.1", "§6.2", "§8.1", "§8.6"],
      "files": [],
      "steps": [
        { "n": 1, "from": "Browser", "to": "SCR-AUTH-001", "type": "ui_action", "action": "로그인 화면 진입 → GitHub OAuth 버튼 클릭",
          "payload": { "screen": "SCR-W-AUTH-001", "element": "GitHub OAuth 버튼" }, "duration": 10,
          "annotations": ["P1 김시냅스", "Epic 1: US-1.1"], "tags": ["OAuth"] },
        { "n": 2, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/auth/oauth/github", "duration": 20 },
        { "n": 3, "from": "Platform", "to": "GitHubOAuth", "type": "external_api", "action": "Authorization code → access_token 교환", "duration": 500,
          "annotations": ["AES-256-GCM 토큰 암호화"] },
        { "n": 4, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "UPSERT oauth_identities + users", "duration": 15 },
        { "n": 5, "from": "Platform", "to": "Redis", "type": "cache_op", "action": "SET refresh:{userId} TTL 7day", "duration": 3 },
        { "n": 6, "from": "Gateway", "to": "Browser", "type": "http_response", "action": "200 OK {accessToken, user}",
          "payload": { "accessToken": "eyJ...", "expiresIn": 900 }, "duration": 5 },

        { "n": 7, "from": "SCR-AUTH-001", "to": "SCR-DASH-001", "type": "ui_action", "action": "대시보드로 리다이렉트 → '새 노트' 버튼 클릭",
          "payload": { "screen": "SCR-W-DASH-001", "element": "새 노트 버튼" }, "duration": 10,
          "annotations": ["P1 김시냅스", "Epic 2: US-2.1"] },
        { "n": 8, "from": "SCR-DASH-001", "to": "SCR-NOTE-002", "type": "ui_action", "action": "노트 에디터 진입 → 마크다운 작성 + [[Spring Boot]] 위키링크 입력",
          "payload": { "screen": "SCR-W-NOTE-002", "element": "에디터 + 위키링크 자동완성" }, "duration": 20,
          "annotations": ["P1 김시냅스", "Epic 2: US-2.1, US-2.2"], "tags": ["마크다운", "위키링크"] },
        { "n": 9, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "PUT /api/v1/notes/{id} (3초 디바운스 자동저장)",
          "payload": { "title": "Spring Boot 4 정리", "content": "## 핵심 개념\n[[DI]]와 [[AOP]]...", "format": "markdown" }, "duration": 30 },
        { "n": 10, "from": "Knowledge", "to": "PostgreSQL", "type": "db_query", "action": "UPDATE notes SET content=? + INSERT backlinks", "duration": 10,
          "annotations": ["위키링크 파싱 → backlinks 테이블 갱신"] },
        { "n": 11, "from": "Knowledge", "to": "Kafka", "type": "kafka_publish", "action": "knowledge.note.note-updated-v1",
          "topic": "knowledge.note.note-updated-v1", "duration": 5, "annotations": ["백링크 갱신 + 임베딩 트리거"] },

        { "n": 12, "from": "SCR-NOTE-002", "to": "SCR-GRAPH-001", "type": "ui_action", "action": "그래프 뷰로 전환 → 노트 연결 구조 탐색",
          "payload": { "screen": "SCR-W-GRAPH-001", "element": "그래프 캔버스" }, "duration": 10,
          "annotations": ["P1 김시냅스", "Epic 5: US-5.1"], "tags": ["그래프"] },
        { "n": 13, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "GET /api/v1/graph?depth=2", "duration": 30 },
        { "n": 14, "from": "Knowledge", "to": "PostgreSQL", "type": "db_query", "action": "WITH RECURSIVE graph_cte SELECT notes + backlinks (2-hop)", "duration": 20,
          "annotations": ["Force-directed 레이아웃용 노드/엣지 반환"] },

        { "n": 15, "from": "SCR-GRAPH-001", "to": "SCR-NOTE-002", "type": "ui_action", "action": "그래프에서 고립 노드 발견 → 노트로 돌아가 'AI 카드 생성' 클릭 (10장)",
          "payload": { "screen": "SCR-W-NOTE-002", "element": "AI 카드 생성 버튼" }, "duration": 10,
          "annotations": ["P1 김시냅스", "Epic 4: US-4.1"], "tags": ["AI카드"] },
        { "n": 16, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/ai/cards/generate {noteId, count: 10}", "duration": 20 },
        { "n": 17, "from": "LearningAI", "to": "ClaudeAPI", "type": "external_api", "action": "노트 청크 + 프롬프트 → 카드 JSON 스트리밍",
          "duration": 3000, "annotations": ["Claude API, 토큰 예산 4096"], "tags": ["Claude"] },
        { "n": 18, "from": "LearningAI", "to": "Kafka", "type": "kafka_publish", "action": "learning.ai.cards-generated-v1",
          "topic": "learning.ai.cards-generated-v1", "duration": 5, "annotations": ["Engagement 서비스 구독 → XP 적립"] },

        { "n": 19, "from": "SCR-NOTE-002", "to": "SCR-COMM-001", "type": "ui_action", "action": "커뮤니티 탭 → 'AWS 스터디' 그룹 가입",
          "payload": { "screen": "SCR-W-COMM-001", "element": "그룹 가입 버튼" }, "duration": 10,
          "annotations": ["P1 김시냅스", "Epic 8: US-8.1"], "tags": ["커뮤니티"] },
        { "n": 20, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/groups/{groupId}/join", "duration": 15 },
        { "n": 21, "from": "Engagement", "to": "PostgreSQL", "type": "db_query", "action": "INSERT group_members (role='member')", "duration": 8 },

        { "n": 22, "from": "SCR-COMM-001", "to": "SCR-COMM-004", "type": "ui_action", "action": "내 덱에서 '공유' → 그룹 선택 → 공유 완료",
          "payload": { "screen": "SCR-W-COMM-004", "element": "덱 공유 버튼" }, "duration": 10,
          "annotations": ["P1 김시냅스", "Epic 8: US-8.2"] },
        { "n": 23, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/decks/{deckId}/share {targetGroupId}", "duration": 15 },
        { "n": 24, "from": "Engagement", "to": "Kafka", "type": "kafka_publish", "action": "engagement.community.deck-shared-v1 + XP 적립",
          "topic": "engagement.community.deck-shared-v1", "duration": 5,
          "annotations": ["덱 공유 XP +50", "Epic 9: US-9.1"], "tags": ["XP"] },

        { "n": 25, "from": "SCR-COMM-004", "to": "SCR-BILLING-001", "type": "ui_action", "action": "할당량 소진 알림 → Pro 플랜 페이지 이동",
          "payload": { "screen": "SCR-W-BILLING-001", "element": "Pro 구독 버튼" }, "duration": 10,
          "annotations": ["P1 김시냅스", "Epic 6: US-6.1"], "tags": ["결제"] },
        { "n": 26, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/billing/checkout {plan: 'pro'}", "duration": 20 },
        { "n": 27, "from": "Platform", "to": "Stripe", "type": "external_api", "action": "Stripe Checkout Session 생성 → 결제 완료 webhook",
          "duration": 2000, "annotations": ["Stripe webhook → payment-completed 이벤트"] },
        { "n": 28, "from": "Gateway", "to": "Browser", "type": "http_response", "action": "200 OK — Pro 플랜 활성화 완료",
          "payload": { "plan": "pro", "status": "active", "nextBilling": "2026-06-21" }, "duration": 5 }
      ],
      "branches": {
        "failures": [
          { "id": "p1-oauth-fail", "trigger": "GitHub OAuth 토큰 교환 실패", "afterStep": 3,
            "result": { "status": 502, "code": "AUTH_OAUTH_FAIL", "message": "GitHub 인증 서버 응답 없음" },
            "recovery": "재시도 버튼 → OAuth 플로우 재시작" },
          { "id": "p1-note-conflict", "trigger": "노트 자동저장 충돌 (동시 편집)", "afterStep": 9,
            "result": { "status": 409, "code": "NOTE_CONFLICT", "message": "서버 버전과 충돌 — diff 표시" },
            "recovery": "버전 diff UI에서 사용자가 병합 선택" },
          { "id": "p1-ai-fail", "trigger": "Claude API 타임아웃", "afterStep": 17,
            "result": { "status": 504, "code": "LLM_TIMEOUT", "message": "Claude API 30초 타임아웃" },
            "recovery": "OpenAI 폴백 → 카드 생성 재시도 (flow-llm-fallback 시나리오 참조)" },
          { "id": "p1-payment-fail", "trigger": "Stripe 카드 거절", "afterStep": 27,
            "result": { "status": 402, "code": "PAYMENT_DECLINED", "message": "카드 결제 거절" },
            "recovery": "다른 결제 수단 입력 → 재시도" }
        ],
        "alternates": [
          { "id": "p1-mobile-login", "trigger": "모바일 앱에서 로그인", "afterStep": 1,
            "result": { "message": "MobileApp → Gateway 경로 + FCM 토큰 등록 추가" } },
          { "id": "p1-stay-free", "trigger": "무료 플랜 유지 선택", "afterStep": 25,
            "result": { "message": "AI 기능 제한 범위 내에서 계속 사용 — 할당량 리셋 대기" } }
        ]
      }
    }
```

- [ ] **Step 2: Verify JSON validity**

```bash
node -e "const d = require('./data/scenarios.json'); const s = d.scenarios.find(x=>x.id==='journey-p1-note-graph-share'); console.log('Steps:', s.steps.length, 'Branches:', (s.branches.failures||[]).length + (s.branches.alternates||[]).length)"
```

Expected: `Steps: 28 Branches: 6`

- [ ] **Step 3: Test in browser**

Open `index.html`, click "P1 김시냅스: 노트 → 그래프 → 그룹 공유" in the sidebar. Verify:
1. Architecture view shows screen actors (green, with ti-app-window icon) in a row between client and edge
2. `ui_action` steps render with dashed green arrows
3. Sequence view shows dashed green arrows for `ui_action` steps
4. Detail panel displays `payload.screen` and `payload.element` for `ui_action` steps
5. Log panel shows "UI" label for `ui_action` steps
6. Auto-play works through all 28 steps
7. Branch items appear in detail panel

- [ ] **Step 4: Commit**

```bash
git add data/scenarios.json
git commit -m "feat: add P1 김시냅스 journey scenario (28 steps, 6 branches)

OAuth login → markdown note + wikilink → graph view → AI card generation
→ study group join + deck share (XP) → Pro subscription.
Covers Epic 1, 2, 4, 5, 6, 8 with Epic 9 XP insertion."
```

---

### Task 4: Add P2 journey scenario — 이연구: 논문→AI카드→시맨틱검색

**Files:**
- Modify: `data/scenarios.json` (scenarios array)

**Source:** Storyboard §3 P2 Journey Map + §2 Epic 1/2/3/4/6 + Sequence Diagrams §5.2/5.4/5.5/5.6/5.7

- [ ] **Step 1: Add P2 journey scenario**

In `data/scenarios.json`, at the end of the `scenarios` array, add:

```json
    ,
    {
      "id": "journey-p2-paper-ai-search",
      "title": "P2 이연구: 논문 → AI 카드 → 시맨틱 검색",
      "category": "journey-learner", "priority": "P0", "status": "implemented",
      "description": "대학원생 이연구의 여정: Google 로그인 → MFA 설정 → 논문 노트(LaTeX) → AI 카드 20장 생성 → 카드 검토·편집 → 일일 복습(SM-2) → 시맨틱 검색 → RAG Q&A → Pro 구독. Epic 1, 2, 3, 4, 6을 관통하며 Epic 10(알림)이 삽입된다.",
      "actors": ["Browser", "SCR-AUTH-001", "SCR-AUTH-003", "SCR-DASH-001", "SCR-NOTE-002", "SCR-CARD-001", "SCR-CARD-005", "SCR-SEARCH-001", "SCR-BILLING-001", "SCR-NOTI-001", "Gateway", "Platform", "Knowledge", "LearningAI", "LearningCard", "Engagement", "PostgreSQL", "Redis", "Kafka", "ClaudeAPI", "GoogleOAuth", "Authenticator", "pgvector", "Stripe"],
      "ui": { "layout": "sequence", "tags": ["MFA", "TOTP", "LaTeX", "AI카드", "SM-2", "시맨틱검색", "RAG"], "estimatedDuration": 8000 },
      "rules": ["§6.2", "§6.3", "§8.1"],
      "files": [],
      "steps": [
        { "n": 1, "from": "Browser", "to": "SCR-AUTH-001", "type": "ui_action", "action": "로그인 화면 → Google OAuth 클릭",
          "payload": { "screen": "SCR-W-AUTH-001", "element": "Google OAuth 버튼" }, "duration": 10,
          "annotations": ["P2 이연구", "Epic 1: US-1.1"] },
        { "n": 2, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/auth/oauth/google", "duration": 20 },
        { "n": 3, "from": "Platform", "to": "GoogleOAuth", "type": "external_api", "action": "Authorization code → access_token 교환", "duration": 500 },
        { "n": 4, "from": "Gateway", "to": "Browser", "type": "http_response", "action": "200 OK {accessToken, user}", "duration": 5 },

        { "n": 5, "from": "SCR-AUTH-001", "to": "SCR-AUTH-003", "type": "ui_action", "action": "보안 설정 → MFA 활성화 → QR 코드 스캔",
          "payload": { "screen": "SCR-W-AUTH-003", "element": "MFA 활성화 토글 + QR" }, "duration": 15,
          "annotations": ["P2 이연구", "Epic 1: US-1.2"], "tags": ["MFA"] },
        { "n": 6, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/auth/mfa/setup", "duration": 20 },
        { "n": 7, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "INSERT mfa_credentials (secret, recovery_codes x10)", "duration": 10,
          "annotations": ["TOTP secret 생성 + 복구 코드 10개"] },
        { "n": 8, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/auth/mfa/verify {code: '123456'}",
          "payload": { "code": "123456" }, "duration": 15 },
        { "n": 9, "from": "Platform", "to": "Authenticator", "type": "internal", "action": "TOTP 검증 (30초 윈도우)", "duration": 5,
          "annotations": ["Google Authenticator 호환"] },

        { "n": 10, "from": "SCR-AUTH-003", "to": "SCR-NOTE-002", "type": "ui_action", "action": "대시보드 → 새 노트 → 논문 요약 작성 (LaTeX 포함)",
          "payload": { "screen": "SCR-W-NOTE-002", "element": "에디터 (LaTeX 수식)" }, "duration": 20,
          "annotations": ["P2 이연구", "Epic 2: US-2.1"], "tags": ["LaTeX"] },
        { "n": 11, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "PUT /api/v1/notes/{id}",
          "payload": { "title": "Attention Is All You Need 정리", "content": "## Transformer\\n$Q K^T / \\sqrt{d_k}$..." }, "duration": 30 },
        { "n": 12, "from": "Knowledge", "to": "Kafka", "type": "kafka_publish", "action": "knowledge.note.note-updated-v1",
          "topic": "knowledge.note.note-updated-v1", "duration": 5 },

        { "n": 13, "from": "SCR-NOTE-002", "to": "SCR-NOTE-002", "type": "ui_action", "action": "'AI 카드 생성' 클릭 → 20장 선택",
          "payload": { "screen": "SCR-W-NOTE-002", "element": "AI 카드 생성 버튼 (count: 20)" }, "duration": 10,
          "annotations": ["P2 이연구", "Epic 4: US-4.1"], "tags": ["AI카드"] },
        { "n": 14, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/ai/cards/generate {noteId, count: 20}", "duration": 20 },
        { "n": 15, "from": "LearningAI", "to": "ClaudeAPI", "type": "external_api", "action": "논문 청크 + 프롬프트 → 카드 20장 스트리밍", "duration": 5000,
          "annotations": ["토큰 예산 8192 (20장)"] },
        { "n": 16, "from": "LearningAI", "to": "Kafka", "type": "kafka_publish", "action": "learning.ai.cards-generated-v1",
          "topic": "learning.ai.cards-generated-v1", "duration": 5 },
        { "n": 17, "from": "SCR-NOTE-002", "to": "SCR-CARD-001", "type": "ui_action", "action": "생성된 카드 미리보기 → 3장 편집 → 2장 삭제 → '모두 저장'",
          "payload": { "screen": "SCR-W-CARD-001", "element": "카드 미리보기/편집/삭제" }, "duration": 15,
          "annotations": ["P2 이연구", "Epic 4: US-4.1 — 카드 검토"] },

        { "n": 18, "from": "SCR-CARD-001", "to": "SCR-CARD-005", "type": "ui_action", "action": "'복습 시작' 클릭 → 복습 세션 진입",
          "payload": { "screen": "SCR-W-CARD-005", "element": "복습 시작 버튼" }, "duration": 10,
          "annotations": ["P2 이연구", "Epic 3: US-3.2"], "tags": ["SM-2"] },
        { "n": 19, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "GET /api/v1/cards/review/today", "duration": 15 },
        { "n": 20, "from": "LearningCard", "to": "PostgreSQL", "type": "db_query", "action": "SELECT due cards WHERE next_review <= NOW() ORDER BY ease_factor",
          "duration": 10, "annotations": ["SM-2 알고리즘 — 간격 반복 스케줄링"] },
        { "n": 21, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/cards/{id}/review {rating: 3}",
          "payload": { "rating": 3, "responseTime": 4200 }, "duration": 10 },
        { "n": 22, "from": "LearningCard", "to": "Kafka", "type": "kafka_publish", "action": "learning.card.review-completed-v1",
          "topic": "learning.card.review-completed-v1", "duration": 5,
          "annotations": ["Engagement 구독 → XP 적립", "Epic 10: 복습 완료 알림 트리거"] },

        { "n": 23, "from": "SCR-CARD-005", "to": "SCR-SEARCH-001", "type": "ui_action", "action": "검색 → 시맨틱 모드 토글 → 'transformer attention' 검색",
          "payload": { "screen": "SCR-W-SEARCH-001", "element": "시맨틱 검색 토글" }, "duration": 10,
          "annotations": ["P2 이연구", "Epic 4: US-4.2"], "tags": ["시맨틱검색"] },
        { "n": 24, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/search/semantic {query: 'transformer attention mechanism'}", "duration": 20 },
        { "n": 25, "from": "Knowledge", "to": "pgvector", "type": "db_query", "action": "HNSW cosine similarity → 상위 10개 노트 반환",
          "duration": 50, "annotations": ["OpenAI text-embedding-3-small → pgvector 검색"] },
        { "n": 26, "from": "Gateway", "to": "Browser", "type": "http_response", "action": "200 OK — 유사 노트 10개 + 유사도 점수",
          "payload": { "results": [{"noteId": "...", "similarity": 0.92, "title": "Self-Attention 메커니즘"}] }, "duration": 5 },

        { "n": 27, "from": "SCR-SEARCH-001", "to": "SCR-BILLING-001", "type": "ui_action", "action": "14일 무료 체험 종료 알림 → Pro 구독 클릭",
          "payload": { "screen": "SCR-W-BILLING-001", "element": "Pro 구독 버튼" }, "duration": 10,
          "annotations": ["P2 이연구", "Epic 6: US-6.1"], "tags": ["결제"] },
        { "n": 28, "from": "Platform", "to": "Stripe", "type": "external_api", "action": "Stripe Checkout → 결제 완료", "duration": 2000 },
        { "n": 29, "from": "Gateway", "to": "Browser", "type": "http_response", "action": "200 OK — Pro 활성화",
          "payload": { "plan": "pro", "status": "active" }, "duration": 5 }
      ],
      "branches": {
        "failures": [
          { "id": "p2-mfa-fail", "trigger": "TOTP 코드 불일치", "afterStep": 8,
            "result": { "status": 401, "code": "MFA_INVALID", "message": "인증 코드가 올바르지 않습니다" },
            "recovery": "재입력 또는 복구 코드 사용" },
          { "id": "p2-ai-quality", "trigger": "AI 카드 품질 낮음 (20장 중 5장 부적절)", "afterStep": 15,
            "result": { "status": 200, "message": "생성 카드 중 일부 품질 미달 — 편집/삭제 필요" },
            "recovery": "카드 미리보기에서 개별 편집·삭제 후 재저장" },
          { "id": "p2-search-empty", "trigger": "시맨틱 검색 결과 0건", "afterStep": 25,
            "result": { "status": 200, "message": "유사 노트 없음 — 노트가 충분하지 않거나 임베딩 미완료" },
            "recovery": "RAG Q&A 챗봇 권장 팝업 표시" }
        ],
        "alternates": [
          { "id": "p2-review-reminder", "trigger": "복습 리마인더 알림 수신 (Epic 10)", "afterStep": 22,
            "result": { "message": "FCM 푸시 알림 → 클릭 시 SCR-W-CARD-005 복습 세션으로 바로 이동" } }
        ]
      }
    }
```

- [ ] **Step 2: Verify JSON and test in browser**

```bash
node -e "const d = require('./data/scenarios.json'); const s = d.scenarios.find(x=>x.id==='journey-p2-paper-ai-search'); console.log('Steps:', s.steps.length, 'Branches:', (s.branches.failures||[]).length + (s.branches.alternates||[]).length)"
```

Expected: `Steps: 29 Branches: 4`

Open browser, select the P2 scenario, verify all steps play through correctly.

- [ ] **Step 3: Commit**

```bash
git add data/scenarios.json
git commit -m "feat: add P2 이연구 journey scenario (29 steps, 4 branches)

Google OAuth + MFA → paper note (LaTeX) → AI card generation (20 cards)
→ card review/edit → SRS review session (SM-2) → semantic search
→ Pro subscription. Covers Epic 1, 2, 3, 4, 6 with Epic 10 insertion."
```

---

### Task 5: Add P3 journey scenario — 박합격: 첫사용→복습루틴→구독

**Files:**
- Modify: `data/scenarios.json` (scenarios array)

**Source:** Storyboard §3 P3 Journey Map + §2 Epic 1/2/3/4/6 + Sequence Diagrams §5.1/5.4/5.5/5.7

- [ ] **Step 1: Add P3 journey scenario**

In `data/scenarios.json`, at the end of the `scenarios` array, add:

```json
    ,
    {
      "id": "journey-p3-onboard-routine-subscribe",
      "title": "P3 박합격: 첫 사용 → 복습 루틴 → 구독",
      "category": "journey-learner", "priority": "P0", "status": "implemented",
      "description": "자격증 준비자 박합격의 여정: 이메일 회원가입 → 온보딩 투어 → AWS 학습 노트 → AI 카드 생성 → 첫 복습(SM-2) → 매일 복습 루틴 + 스트릭(Epic 9) → 할당량 소진 → Pro 구독. Epic 1, 2, 3, 4, 6을 관통하며 Epic 9(스트릭)가 삽입된다.",
      "actors": ["MobileApp", "SCR-AUTH-002", "SCR-DASH-001", "SCR-NOTE-002", "SCR-CARD-001", "SCR-CARD-005", "SCR-GAME-001", "SCR-BILLING-001", "Gateway", "Platform", "Knowledge", "LearningAI", "LearningCard", "Engagement", "PostgreSQL", "Redis", "Kafka", "ClaudeAPI", "Stripe"],
      "ui": { "layout": "sequence", "tags": ["이메일가입", "온보딩", "SM-2", "스트릭", "모바일"], "estimatedDuration": 6000 },
      "rules": ["§6.1", "§8.1"],
      "files": [],
      "steps": [
        { "n": 1, "from": "MobileApp", "to": "SCR-AUTH-002", "type": "ui_action", "action": "앱 설치 → 회원가입 화면 → 이메일+비밀번호 입력",
          "payload": { "screen": "SCR-W-AUTH-002", "element": "이메일 가입 폼" }, "duration": 15,
          "annotations": ["P3 박합격", "Epic 1: US-1.3"], "tags": ["이메일가입"] },
        { "n": 2, "from": "MobileApp", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/auth/signup {email, password, displayName}",
          "payload": { "email": "park@example.com", "password": "***", "displayName": "박합격" }, "duration": 20 },
        { "n": 3, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "BEGIN → INSERT users + tenants + tenant_members + usage_counters → COMMIT",
          "duration": 25, "annotations": ["Free 플랜 초기값 설정"] },
        { "n": 4, "from": "Platform", "to": "Kafka", "type": "kafka_publish", "action": "platform.auth.user-registered-v1",
          "topic": "platform.auth.user-registered-v1", "duration": 5, "annotations": ["이메일 인증 메일 발송 트리거"] },
        { "n": 5, "from": "Gateway", "to": "MobileApp", "type": "http_response", "action": "201 Created {userId, accessToken}", "duration": 5 },

        { "n": 6, "from": "SCR-AUTH-002", "to": "SCR-DASH-001", "type": "ui_action", "action": "이메일 인증 완료 → 대시보드 → 온보딩 투어 시작",
          "payload": { "screen": "SCR-W-DASH-001", "element": "온보딩 투어 모달" }, "duration": 15,
          "annotations": ["P3 박합격", "Epic 1 — 첫 사용 경험"] },
        { "n": 7, "from": "SCR-DASH-001", "to": "SCR-NOTE-002", "type": "ui_action", "action": "온보딩 완료 → 샘플 노트 확인 → AWS SAA 학습 노트 작성",
          "payload": { "screen": "SCR-W-NOTE-002", "element": "에디터" }, "duration": 20,
          "annotations": ["P3 박합격", "Epic 2: US-2.1"], "tags": ["노트"] },
        { "n": 8, "from": "MobileApp", "to": "Gateway", "type": "http_request", "action": "PUT /api/v1/notes/{id}",
          "payload": { "title": "AWS EC2 핵심 정리", "content": "## 인스턴스 유형\n- t3.micro: 버스트 가능..." }, "duration": 30 },
        { "n": 9, "from": "Knowledge", "to": "PostgreSQL", "type": "db_query", "action": "UPDATE notes + INSERT search index", "duration": 10 },

        { "n": 10, "from": "SCR-NOTE-002", "to": "SCR-NOTE-002", "type": "ui_action", "action": "'AI 카드 생성' 버튼 클릭 (5장)",
          "payload": { "screen": "SCR-W-NOTE-002", "element": "AI 카드 생성 버튼 (count: 5)" }, "duration": 10,
          "annotations": ["P3 박합격", "Epic 4: US-4.1"], "tags": ["AI카드"] },
        { "n": 11, "from": "MobileApp", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/ai/cards/generate {noteId, count: 5}", "duration": 20 },
        { "n": 12, "from": "LearningAI", "to": "ClaudeAPI", "type": "external_api", "action": "노트 청크 → 카드 5장 생성", "duration": 2000 },
        { "n": 13, "from": "LearningAI", "to": "Kafka", "type": "kafka_publish", "action": "learning.ai.cards-generated-v1",
          "topic": "learning.ai.cards-generated-v1", "duration": 5 },

        { "n": 14, "from": "SCR-NOTE-002", "to": "SCR-CARD-005", "type": "ui_action", "action": "'복습 시작' → 첫 복습 세션 진입",
          "payload": { "screen": "SCR-W-CARD-005", "element": "복습 시작 버튼" }, "duration": 10,
          "annotations": ["P3 박합격", "Epic 3: US-3.2"], "tags": ["SM-2"] },
        { "n": 15, "from": "MobileApp", "to": "Gateway", "type": "http_request", "action": "GET /api/v1/cards/review/today", "duration": 15 },
        { "n": 16, "from": "LearningCard", "to": "PostgreSQL", "type": "db_query", "action": "SELECT due cards WHERE next_review <= NOW()", "duration": 10 },
        { "n": 17, "from": "MobileApp", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/cards/{id}/review {rating: 4}",
          "payload": { "rating": 4, "responseTime": 3500 }, "duration": 10 },
        { "n": 18, "from": "LearningCard", "to": "Kafka", "type": "kafka_publish", "action": "learning.card.review-completed-v1",
          "topic": "learning.card.review-completed-v1", "duration": 5 },
        { "n": 19, "from": "Gateway", "to": "MobileApp", "type": "http_response", "action": "복습 완료 — 정답률 80%, 5장 완료",
          "payload": { "accuracy": 0.8, "cardsReviewed": 5, "timeSpent": "4min 30s" }, "duration": 5 },

        { "n": 20, "from": "Engagement", "to": "Engagement", "type": "internal", "action": "스트릭 체크: 연속 7일 학습 → 보너스 XP +100",
          "duration": 5, "annotations": ["Epic 9: US-9.4 — 스트릭 메카닉"], "tags": ["스트릭"] },
        { "n": 21, "from": "Engagement", "to": "Kafka", "type": "kafka_publish", "action": "engagement.gamification.streak-bonus-v1",
          "topic": "engagement.gamification.badge-earned-v1", "duration": 5,
          "annotations": ["스트릭 7일 보너스 XP", "SCR-W-GAME-001에 🔥 표시"] },

        { "n": 22, "from": "SCR-CARD-005", "to": "SCR-BILLING-001", "type": "ui_action", "action": "할당량 소진 알림 팝업 → Pro 플랜 페이지",
          "payload": { "screen": "SCR-W-BILLING-001", "element": "Pro 구독 버튼" }, "duration": 10,
          "annotations": ["P3 박합격", "Epic 6: US-6.1"], "tags": ["결제"] },
        { "n": 23, "from": "MobileApp", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/billing/checkout {plan: 'pro'}", "duration": 20 },
        { "n": 24, "from": "Platform", "to": "Stripe", "type": "external_api", "action": "Stripe Checkout → 결제 완료", "duration": 2000 },
        { "n": 25, "from": "Gateway", "to": "MobileApp", "type": "http_response", "action": "200 OK — Pro 활성화 완료",
          "payload": { "plan": "pro", "status": "active" }, "duration": 5 }
      ],
      "branches": {
        "failures": [
          { "id": "p3-email-unverified", "trigger": "이메일 인증 미완료", "afterStep": 4,
            "result": { "status": 403, "code": "EMAIL_UNVERIFIED", "message": "이메일 인증이 필요합니다" },
            "recovery": "인증 메일 재발송 → 인증 링크 클릭" },
          { "id": "p3-streak-reset", "trigger": "학습 없는 날 → 스트릭 리셋", "afterStep": 20,
            "result": { "message": "0시 Cron → 스트릭 0일로 리셋 + '다시 시작하세요' 알림" },
            "recovery": "다음 날 학습 시 스트릭 1일부터 재시작" },
          { "id": "p3-payment-fail", "trigger": "결제 실패", "afterStep": 24,
            "result": { "status": 402, "code": "PAYMENT_DECLINED", "message": "결제 수단 거절" },
            "recovery": "다른 카드 입력 → 재시도" }
        ],
        "alternates": [
          { "id": "p3-skip-onboarding", "trigger": "온보딩 투어 건너뛰기", "afterStep": 6,
            "result": { "message": "빈 대시보드에 '첫 노트를 작성해보세요' 가이드 카드 표시" } }
        ]
      }
    }
```

- [ ] **Step 2: Verify and test**

```bash
node -e "const d = require('./data/scenarios.json'); const s = d.scenarios.find(x=>x.id==='journey-p3-onboard-routine-subscribe'); console.log('Steps:', s.steps.length, 'Branches:', (s.branches.failures||[]).length + (s.branches.alternates||[]).length)"
```

Expected: `Steps: 25 Branches: 4`

Test in browser — select P3, verify auto-play through all steps.

- [ ] **Step 3: Commit**

```bash
git add data/scenarios.json
git commit -m "feat: add P3 박합격 journey scenario (25 steps, 4 branches)

Email signup → onboarding tour → AWS study note → AI card gen
→ first SRS review → daily routine + streak (Epic 9)
→ quota exhaustion → Pro subscription.
Covers Epic 1, 2, 3, 4, 6 with Epic 9 streak insertion."
```

---

### Task 6: Add P4 journey scenario — 최운영: 관리자 일과

**Files:**
- Modify: `data/scenarios.json` (scenarios array)

**Source:** Storyboard §3 P4 Journey Map + §2 Epic 7/11/12

- [ ] **Step 1: Add P4 admin journey scenario**

In `data/scenarios.json`, at the end of the `scenarios` array, add:

```json
    ,
    {
      "id": "journey-p4-admin-daily",
      "title": "P4 최운영: 관리자 일과",
      "category": "journey-admin", "priority": "P0", "status": "implemented",
      "description": "관리자 최운영의 일과: 대시보드 확인 → 신고 처리(dismiss/warn/remove) → GDPR 데이터 내보내기 → 콘텐츠 모더레이션 → 감사 로그 CSV 내보내기. Epic 7, 11을 관통하며 Epic 12(GDPR)가 삽입된다.",
      "actors": ["Browser", "SCR-ADMIN-001", "Gateway", "Platform", "Engagement", "PostgreSQL", "Kafka"],
      "ui": { "layout": "sequence", "tags": ["관리자", "신고처리", "GDPR", "감사로그", "모더레이션"], "estimatedDuration": 3000 },
      "rules": ["§6.5"],
      "files": [],
      "steps": [
        { "n": 1, "from": "Browser", "to": "SCR-ADMIN-001", "type": "ui_action", "action": "관리자 대시보드 진입 → 미처리 신고 8건 · GDPR 요청 3건 확인",
          "payload": { "screen": "SCR-A-ADMIN-001", "element": "처리 대기 카드" }, "duration": 10,
          "annotations": ["P4 최운영", "Epic 7: US-7.1"], "tags": ["관리자"] },
        { "n": 2, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "GET /api/v1/admin/dashboard", "duration": 20 },
        { "n": 3, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "SELECT count(*) FROM reports WHERE status='pending' + GDPR requests", "duration": 10 },
        { "n": 4, "from": "Gateway", "to": "Browser", "type": "http_response", "action": "200 OK {pendingReports: 8, gdprRequests: 3, dau: 1247}",
          "payload": { "pendingReports": 8, "gdprRequests": 3, "dau": 1247, "mau": 5830 }, "duration": 5 },

        { "n": 5, "from": "SCR-ADMIN-001", "to": "SCR-ADMIN-001", "type": "ui_action", "action": "신고 목록 → 상세 검토 시작 (1건: 스팸 덱)",
          "payload": { "screen": "SCR-A-ADMIN-006", "element": "신고 상세 패널" }, "duration": 10,
          "annotations": ["P4 최운영", "Epic 11: US-11.1"] },
        { "n": 6, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "GET /api/v1/admin/reports/{reportId}", "duration": 15 },
        { "n": 7, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "PUT /api/v1/admin/reports/{reportId} {action: 'remove', reason: 'spam'}",
          "payload": { "action": "remove", "reason": "스팸 콘텐츠" }, "duration": 15 },
        { "n": 8, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "UPDATE reports SET status='resolved' + INSERT audit_logs", "duration": 10,
          "annotations": ["audit_logs 자동 기록"] },
        { "n": 9, "from": "Platform", "to": "Kafka", "type": "kafka_publish", "action": "platform.admin.report-resolved-v1",
          "topic": "platform.admin.report-resolved-v1", "duration": 5, "annotations": ["신고자에게 결과 알림 발송"] },

        { "n": 10, "from": "SCR-ADMIN-001", "to": "SCR-ADMIN-001", "type": "ui_action", "action": "GDPR 데이터 내보내기 요청 2건 → '내보내기 실행' 클릭",
          "payload": { "screen": "SCR-A-ADMIN-010", "element": "내보내기 실행 버튼" }, "duration": 10,
          "annotations": ["P4 최운영", "Epic 12: US-12.2, Epic 7: US-7.2"], "tags": ["GDPR"] },
        { "n": 11, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/admin/gdpr/export/{requestId}", "duration": 15 },
        { "n": 12, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "SELECT * FROM notes, cards, reviews, settings WHERE user_id=? → ZIP 생성 (비동기)",
          "duration": 30, "annotations": ["백그라운드 작업 — 최대 72시간"] },
        { "n": 13, "from": "Platform", "to": "Kafka", "type": "kafka_publish", "action": "platform.gdpr.export-completed-v1",
          "topic": "platform.gdpr.export-completed-v1", "duration": 5, "annotations": ["사용자에게 다운로드 링크 이메일 발송"] },

        { "n": 14, "from": "SCR-ADMIN-001", "to": "SCR-ADMIN-001", "type": "ui_action", "action": "콘텐츠 모더레이션 → 공유 덱 상태 변경 (active → removed)",
          "payload": { "screen": "SCR-A-ADMIN-007", "element": "콘텐츠 상태 변경" }, "duration": 10,
          "annotations": ["P4 최운영", "Epic 11: US-11.2"] },
        { "n": 15, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "PUT /api/v1/admin/content/{deckId} {status: 'removed'}", "duration": 15 },
        { "n": 16, "from": "Engagement", "to": "PostgreSQL", "type": "db_query", "action": "UPDATE shared_decks SET status='removed' + INSERT audit_logs", "duration": 10 },

        { "n": 17, "from": "SCR-ADMIN-001", "to": "SCR-ADMIN-001", "type": "ui_action", "action": "감사 로그 → 필터(기간: 이번 달) → CSV 내보내기",
          "payload": { "screen": "SCR-A-ADMIN-004", "element": "CSV 내보내기 버튼" }, "duration": 10,
          "annotations": ["P4 최운영", "Epic 7: US-7.5"], "tags": ["감사로그"] },
        { "n": 18, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "GET /api/v1/admin/audit-logs?from=2026-05-01&to=2026-05-31&format=csv", "duration": 20 },
        { "n": 19, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "SELECT audit_logs WHERE created_at BETWEEN ? AND ? → CSV 직렬화", "duration": 30 },
        { "n": 20, "from": "Gateway", "to": "Browser", "type": "http_response", "action": "200 OK — CSV 다운로드 (Content-Disposition: attachment)", "duration": 5 }
      ],
      "branches": {
        "failures": [
          { "id": "p4-gdpr-timeout", "trigger": "GDPR 내보내기 72시간 초과", "afterStep": 12,
            "result": { "status": 504, "code": "EXPORT_TIMEOUT", "message": "데이터 내보내기 작업 시간 초과" },
            "recovery": "재시도 큐에 자동 등록 → 관리자 알림" }
        ],
        "alternates": [
          { "id": "p4-dismiss-report", "trigger": "신고 내용이 부적절하지 않다고 판단", "afterStep": 6,
            "result": { "message": "dismiss 처리 → audit_logs 기록 + 신고자에게 '정상 콘텐츠' 알림" } },
          { "id": "p4-account-delete-cancel", "trigger": "사용자가 30일 유예 내 로그인 → 삭제 취소", "afterStep": 13,
            "result": { "message": "계정 재활성화 + 삭제 요청 철회 + 확인 이메일 발송" } }
        ]
      }
    }
```

- [ ] **Step 2: Verify and test**

```bash
node -e "const d = require('./data/scenarios.json'); const s = d.scenarios.find(x=>x.id==='journey-p4-admin-daily'); console.log('Steps:', s.steps.length, 'Branches:', (s.branches.failures||[]).length + (s.branches.alternates||[]).length)"
```

Expected: `Steps: 20 Branches: 3`

- [ ] **Step 3: Commit**

```bash
git add data/scenarios.json
git commit -m "feat: add P4 최운영 admin journey scenario (20 steps, 3 branches)

Admin dashboard → report review (dismiss/warn/remove) → GDPR export
→ content moderation → audit log CSV export.
Covers Epic 7, 11 with Epic 12 GDPR insertion."
```

---

### Task 7: Add 3 supplement scenarios

**Files:**
- Modify: `data/scenarios.json` (scenarios array)

- [ ] **Step 1: Add gamification supplement scenario**

```json
    ,
    {
      "id": "supplement-gamification",
      "title": "게이미피케이션: XP → 레벨업 → 배지",
      "category": "supplement", "priority": "P0", "status": "implemented",
      "description": "복습 완료 시 XP 적립 → Engagement Service가 레벨업 판정 → 배지 조건 평가 → 축하 알림 → 리더보드 갱신. Epic 9 전체 흐름.",
      "actors": ["Browser", "SCR-CARD-005", "SCR-GAME-001", "SCR-NOTI-001", "Gateway", "LearningCard", "Engagement", "Platform", "PostgreSQL", "Kafka"],
      "ui": { "layout": "sequence", "tags": ["XP", "레벨업", "배지", "리더보드", "스트릭"], "estimatedDuration": 2000 },
      "rules": [],
      "files": [],
      "steps": [
        { "n": 1, "from": "Browser", "to": "SCR-CARD-005", "type": "ui_action", "action": "복습 세션 완료 → 결과 화면 표시",
          "payload": { "screen": "SCR-W-CARD-005", "element": "복습 결과 요약" }, "duration": 10,
          "annotations": ["Epic 9: US-9.1"] },
        { "n": 2, "from": "LearningCard", "to": "Kafka", "type": "kafka_publish", "action": "learning.card.review-completed-v1",
          "topic": "learning.card.review-completed-v1", "duration": 5 },
        { "n": 3, "from": "Kafka", "to": "Engagement", "type": "kafka_consume", "action": "review-completed 이벤트 수신", "duration": 5 },
        { "n": 4, "from": "Engagement", "to": "PostgreSQL", "type": "db_query", "action": "UPDATE user_xp SET xp = xp + 30 (복습 완료 보상)",
          "duration": 8, "annotations": ["이벤트별 XP: review_complete=30, note_create=20, card_create=10"] },
        { "n": 5, "from": "Engagement", "to": "Engagement", "type": "internal", "action": "레벨업 판정: xp >= required_xp[currentLevel+1]?",
          "duration": 3, "annotations": ["레벨 1~10, 각 레벨별 필요 XP 정의"] },
        { "n": 6, "from": "Engagement", "to": "PostgreSQL", "type": "db_query", "action": "UPDATE user_levels SET level=5, title='학습 전문가'",
          "duration": 5, "annotations": ["레벨 5 달성! 🎉"] },
        { "n": 7, "from": "Engagement", "to": "Engagement", "type": "internal", "action": "배지 조건 평가: '100일 연속 학습' → 충족!",
          "duration": 3, "annotations": ["배지 criteria_json 매칭"] },
        { "n": 8, "from": "Engagement", "to": "Kafka", "type": "kafka_publish", "action": "engagement.gamification.badge-earned-v1",
          "topic": "engagement.gamification.badge-earned-v1", "duration": 5,
          "annotations": ["Platform 구독 → 축하 알림 발송"] },
        { "n": 9, "from": "SCR-CARD-005", "to": "SCR-GAME-001", "type": "ui_action", "action": "축하 모달 표시 → 게이미피케이션 프로필 확인",
          "payload": { "screen": "SCR-W-GAME-001", "element": "레벨업 축하 모달 + 배지 갤러리" }, "duration": 10,
          "annotations": ["Epic 9: US-9.2"] },
        { "n": 10, "from": "Engagement", "to": "PostgreSQL", "type": "db_query", "action": "UPDATE leaderboard SET score=... (주간/월간/전체)",
          "duration": 8, "annotations": ["리더보드 갱신 — Epic 9: US-9.3"] }
      ],
      "branches": {
        "failures": [
          { "id": "xp-cap", "trigger": "일일 XP cap 100 도달", "afterStep": 4,
            "result": { "message": "추가 XP 적립 차단 — '내일 다시 시도하세요' 안내" } }
        ],
        "alternates": [
          { "id": "badge-not-met", "trigger": "배지 조건 미충족", "afterStep": 7,
            "result": { "message": "배지 갤러리에서 미획득 배지 — 남은 조건 표시" } }
        ]
      }
    }
```

- [ ] **Step 2: Add notification supplement scenario**

```json
    ,
    {
      "id": "supplement-notification",
      "title": "알림: 복습 리마인더 → 인앱 알림센터",
      "category": "supplement", "priority": "P0", "status": "implemented",
      "description": "Cron 트리거 → 오늘 복습 카드 조회 → FCM 푸시 알림 발송 → 인앱 알림센터 표시 → 클릭 시 복습 화면 이동. Epic 10 전체 흐름.",
      "actors": ["MobileApp", "SCR-NOTI-001", "SCR-CARD-005", "Gateway", "Platform", "LearningCard", "PostgreSQL", "Kafka", "FCM", "Cron"],
      "ui": { "layout": "sequence", "tags": ["알림", "FCM", "Cron", "리마인더"], "estimatedDuration": 1500 },
      "rules": [],
      "files": [],
      "steps": [
        { "n": 1, "from": "Cron", "to": "Platform", "type": "internal", "action": "매일 09:00 복습 리마인더 Cron 트리거",
          "duration": 5, "annotations": ["Epic 10: US-10.1 — 사용자 타임존 기준"] },
        { "n": 2, "from": "Platform", "to": "LearningCard", "type": "http_request", "action": "GET /internal/cards/due-count?userId={userId}", "duration": 10 },
        { "n": 3, "from": "LearningCard", "to": "PostgreSQL", "type": "db_query", "action": "SELECT count(*) FROM cards WHERE next_review <= NOW() AND user_id=?",
          "duration": 8, "annotations": ["오늘 복습 예정: 15장"] },
        { "n": 4, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "SELECT notification_settings WHERE user_id=? AND category='review'",
          "duration": 5, "annotations": ["알림 설정 확인 — on/off, 방해금지 시간대"] },
        { "n": 5, "from": "Platform", "to": "FCM", "type": "external_api", "action": "POST FCM → '오늘 복습 카드 15장이 기다리고 있어요 📚'",
          "duration": 100, "annotations": ["FCM 푸시 알림 발송"], "tags": ["FCM"] },
        { "n": 6, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "INSERT notifications (type='review_reminder', status='unread')",
          "duration": 5, "annotations": ["인앱 알림 저장"] },
        { "n": 7, "from": "MobileApp", "to": "SCR-NOTI-001", "type": "ui_action", "action": "푸시 알림 클릭 → 인앱 알림센터 확인",
          "payload": { "screen": "SCR-W-NOTI-001", "element": "알림 센터 목록" }, "duration": 10,
          "annotations": ["Epic 10: US-10.5 — 인앱 알림 센터"] },
        { "n": 8, "from": "SCR-NOTI-001", "to": "SCR-CARD-005", "type": "ui_action", "action": "복습 리마인더 클릭 → 복습 세션 화면으로 이동",
          "payload": { "screen": "SCR-W-CARD-005", "element": "복습 시작" }, "duration": 10,
          "annotations": ["Epic 10: US-10.1 — 클릭 시 복습 화면 이동"] }
      ],
      "branches": {
        "failures": [],
        "alternates": [
          { "id": "noti-off", "trigger": "사용자가 복습 알림을 off로 설정", "afterStep": 4,
            "result": { "message": "알림 발송 스킵 — 인앱 알림만 조용히 저장" } },
          { "id": "dnd-hours", "trigger": "방해금지 시간대 (22:00~08:00)", "afterStep": 4,
            "result": { "message": "푸시 발송 지연 → 08:00에 발송 큐 등록" } }
        ]
      }
    }
```

- [ ] **Step 3: Add data export supplement scenario**

```json
    ,
    {
      "id": "supplement-data-export",
      "title": "GDPR: 전체 내보내기 → 계정 삭제",
      "category": "supplement", "priority": "P0", "status": "implemented",
      "description": "사용자가 전체 데이터 내보내기(JSON+ZIP) 요청 → 백그라운드 작업 → 이메일 발송 → 다운로드 → 계정 삭제 요청 → 30일 유예 → hard delete. Epic 12 전체 흐름.",
      "actors": ["Browser", "SCR-BILLING-001", "Gateway", "Platform", "PostgreSQL", "Kafka"],
      "ui": { "layout": "sequence", "tags": ["GDPR", "데이터내보내기", "계정삭제", "30일유예"], "estimatedDuration": 3000 },
      "rules": ["§11.1"],
      "files": [],
      "steps": [
        { "n": 1, "from": "Browser", "to": "SCR-BILLING-001", "type": "ui_action", "action": "설정 → 데이터 관리 → '전체 데이터 내보내기' 클릭",
          "payload": { "screen": "SCR-W-SETTINGS-004", "element": "전체 데이터 내보내기 버튼" }, "duration": 10,
          "annotations": ["Epic 12: US-12.2"], "tags": ["GDPR"] },
        { "n": 2, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/data/export", "duration": 15 },
        { "n": 3, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "INSERT data_export_jobs (status='processing') — 중복 요청 시 409",
          "duration": 8, "annotations": ["진행 중 요청 1건 제한"] },
        { "n": 4, "from": "Gateway", "to": "Browser", "type": "http_response", "action": "202 Accepted {jobId, estimatedCompletion: '72h'}",
          "payload": { "jobId": "export-001", "estimatedCompletion": "72h" }, "duration": 5 },
        { "n": 5, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "SELECT notes, cards, reviews, settings, audit_logs WHERE user_id=? → JSON 직렬화 → ZIP",
          "duration": 50, "annotations": ["백그라운드 비동기 작업"] },
        { "n": 6, "from": "Platform", "to": "Kafka", "type": "kafka_publish", "action": "platform.gdpr.export-completed-v1",
          "topic": "platform.gdpr.export-completed-v1", "duration": 5,
          "annotations": ["이메일 발송 트리거 — 다운로드 링크 24시간 유효"] },
        { "n": 7, "from": "SCR-BILLING-001", "to": "SCR-BILLING-001", "type": "ui_action", "action": "이메일 수신 → 다운로드 링크 클릭 → ZIP 다운로드 완료",
          "payload": { "screen": "SCR-W-SETTINGS-004", "element": "다운로드 링크" }, "duration": 10,
          "annotations": ["Epic 12: US-12.2 — 다운로드"] },

        { "n": 8, "from": "SCR-BILLING-001", "to": "SCR-BILLING-001", "type": "ui_action", "action": "'계정 삭제' 클릭 → 30일 유예 안내 + 복구 불가 경고 확인",
          "payload": { "screen": "SCR-W-SETTINGS-004", "element": "계정 삭제 버튼 + 경고 모달" }, "duration": 10,
          "annotations": ["Epic 12: US-12.3"], "tags": ["계정삭제"] },
        { "n": 9, "from": "Browser", "to": "Gateway", "type": "http_request", "action": "POST /api/v1/account/delete",
          "payload": { "confirmationCode": "ABC123" }, "duration": 15, "annotations": ["이메일 확인 코드로 본인 확인"] },
        { "n": 10, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "UPDATE users SET status='pending_deletion', deletion_date=NOW()+30d",
          "duration": 8, "annotations": ["즉시 비활성화 + 30일 카운트다운 시작"] },
        { "n": 11, "from": "Platform", "to": "Kafka", "type": "kafka_publish", "action": "platform.account.deletion-scheduled-v1",
          "topic": "platform.account.deletion-scheduled-v1", "duration": 5 },
        { "n": 12, "from": "Platform", "to": "PostgreSQL", "type": "db_query", "action": "(30일 후) hard DELETE users, notes, cards, reviews CASCADE + 삭제 확인 메일",
          "duration": 30, "annotations": ["Cron 트리거 — deletion_date <= NOW() 인 계정 영구 삭제"] }
      ],
      "branches": {
        "failures": [
          { "id": "export-duplicate", "trigger": "이미 진행 중인 내보내기 요청 존재", "afterStep": 3,
            "result": { "status": 409, "code": "EXPORT_IN_PROGRESS", "message": "진행 중인 내보내기 작업이 있습니다" },
            "recovery": "기존 작업 완료 대기" },
          { "id": "export-timeout", "trigger": "내보내기 72시간 초과", "afterStep": 5,
            "result": { "status": 504, "code": "EXPORT_TIMEOUT", "message": "내보내기 작업 시간 초과" },
            "recovery": "관리자에게 알림 → 수동 재시도" }
        ],
        "alternates": [
          { "id": "cancel-deletion", "trigger": "유예 기간 내 로그인 → 삭제 취소", "afterStep": 10,
            "result": { "message": "계정 재활성화: UPDATE users SET status='active', deletion_date=NULL" } }
        ]
      }
    }
```

- [ ] **Step 4: Verify all 3 supplement scenarios**

```bash
node -e "const d = require('./data/scenarios.json'); ['supplement-gamification','supplement-notification','supplement-data-export'].forEach(id => { const s = d.scenarios.find(x=>x.id===id); console.log(id, 'Steps:', s.steps.length, 'Branches:', (s.branches.failures||[]).length + (s.branches.alternates||[]).length) })"
```

Expected:
```
supplement-gamification Steps: 10 Branches: 2
supplement-notification Steps: 8 Branches: 2
supplement-data-export Steps: 12 Branches: 3
```

- [ ] **Step 5: Test all supplement scenarios in browser**

Open each supplement scenario and verify auto-play works, architecture/sequence views render, branches display correctly.

- [ ] **Step 6: Commit**

```bash
git add data/scenarios.json
git commit -m "feat: add 3 supplement scenarios (gamification, notification, data export)

supplement-gamification: XP → level-up → badge (10 steps, Epic 9)
supplement-notification: cron → FCM push → notification center (8 steps, Epic 10)
supplement-data-export: GDPR export + account deletion (12 steps, Epic 12)"
```

---

### Task 8: Final verification and integration test

**Files:** None (read-only verification)

- [ ] **Step 1: Verify total data counts**

```bash
node -e "const d = require('./data/scenarios.json'); console.log('Categories:', d.categories.length); console.log('Actors:', d.actors.length); console.log('Scenarios:', d.scenarios.length); let steps=0, branches=0; d.scenarios.forEach(s => { steps += s.steps.length; branches += (s.branches?.failures||[]).length + (s.branches?.alternates||[]).length }); console.log('Total steps:', steps); console.log('Total branches:', branches)"
```

Expected (approximately):
```
Categories: 7
Actors: 56+
Scenarios: 25
Total steps: ~250
Total branches: ~50
```

- [ ] **Step 2: Verify all scenario IDs match category scenarioIds**

```bash
node -e "const d = require('./data/scenarios.json'); d.categories.forEach(c => { c.scenarioIds.forEach(id => { if (!d.scenarios.find(s=>s.id===id)) console.error('MISSING:', id, 'in', c.id) }) }); console.log('All scenario IDs validated')"
```

Expected: `All scenario IDs validated` (no MISSING errors)

- [ ] **Step 3: Verify all actor IDs referenced in steps exist**

```bash
node -e "const d = require('./data/scenarios.json'); const actorIds = new Set(d.actors.map(a=>a.id)); let missing = []; d.scenarios.forEach(s => s.steps.forEach(st => { if (!actorIds.has(st.from)) missing.push(s.id+':'+st.n+' from='+st.from); if (!actorIds.has(st.to)) missing.push(s.id+':'+st.n+' to='+st.to) })); if (missing.length) console.error('Missing actors:', missing); else console.log('All actor references valid')"
```

Expected: `All actor references valid`

- [ ] **Step 4: Browser full test**

Open `index.html` and test each of the 7 new scenarios:
1. Click each scenario in the sidebar
2. Verify Architecture View renders (screen actors appear in green between client and edge rows)
3. Verify Sequence View renders (ui_action steps show dashed green arrows)
4. Press play and let auto-play run through all steps
5. Check Detail Panel shows `payload.screen` and `payload.element` for ui_action steps
6. Check Log Panel shows "UI" label for ui_action steps
7. Click at least one branch in each scenario to verify branch display

- [ ] **Step 5: Verify existing scenarios still work**

Click an existing scenario (e.g., "로그인 (웹·모바일 통합)") and verify:
1. Architecture view renders normally
2. Sequence view renders normally
3. Auto-play works
4. No console errors

- [ ] **Step 6: Commit (if any fixes were needed)**

Only commit if fixes were applied. If everything works, skip this step.

```bash
git add -A
git commit -m "fix: integration test fixes for storyboard scenarios"
```
