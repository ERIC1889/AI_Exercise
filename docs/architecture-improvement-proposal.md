# FormCoach 코드베이스 개선안 (v1)

> 작성일: 2026-08-11
> 근거: Codex 기반 코드베이스/구조 분석 (현재 디렉토리 스켈레톤 + README.md 단계)
> 상태: 초안 — 팀 검토 후 반영

이 문서는 구현이 본격적으로 시작되기 전, 현재 README에 정리된 아키텍처 원칙과
모노레포 디렉토리 구조를 점검하고, 실제 구현 단계에서 원칙이 흔들리지 않도록
미리 합의해야 할 사항을 정리한다.

---

## 1. 설계 원칙 관련 개선안

### 1-1. `ARCH_MODE` 정의를 명확히 한다
현재 원칙: *"`ARCH_MODE`는 부트스트랩에서 어댑터 1개만 선택 — 비즈니스 로직에 분기 금지"*

- [ ] `ARCH_MODE`가 **프로세스 단위로 고정**되는지, 런타임 중 변경 가능한지 문서화
- [ ] 테스트 코드에서 `ARCH_MODE`를 어떻게 주입/오버라이드하는지 정의
- [ ] 실험 산출물(로그, 리포트, `experiments/rq1-architecture`)에 선택된 모드가
      항상 함께 기록되도록 규약 추가
- [ ] `ARCH_MODE` 분기는 **composition root(NestJS 부트스트랩/모듈 조립 지점)** 에만
      허용하고, 서비스 내부 코드에 `if (sync) / if (event)` 형태가 생기면
      코드 리뷰에서 반려하는 규칙을 팀 컨벤션에 명시

### 1-2. sync/event 인제스트의 "Gateway 이후" 계약 차이를 명세한다
현재 원칙: *"인제스트는 sync/event 양쪽 모두 WebSocket — 차이는 Gateway 이후만"*

WebSocket 표면이 같아도 아래 항목은 sync/event 모드에 따라 의미가 달라질 수
있으므로, 각 모드별로 명시적으로 정의해야 한다.

- [ ] 백프레셔(back-pressure) 처리 방식
- [ ] 재시도(retry) 정책과 멱등성 보장 범위
- [ ] 메시지 순서 보장 여부
- [ ] 중복 메시지 처리 의미 (dedup 기준)

### 1-3. Envelope 4종 필드의 의미를 스키마 수준에서 확정한다
현재 원칙: *"모든 메시지에 envelope — eventId, schemaVersion, frameSeq, 타임스탬프 4종"*

- [ ] 타임스탬프가 **클라이언트 생성 시각**인지 **서버 수신 시각**인지 (또는 둘 다 필요한지)
- [ ] `frameSeq`가 **세션 단위**인지 **스트림 단위**인지
- [ ] `eventId` 중복 수신 시 처리 방식 (무시/에러/멱등 처리)
- [ ] `packages/contracts/schemas/websocket`에서 위 4개 필드를 **required**로 강제
- [ ] `fixtures/valid` / `fixtures/invalid`에 다음 케이스를 초기부터 포함:
  - envelope 필드 누락
  - 잘못된 timestamp 형식
  - 중복 `eventId`
  - 역전된 `frameSeq` (순서 뒤바뀜)

---

## 2. 모노레포 디렉토리 구조 관련 개선안

### 2-1. `apps/api/src` 내부 경계 문서화
`architecture/ports`, `architecture/sync`, `architecture/event`, `modules/` 사이에서
- [ ] 어떤 코드가 **포트(port)** 에만 의존하고, 어떤 코드가 **어댑터**에 의존하는지
      경계 규칙을 `docs/adr/`에 ADR로 남긴다.

### 2-2. 컴포넌트 간 계약 스키마 커버리지 확인
`apps/web`(MediaPipe 좌표 추출) ↔ `services/vision`(Python FastAPI 분석) 사이에서
- [ ] 좌표 스키마, 분석 요청/응답 스키마가 `packages/contracts/`에 모두 포함되는지 점검
- [ ] 누락된 스키마가 있다면 `packages/contracts/schemas/`에 추가

### 2-3. `data/` 디렉토리 운영 규칙 구체화
README상 "git 커밋 금지"만 명시되어 있음.
- [ ] `.gitignore`에 `data/` 커밋 금지 규칙 추가
- [ ] manifest 파일 위치/포맷 정의
- [ ] 예외적으로 허용되는 소형 샘플 데이터의 크기/형식 기준 정의

### 2-4. 테스트 경계 디렉토리 합의
현재 CI 4종(ci-typescript, ci-python, contract-compatibility, compose-smoke)과
contract-check는 있으나, 아래는 아직 구조적으로 정의되지 않음.
- [ ] 단위 테스트 배치 규칙 (패키지별 `__tests__` vs `tests/` 등)
- [ ] 통합/e2e 테스트 디렉토리 위치
- [ ] 실험(`experiments/`) 검증 결과물 저장 위치 및 포맷

---

## 3. 구현 단계 리스크 및 조언

| 리스크 | 조언 |
|---|---|
| `ARCH_MODE` 분기가 비즈니스 로직에 퍼짐 | composition root 외 분기 금지를 린트/코드리뷰 규칙으로 강제 |
| WebSocket 프로토콜 버전 관리 부재 | `schemaVersion` 기준 하위 호환 규칙과 breaking change 기준을 `packages/contracts/`에 문서화 |
| envelope 필드가 "관례"로만 지켜짐 | JSON Schema에서 required로 강제 + invalid fixture로 회귀 테스트 |
| sync/event 비교 실험 결과 불일치 | 동일 입력 JSONL을 두 모드로 재생했을 때 지표가 비교 가능하도록 `tools/replay` ↔ `experiments/rq1-architecture` 인터페이스를 먼저 고정 |
| 생성 타입(`generated/typescript`, `generated/python`)과 원본 스키마 drift | CI `contract-compatibility`에서 재생성 후 diff 검사 자동화 |
| `services/vision`(친구 셀프 머지) 이 `packages/contracts`(공동 승인) 를 바꿔야 하는 상황 | 계약 변경 절차를 ADR로 먼저 정의해 오너십 충돌 방지 |

---

## 4. 다음 액션

1. 위 체크리스트 중 **원칙 관련 항목(1-1~1-3)** 을 우선 팀 리뷰 후 README/ADR에 반영
2. `packages/contracts/schemas/`에 envelope 및 컴포넌트 간 스키마 초안 작성
3. `docs/adr/`에 "ARCH_MODE 경계", "계약 변경 절차" ADR 2건 작성
4. CI 셋업 시 `contract-compatibility`에 스키마-생성타입 drift 검사 포함
