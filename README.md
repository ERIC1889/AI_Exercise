# FormCoach

카메라 기반 실시간 자세 분석 + AI 코치 운동 코칭 서비스. 졸업논문 겸 포트폴리오 프로젝트.

> 현재 상태: **디렉토리 스켈레톤만 생성된 단계.** 설정 파일·의존성·CI는 팀 검토 완료 후 셋업한다.
> 기획서·기술설계서·협업컨벤션 등 팀 문서는 검토 완료 후 `docs/`로 들어올 예정.

## 구조

| 경로 | 역할 | 오너 |
|---|---|---|
| `docs/` | 기획서·기술설계서·ADR·연구 프로토콜 (검토 후 이동 예정) | 공동 (변경 시 상호 승인) |
| `packages/contracts/` | ★ 단일 계약 원본 — JSON Schema, fixture, TS/Python 생성 타입 | 공동 (변경 시 상호 승인) |
| `packages/domain-core/` | 순수 TS 도메인 규칙 (accuracy 수식 포함) | 준영 |
| `packages/database/` | Prisma schema + migrations | 준영 |
| `apps/mobile/` | React Native + Expo + MediaPipe 온디바이스 (좌표만 추출, 영상은 전송 안 함) | 준영 |
| `apps/api/` | NestJS — 모듈(session·ingest·repetition·posture·safety·plan·coaching·report·notification) + AI Agent 모듈 | 준영 |
| `apps/api/src/architecture/` | ports(ProcessingPipelinePort 등) / sync·event 어댑터 — RQ1 비교 구조 | 준영 |
| `services/vision/` | Python FastAPI 분석 서비스 | 친구 (셀프 머지 구역) |
| `tools/` | JSONL 리플레이어, mock-vision, contract-check | 준영 |
| `experiments/` | rq1-architecture(준영) / rq2-coaching(준영) / rq3-posture(친구) | RQ별 오너 |
| `data/` | 연구 데이터 — **git 커밋 금지** (README·manifest·소형 샘플만 예외) | 친구 |
| `infra/` | Dockerfile + compose (base/sync/event/experiment) | 준영 |
| `.github/workflows/` | CI 4종: ci-typescript · ci-python · contract-compatibility · compose-smoke | 공동 |

## 핵심 원칙 3가지

1. `ARCH_MODE`는 부트스트랩에서 어댑터 1개만 선택 — 비즈니스 로직에 분기 금지
2. 인제스트는 sync/event 양쪽 모두 WebSocket — 차이는 Gateway 이후만
3. 모든 메시지에 envelope — eventId, schemaVersion, frameSeq, 타임스탬프 4종
