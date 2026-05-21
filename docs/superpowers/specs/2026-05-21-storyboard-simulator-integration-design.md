# 스토리보드 #8 → 인터랙티브 시뮬레이터 통합 설계

> **작성일**: 2026-05-21  
> **프로젝트**: Synapse Flow Simulator  
> **소스 문서**: [08_스토리_보드 v2.0](https://github.com/team-project-final/documents/wiki/08:-%EC%8A%A4%ED%86%A0%EB%A6%AC-%EB%B3%B4%EB%93%9C), [05_화면_흐름_시퀀스_다이어그램](https://github.com/team-project-final/documents/wiki/05:-%ED%99%94%EB%A9%B4-%ED%9D%90%EB%A6%84-%EC%8B%9C%ED%80%80%EC%8A%A4-%EB%8B%A4%EC%9D%B4%EC%96%B4%EA%B7%B8%EB%9E%A8)

---

## 1. 목표

스토리보드 #8의 12개 Epic, 49+ User Story를 Synapse Flow Simulator에 **인터랙티브 시나리오**로 추가한다. 사용자 관점(화면 전환, UI 행위)과 백엔드 관점(서비스 간 통신)을 하이브리드로 표현하여, 시뮬레이터가 기술 흐름뿐 아니라 **제품 흐름**까지 시각화하도록 확장한다.

## 2. 접근 방식

**데이터 중심 확장** — `data/scenarios.json`에 새 카테고리·액터·시나리오를 추가하고, `index.html`에는 새로운 step type(`ui_action`) 렌더링 로직만 최소 추가한다. 기존 Architecture View / Sequence View에서 스토리보드 시나리오를 즉시 동작시킨다.

## 3. 카테고리 구조

기존 4개 카테고리를 그대로 유지하고, 스토리보드 전용 3개 카테고리를 추가한다.

### 기존 (변경 없음)

| ID | 이름 | 시나리오 수 |
|---|---|---|
| `auth` | 인증 및 보안 | 5 |
| `flow` | AI 및 이벤트 흐름 | 5 |
| `failure` | 장애 및 복구 | 4 |
| `ops` | 운영 및 GitOps | 4 |

### 신규

| ID | 이름 | 색상 | 설명 |
|---|---|---|---|
| `journey-learner` | 학습자 여정 | green | P1·P2·P3 페르소나 여정 시나리오 |
| `journey-admin` | 관리자 여정 | purple | P4 관리자 일과 시나리오 |
| `supplement` | 보충 흐름 | cyan | 여정에서 독립적으로 다뤄야 할 Epic 흐름 |

## 4. 시나리오 목록 (7개)

### 학습자 여정 (3개)

#### `journey-p1-note-graph-share` — P1 김시냅스: 노트→그래프→그룹 공유

- **관통 Epic**: 1, 2, 4, 5, 6, 8 + Epic 9 XP 삽입
- **예상 스텝**: 20개
- **여정 흐름**: GitHub OAuth 로그인 → 마크다운 노트 작성 + 위키링크 → 백링크 확인 → 그래프 뷰 탐색 → AI 카드 생성 → 스터디 그룹 가입 → 덱 공유 (XP 적립) → 할당량 소진 → Pro 구독
- **브랜치**: OAuth 실패, 노트 자동저장 충돌(409), AI 카드 생성 실패(LLM 폴백), 결제 실패, 모바일 앱 로그인(alternate), 무료 유지(alternate)

#### `journey-p2-paper-ai-search` — P2 이연구: 논문→AI카드→시맨틱검색

- **관통 Epic**: 1, 2, 3, 4, 6 + Epic 10 알림 삽입
- **예상 스텝**: 22개
- **여정 흐름**: Google 로그인 → MFA 설정 → 논문 노트 작성(LaTeX) → AI 카드 자동 생성(20장) → 카드 검토·편집·저장 → 일일 복습 세션 → 시맨틱 검색 → RAG Q&A → 14일 무료 체험 → Pro 구독
- **브랜치**: MFA 검증 실패, AI 카드 품질 낮음(편집/삭제), 시맨틱 검색 결과 0건(RAG 권장), 복습 리마인더 알림(Epic 10)

#### `journey-p3-onboard-routine-subscribe` — P3 박합격: 첫사용→복습루틴→구독

- **관통 Epic**: 1, 2, 3, 4, 6 + Epic 9 스트릭
- **예상 스텝**: 25개
- **여정 흐름**: 앱 설치 → Google 로그인 → 온보딩 투어 → 샘플 노트 확인 → AWS 학습 노트 작성 → AI 카드 생성 → 첫 복습 세션(SM-2) → 매일 복습 루틴 + 스트릭(Epic 9) → 무료 할당량 소진 → Pro 구독
- **브랜치**: 이메일 인증 미완료, 온보딩 스킵, 복습 스트릭 리셋, 결제 실패

### 관리자 여정 (1개)

#### `journey-p4-admin-daily` — P4 최운영: 관리자 일과

- **관통 Epic**: 7, 11 + Epic 12 GDPR
- **예상 스텝**: 18개
- **여정 흐름**: 관리자 대시보드 진입 → 미처리 신고 8건·GDPR 요청 3건 확인 → 신고 상세 검토(dismiss/warn/remove) → GDPR 데이터 내보내기 실행 → 콘텐츠 모더레이션 → 감사 로그 CSV 내보내기 → 일과 종료
- **브랜치**: 신고 dismiss(alternate), GDPR 내보내기 타임아웃, 계정 삭제 취소(30일 유예 내 로그인)

### 보충 흐름 (3개)

#### `supplement-gamification` — XP 적립→레벨업→배지

- **Epic**: 9
- **예상 스텝**: 10개
- **흐름**: 복습 완료 → XP 적립 이벤트 → Engagement Service 처리 → 레벨업 판정 → 배지 조건 평가 → 축하 알림 → 리더보드 갱신
- **브랜치**: 일일 XP cap 도달, 배지 조건 미충족

#### `supplement-notification` — 복습 리마인더→인앱 알림센터

- **Epic**: 10
- **예상 스텝**: 8개
- **흐름**: Cron 트리거 → 오늘 복습 카드 조회 → 푸시 알림 발송(FCM) → 인앱 알림센터 표시 → 사용자 클릭 → 복습 화면 이동
- **브랜치**: 알림 off 설정, 방해금지 시간대

#### `supplement-data-export` — GDPR 전체 내보내기→계정 삭제

- **Epic**: 12
- **예상 스텝**: 12개
- **흐름**: 설정 화면 → 전체 데이터 내보내기 요청 → 백그라운드 작업 → 이메일 발송 → 다운로드 → 계정 삭제 요청 → 이메일 확인 → 30일 유예 → hard delete
- **브랜치**: 중복 요청(409), 유예 기간 내 삭제 취소, 내보내기 타임아웃

## 5. 액터 설계

### 새 화면 액터 (16개)

`kind: "screen"` — Architecture View에서 `client`와 `edge` 사이 행에 배치한다.

| 액터 ID | label | icon | 주요 Epic |
|---|---|---|---|
| `SCR-AUTH-001` | 로그인 화면 | ti-app-window | 1 |
| `SCR-AUTH-002` | 회원가입 화면 | ti-app-window | 1 |
| `SCR-AUTH-003` | MFA 설정 | ti-app-window | 1 |
| `SCR-DASH-001` | 대시보드 | ti-app-window | 3, 9 |
| `SCR-NOTE-001` | 노트 목록 | ti-app-window | 2 |
| `SCR-NOTE-002` | 노트 에디터 | ti-app-window | 2, 4 |
| `SCR-GRAPH-001` | 전체 그래프 | ti-app-window | 5 |
| `SCR-CARD-001` | 덱 목록 | ti-app-window | 3 |
| `SCR-CARD-005` | 복습 세션 | ti-app-window | 3 |
| `SCR-SEARCH-001` | 검색 | ti-app-window | 4 |
| `SCR-BILLING-001` | 플랜/결제 | ti-app-window | 6 |
| `SCR-COMM-001` | 커뮤니티 목록 | ti-app-window | 8 |
| `SCR-COMM-004` | 공유 덱 탐색 | ti-app-window | 8 |
| `SCR-GAME-001` | 게이미피케이션 프로필 | ti-app-window | 9 |
| `SCR-NOTI-001` | 알림 센터 | ti-app-window | 10 |
| `SCR-ADMIN-001` | 관리자 대시보드 | ti-app-window | 7, 11 |

모든 화면 액터는 `color: "green"`으로 통일한다.

### 기존 액터 (변경 없음)

Browser, MobileApp, Gateway, Platform, Knowledge, Engagement, LearningCard, LearningAI, PostgreSQL, Redis, pgvector, Elasticsearch, Kafka, Claude API, OpenAI, Stripe, GitHub OAuth, ArgoCD 등 기존 40+ 액터를 그대로 유지한다.

## 6. Step Type 확장

### 새 type: `ui_action`

```json
{
  "n": 1,
  "from": "Browser",
  "to": "SCR-AUTH-001",
  "type": "ui_action",
  "action": "GitHub 로그인 버튼 클릭",
  "payload": {
    "screen": "SCR-W-AUTH-001",
    "element": "OAuth 버튼"
  },
  "duration": 10,
  "annotations": ["P1 김시냅스", "Epic 1: US-1.1"],
  "tags": ["OAuth", "GitHub"]
}
```

#### 필드 규칙

- `from`/`to`: 화면 액터 ID(SCR-*) 또는 기존 클라이언트 액터(Browser, MobileApp)
- `payload.screen`: 스토리보드/화면정의서의 원본 화면 ID (SCR-W-*, SCR-A-* 형식)
- `payload.element`: 사용자가 상호작용하는 UI 요소
- `annotations`: 페르소나 이름과 Epic/US 참조를 포함
- `duration`: 사용자 행위이므로 짧게 (10~30ms)

#### 렌더링 규칙

| 뷰 | ui_action 표현 |
|---|---|
| Architecture View | 점선 bezier 화살표 + 초록색 + `ti-app-window` 아이콘 |
| Sequence View | 점선 수평 화살표 + 초록색 레이블 |
| Detail Panel | `screen`, `element` 필드 표시 + 페르소나/US 참조 |
| Log Panel | 기존 type별 색상 시스템에 `ui_action: green` 추가 (자동 적용) |

## 7. index.html 코드 변경

### 7.1 CSS 추가 (~5줄)

```css
--color-ui-action: #4ade80;
--kind-screen: #4ade80;
```

### 7.2 JS 색상 맵 확장 (~4줄)

기존 CSS 커스텀 프로퍼티 기반 맵(D03 리팩토링 패턴)에 추가:

```js
// stepColors
ui_action: '--color-ui-action'

// kindColors
screen: '--kind-screen'
```

### 7.3 Architecture View — `renderArchView()` (~10줄)

- `kindOrder` 배열에 `'screen'`을 `'client'` 뒤에 삽입
- `ui_action` step의 화살표를 `stroke-dasharray: "6,4"`로 렌더링

### 7.4 Sequence View — `renderSeqView()` (~8줄)

- `ui_action` step의 화살표를 점선으로 렌더링
- 레이블 색상을 `--color-ui-action`으로 적용

### 7.5 Detail Panel — `renderDetail()` (~8줄)

- `ui_action` type일 때 `payload.screen`, `payload.element` 필드를 표시
- `annotations`에 포함된 페르소나/US 참조를 강조 표시

### 총 변경량: ~35줄

## 8. 데이터 크기 예상

| 항목 | 현재 | 추가 | 합계 |
|---|---|---|---|
| 카테고리 | 4 | 3 | 7 |
| 액터 | 40+ | 16 | 56+ |
| 시나리오 | 18 | 7 | 25 |
| 스텝 (총) | 135 | ~115 | ~250 |
| 브랜치 (총) | 25+ | ~28 | ~53 |
| scenarios.json 크기 | 54KB | ~45KB | ~99KB |

## 9. 시나리오-스토리보드 매핑표

모든 Epic/US가 최소 하나의 시나리오에 매핑되는지 확인한다.

| Epic | 여정 시나리오 | 보충 시나리오 |
|---|---|---|
| 1. 인증/계정 | P1(US-1.1), P2(US-1.2), P3(US-1.3) | — |
| 2. 노트 관리 | P1(US-2.1,2.2), P2(US-2.3), P3(US-2.1) | — |
| 3. 카드/SRS | P2(US-3.2), P3(US-3.1,3.2,3.3) | — |
| 4. AI 기능 | P1(US-4.1), P2(US-4.1,4.2,4.4) | — |
| 5. 지식 그래프 | P1(US-5.1) | — |
| 6. 빌링 | P1(US-6.1), P2(US-6.1), P3(US-6.1) | — |
| 7. 관리자 | P4(US-7.1,7.2,7.3,7.4,7.5) | — |
| 8. 커뮤니티 | P1(US-8.1,8.2) | — |
| 9. 게이미피케이션 | P1(US-9.1 삽입), P3(US-9.4 삽입) | supplement-gamification |
| 10. 알림 | P2(US-10.1 삽입) | supplement-notification |
| 11. 모더레이션 | P4(US-11.1,11.2) | — |
| 12. 데이터/프라이버시 | P4(US-12.2 삽입) | supplement-data-export |

**커버리지**: 12/12 Epic, 주요 P0 User Story 전수 매핑 완료.  
일부 P1 User Story(US-2.4 폴더 구조, US-2.5 버전 히스토리, US-3.4 덱 관리, US-3.5 Anki 가져오기, US-3.6 커스텀 복습, US-4.3 노트 요약, US-5.2 로컬 그래프, US-6.2 플랜 변경, US-6.3 인보이스, US-8.3 공유 덱 탐색, US-8.6 노트 공유, US-9.2 배지, US-9.3 리더보드, US-11.3 게이미피케이션 설정, US-12.1 노트 내보내기)는 브랜치 또는 annotation으로 참조한다.

## 10. 구현 순서

1. `data/scenarios.json`에 새 카테고리 3개 추가
2. `data/scenarios.json`에 화면 액터 16개 추가
3. `index.html`에 CSS 변수 + JS 색상 맵 확장
4. `index.html`에 Architecture/Sequence View `ui_action` 렌더링 로직 추가
5. `index.html`에 Detail Panel `ui_action` 표시 로직 추가
6. `data/scenarios.json`에 7개 시나리오 + 스텝 + 브랜치 데이터 작성
7. 배포된 사이트에서 전체 시나리오 동작 확인
