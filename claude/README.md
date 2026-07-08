# Claude Code 버전

에이전트 지침을 Claude Code의 세 가지 메커니즘으로 매핑한 버전.

## 구성 및 매핑

| 파일 | Claude Code 메커니즘 | 역할 |
|---|---|---|
| `CLAUDE.md` | 메모리 (항상 로드) | 핵심 원칙 + 작업 규율 + 자체 검토 + 진행 보고 검증 |
| `skills/analysis-protocol/` | Skill (상황별 자동 로드) | 데이터 분석 검증 프로토콜 |
| `skills/design-protocol/` | Skill (상황별 자동 로드) | 설계·구조 변경 프로토콜 |
| `agents/orchestrator.md` | 서브에이전트 | 루프 **계획** 반환 (플래너 — 산출물·직접 호출 금지) |
| `agents/reviewer.md` | 서브에이전트 | 산출물 승인·검증 (별도 컨텍스트) |
| `.claude/memory/` | 프로젝트 메모 | 세션 간 교훈 축적 (한 교훈 = 한 파일) |
| `.claude/state/` | 프로젝트 상태 | 3-에이전트 루프 상태표 (`loop-status.md` 런타임 생성) |

**핵심 이점 — 검증자 분리**: reviewer 서브에이전트는 작성자(메인 에이전트)의 추론 과정을 보지 않고 산출물만 판정하므로, 자기 결과물에 관대해지는 편향이 구조적으로 차단된다. orchestrator 서브에이전트는 **분해·위험등급·라우팅 계획을 메인 에이전트에 반환하는 플래너**이며, 실제 Worker·reviewer 호출과 상태 추적은 메인 에이전트가 수행한다(서브에이전트의 중첩 호출은 버전·안정성에 의존하므로 실제 라우팅 주체를 메인 에이전트로 고정). (Cursor 버전도 `.cursor/agents/`로 동일한 분리를 지원한다.)

## 설치

> **AI 채팅으로 설치**: 대상 프로젝트를 연 채팅에 붙여넣을 Claude Code용 프롬프트는 [루트 README — AI 채팅으로 설치](../README.md#ai-채팅으로-설치-권장) 참조.
>
> **업데이트**: upstream 버전 반영은 [루트 README — 업데이트](../README.md#업데이트-버전-반영) 참조.

**프로젝트 단위 적용** (권장 — repo에 커밋되어 팀 공유):
```bash
# 대상 프로젝트 루트에서
cp <이 repo>/claude/CLAUDE.md ./CLAUDE.md        # 이미 있으면 내용을 병합
mkdir -p .claude
cp -r <이 repo>/claude/skills .claude/skills
cp -r <이 repo>/claude/agents .claude/agents
cp -r <이 repo>/claude/.claude/memory .claude/memory
cp -r <이 repo>/claude/.claude/state .claude/state
```

**개인 전역 적용** (모든 프로젝트에서 사용):
```bash
cp -r <이 repo>/claude/skills/* ~/.claude/skills/
cp -r <이 repo>/claude/agents/* ~/.claude/agents/
# CLAUDE.md 내용은 ~/.claude/CLAUDE.md 에 병합
```

**적용 확인**: Claude Code에서 `/memory`로 CLAUDE.md 로드 확인, `/agents`로 orchestrator·reviewer 등록 확인. `.claude/memory/`, `.claude/state/` 디렉터리 존재 확인. 스킬은 해당 작업을 시키면 자동으로 참조된다.

## 사용법 (프롬프트 가이드)

### 기본 골격
```
[목적] 왜 필요한지
[작업] 무엇을 해달라는지 한 문장
[입력] 참고 파일·데이터 경로
[제약] 형식·범위·제외할 것
```

### 복사해서 쓰는 템플릿

**① 데이터 분석 + 자동 검증**
```
[목적] 이번 주 팀 회의 공유용
[작업] data/sales_2025Q2.csv 에서 채널별 매출 추이와 Q1 대비 변화 분석
[제약] pandas, 결과는 markdown 표

완료되면 reviewer 서브에이전트로 검증받고, 판정표와 함께 결과를 제출해.
반려되면 수정 지시대로 고친 뒤 재검증까지 마쳐. 최대 3회, 그래도 안 되면 멈추고 나에게 보고해.
```
→ 이 한 문장으로 **작성 → 검증 → 수정 → 재검증 루프**가 Claude Code 안에서 돈다.

**② 설계 작업 (체크포인트 방식)**
```
[목적] 사용자 역할 기능 추가 대응
[작업] role_permissions 테이블 설계
[제약] 기존 users.role 값 변경 불가

바로 구현하지 말고, 설계 구조(변경 테이블 / 마이그레이션 순서 / 영향 코드)만
먼저 제안해서 내 확인을 받아. 결정/미결정을 구분하고 미결정은 옵션+트레이드오프로.
```

**③ 대량 변경 (샘플 승인 방식)**
```
[작업] src/ 전체에서 구 API 호출을 신규 API로 마이그레이션
[제약] 테스트 파일 제외

1. 대상 파일 목록과 변경 패턴을 먼저 보여줘
2. 파일 1개 diff를 샘플로 보여줘
3. 내가 승인하면 전체 진행하고, 완료 후 reviewer 검증을 거쳐
```

**④ 수정 요청**
```
[수정] 3번 섹션: 어조가 너무 단정적 → 추정 표현으로 완화
[수정] 집계 표: 합계 불일치 → 재검증
지적한 두 항목만 수정. 다른 부분은 건드리지 마.
```

**⑤ 3-에이전트 루프 (orchestrator + Worker + reviewer)**
```
[목적] 대규모 리팩터링을 단계별로 안전하게 진행
[작업] src/ 의 구 API 호출을 신규 API로 마이그레이션
[제약] 테스트 파일 제외, 되돌리기 어려운 DB 변경 없음

orchestrator 서브에이전트로 이 작업의 분해·위험 등급(하/중/상)·라우팅 계획을 만들어줘.
그 계획을 내가 승인하면, 너(메인 에이전트)가 계획대로 Worker를 진행시키고 각 단계 산출물은 reviewer로 검증해.
반려는 최대 3회, 같은 사유 2연속 반려 시 멈추고 나에게 에스컬레이션해.
```
→ orchestrator는 **계획을 반환**하고, 실제 루프(Worker 진행·reviewer 호출·상태 추적)는 메인 에이전트가 돈다. 반려 횟수·작업 상태는 `.claude/state/loop-status.md`에 기록하고 **매 판정 후 갱신**한다 (상세: `.claude/state/README.md`). 처음에는 **reviewer만** 쓰고, 루프가 안정되면 orchestrator 플래너를 추가하는 것을 권장 (`docs/multi-agent-orchestration.md` 9장).

### 모호함 처리 모드
- 정밀 우선: `애매하면 진행하지 말고 질문해.`
- 속도 우선: `애매한 건 합리적으로 가정하고, 가정 목록을 결과에 명시해.`

## 검증 루프 운영 규칙

`docs/multi-agent-orchestration.md`의 종료 조건을 그대로 따른다. 반려 횟수·작업 상태는 `.claude/state/loop-status.md`에 기록하고 **매 판정 후 갱신**한다 (상세: `.claude/state/README.md`).

| 상황 | 처리 |
|---|---|
| reviewer 통과 | 사용자에게 판정표와 함께 제출 |
| 반려 1~2회 | 수정 지시 항목만 고쳐 재검증 |
| 같은 사유 반려 2회 연속 / 반려 3회 | 멈추고 사용자에게 이력과 함께 보고 |
| reviewer "판정 불가" | 멈추고 사용자에게 보고 |

되돌리기 어려운 작업(DB 변경, 외부 전달물)은 reviewer 통과 후에도 사용자 최종 승인을 받는다.

## 병렬 Worker 가이드

의존 관계 없는 하위 작업은 **병렬 서브에이전트**에 위임하고, 메인 에이전트는 다른 작업을 계속한다.

- **병렬에 적합**: 서로 독립된 조사·분석·파일 탐색 (예: A 모듈 분석 + B 모듈 분석)
- **병렬에 부적합**: 이전 결과에 의존하는 작업, 같은 파일을 동시에 수정하는 작업
- **reviewer는 완료 후**: 병렬 Worker가 각자 산출물을 제출한 뒤, 통합·검증 단계에서 reviewer를 호출한다. 병렬 진행 중간에 reviewer를 돌리지 않는다.
- **3-에이전트 루프**: orchestrator + Worker + reviewer 전체 루프를 쓸 때, orchestrator 플래너는 분해·라우팅 **계획**을 반환하고, 실제 상태 추적·라우팅·서브에이전트 호출은 **메인 에이전트**가 수행한다. orchestrator는 산출물을 직접 만들지 않는다.

## 알려진 한계
- reviewer와 작성자가 같은 모델이므로 맹점을 공유할 수 있다. 통과된 산출물도 주기적으로 사람이 샘플 검수하고, 오판 사례를 reviewer.md 체크리스트에 반영할 것.
- Claude Code reviewer는 `tools` 화이트리스트로 Bash를 허용하므로, Cursor의 `readonly`처럼 구조적으로 수정을 차단하지는 않는다. "직접 수정하지 않는다"는 행동 지침에 의존한다.
- CLAUDE.md가 길수록 준수율이 떨어진다. 항목 추가 시 덜 중요한 항목 제거를 함께 고려.
