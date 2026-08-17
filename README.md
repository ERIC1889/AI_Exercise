# FormCoach

**홈트레이닝 초보~중급자가 "지금 내 자세가 맞는지, 오늘 얼마나 해야 하는지, 어느 정도 성장하고 있는지"를 한 곳에서 추적·분석·피드백받는 모바일 앱.** 졸업논문 + 백엔드 포트폴리오 겸용 프로젝트 (2026-07 ~ 12).

- 영상은 기기 밖으로 나가지 않는다 — 온디바이스 MediaPipe로 관절 **좌표만** 서버 전송 (ADR-1)
- 피드백 2계층 — 즉시 교정 알림(규칙, 300ms급) + AI 코치 메시지(LLM Tool Calling, 세트 사이) (ADR-3)
- 사용자 수준(입문🌱/성장🔥)에 따라 알림 밀도·계획·리포트가 달라지는 **코칭 프로파일** (ADR-7)
- 연구 질문 3개: RQ1 이벤트 기반 vs 동기 아키텍처 / RQ2 Tool-using Agent 코칭 품질 / RQ3 자세 오류 판별 모델 비교

상세는 **[기획서.md](./기획서.md)** (v3.0 — 문제 정의·경쟁 분석·요구사항·화면·ADR·개발 계획). 기술설계서·협업컨벤션 등 나머지 팀 문서는 별도 공유.

> 현재 상태: **디렉토리 스켈레톤 단계.** 설정 파일·의존성·CI는 팀 검토 완료 후 셋업한다 (셋업 절차는 협업컨벤션 Part 9).

## 구조

| 경로 | 역할 | 오너 |
|---|---|---|
| `docs/` | ADR·연구 프로토콜 (measurement-spec, labeling-protocol 등) | 공동 (변경 시 상호 승인) |
| `packages/contracts/` | ★ 단일 계약 원본 — JSON Schema, fixture, TS/Python 생성 타입 | 공동 (변경 시 상호 승인) |
| `packages/domain-core/` | 순수 TS 도메인 규칙 (accuracy 수식, 승급 규칙 포함) | 준영 |
| `packages/database/` | Prisma schema + migrations | 준영 |
| `apps/mobile/` | React Native + Expo + MediaPipe 온디바이스 (좌표만 추출, 영상 미전송) | 준영 |
| `apps/api/` | NestJS 백엔드 — 아래 모듈 10개 + AI Agent(Tool Calling) 포함 | 준영 |
| `apps/api/src/architecture/` | ports(ProcessingPipelinePort 등) / sync·event 어댑터 — RQ1 비교 구조 | 준영 |
| `services/vision/` | Python FastAPI 분석 서비스 — 반복 카운트·오류 판별. **코칭 프로파일과 무관** | 친구 (셀프 머지 구역) |
| `tools/` | JSONL 리플레이어(E1 부하 도구), mock-vision, contract-check | 준영 |
| `experiments/` | rq1-architecture(준영) / rq2-coaching(준영) / rq3-posture(친구) | RQ별 오너 |
| `data/` | 연구 데이터 — **git 커밋 금지** (README·manifest·소형 샘플만 예외) | 친구 |
| `infra/` | Dockerfile + compose (base/sync/event/experiment) | 준영 |
| `.github/workflows/` | CI 4종: ci-typescript · ci-python · contract-compatibility · compose-smoke | 공동 |

**`apps/api/src/modules/`** — session(생명주기) · ingest(WS 수신) · repetition(카운트 집계) · posture(오류 이벤트) · safety(규칙 엔진 — 통증·강도 상한) · **profile(수준·코칭 프로파일·승급 규칙)** · plan(Planner Agent) · coaching(Coaching Agent) · report(Report Agent) · notification(Tier 1 피드백 push — 프로파일 필터 적용 지점)

## 핵심 원칙 4가지

1. `ARCH_MODE`는 부트스트랩에서 어댑터 1개만 선택 — 비즈니스 로직에 분기 금지 (RQ1 공정 비교)
2. 인제스트는 sync/event 양쪽 모두 WebSocket — 차이는 Gateway 이후만
3. 모든 메시지에 envelope — eventId, schemaVersion, frameSeq, 타임스탬프 4종
4. 분석 서비스는 코칭 프로파일의 존재를 모른다 — 프로파일 적용은 백엔드 피드백·Agent 계층에서만 (ADR-7)

## 일정 (2026-08-17 재산정)

M0 계약·PoC(~9월 첫째 주, RN 포즈 추출 스파이크 최우선) → **M1 스쿼트 E2E 게이트(~10월 중순)** → M2 AI 코치·프로파일(~11월 중순) → M3 실험(~12월 첫째 주) → M4 논문(12월). M1 미달 시 부가 기능 컷.
