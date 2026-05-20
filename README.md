# Synapse Flow Simulator

Synapse 프로젝트의 전체 시스템 흐름을 인터랙티브 애니메이션으로 시각화하는 웹 시뮬레이터.

**Live**: https://team-project-final.github.io/synapse-flow-simulator/

## 기능

- 18개 시나리오 (인증, AI 흐름, 장애, 운영)
- 아키텍처 뷰 + 시퀀스 뷰 탭 전환
- 단계별 클릭 진행 + 자동 재생
- 정상/에러 흐름 전환
- 요청/응답 페이로드 실시간 표시

## 기술 스택

- 순수 HTML/CSS/JS (빌드 도구 없음)
- CSS Animation + SVG
- GitHub Pages 정적 배포

## 시나리오 추가

`data/scenarios.json`에 시나리오 객체를 추가하면 자동으로 UI에 반영됩니다.

## 시나리오 목록

### 🔐 인증 및 보안
1. 로그인 (웹/모바일)
2. OAuth2 (Google/GitHub)
3. MFA TOTP 2차 인증
4. 어드민 RBAC + IDOR 차단
5. JWT 만료 → 리프레시

### 🌊 AI 및 이벤트 흐름
6. 노트 → AI 카드 → fan-in
7. 시맨틱 검색 (pgvector)
8. LLM Fallback (Claude → OpenAI)
9. 복습 → 점수 → 뱃지
10. CloudEvents 멱등성

### 💥 장애 및 복구
11. 서비스 장애 분류 (500)
12. DB 연결 실패 → CrashLoopBackOff
13. Kafka 실패 → DLQ → 재처리
14. Schema 호환성 깨짐 (CI)

### ☁️ 운영 및 GitOps
15. ArgoCD 자동 동기화
16. Image Updater (semver)
17. External Secrets 5분 회전
18. 분산 추적 (traceId)
