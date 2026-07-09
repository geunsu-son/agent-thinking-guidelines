# Agent Thinking Guidelines

AI 에이전트가 설계·분석 작업을 정확하고 깊이 있게 수행하도록 만드는 지침 세트.
동일한 원본 지침(`docs/`)을 **Cursor 버전**과 **Claude Code 버전**으로 각각 매핑했다.

## 기본 적용 모드: 호출 시에만 (옵트인)

지침은 품질을 올리지만 **토큰·리소스 비용이 크다.** 그래서 설치 직후 기본값은 **항상 자동 적용이 아니라, 호출했을 때만 사용**이다.

| 호출 방법 | 예시 | 효과 |
|---|---|---|
| **1. 전문 지침 첨부** | `@docs/agent-thinking-guidelines.md` | 해당 턴(및 첨부된 동안) 핵심 원칙·규율 적용 |
| **2. 검증 서브에이전트** | `/reviewer …` | 산출물 품질 검증 (별도 컨텍스트) |
| **3. 계획 서브에이전트** | `/orchestrator …` | 작업 분해·위험 등급·라우팅 계획 |

설치·업데이트가 끝나면 에이전트가 위 호출법을 안내한 뒤, **"항상 적용되도록 적용할까요?"** 를 묻는다. `예`면 Cursor는 `alwaysApply: true`로, Claude Code는 `CLAUDE.always.md` 내용을 `CLAUDE.md`에 반영한다.

## 이 지침이 하는 것

- **모호함 처리 기준 내재화**: "틀리면 재작업인가?"를 기준으로 질문/가정을 스스로 판단
- **불확실성 강제 표기**: 확인됨/추정/가정을 구분, 그럴듯한 빈칸 채우기 차단
- **검증 프로토콜**: 샘플 검증 후 전체 확장, 행 수 추적, 집계 역산 검증
- **범위 규율**: 지시받은 것만 수정, 발견한 문제는 보고만 (임의 리팩터링 차단)
- **자기신고 문화**: 가정·미해결 사항을 숨기지 않도록 인센티브 설계

하지 못하는 것: 모델의 판단 능력 자체를 올리지는 못한다. 지침은 행동 패턴을 교정할 뿐, 준수 품질은 사용하는 모델 성능에 따른다.

## 구조

```
.
├── docs/                                  # 원본 지침 (SSOT — 모든 수정은 여기 먼저)
│   ├── agent-thinking-guidelines.md
│   ├── multi-agent-orchestration.md
│   └── agent-system-overview.md           #   구현된 에이전트 동작 흐름 설명
├── cursor/                                # Cursor 버전
│   ├── .cursor/rules/                     #   4개 rule (.mdc)
│   ├── .cursor/skills/                    #   상황별 프로토콜 (analysis / design)
│   ├── .cursor/agents/                    #   orchestrator + reviewer 서브에이전트
│   ├── .cursor/memory/                    #   세션 간 교훈 축적
│   ├── .cursor/state/                     #   루프 상태표 영속화
│   └── README.md                          #   설치·프롬프트 가이드
└── claude/                                # Claude Code 버전
    ├── CLAUDE.md                          #   옵트인 스텁 (호출법·항상적용 전환 안내)
    ├── CLAUDE.always.md                   #   항상 적용 선택 시 병합할 전문
    ├── skills/                            #   상황별 프로토콜 (analysis / design)
    ├── agents/                            #   orchestrator + reviewer 서브에이전트
    ├── .claude/memory/                    #   세션 간 교훈 축적
    ├── .claude/state/                     #   루프 상태표 영속화
    └── README.md                          #   설치·프롬프트 가이드
```

## 버전 선택

| | Cursor 버전 | Claude Code 버전 |
|---|---|---|
| 매핑 | Rules (.mdc) + Skills + Subagent | CLAUDE.md + Skills + Subagent |
| **기본 적용** | **옵트인** (`alwaysApply: false`) — `@docs/…` / `/reviewer` 등으로 호출 | **옵트인** (스텁 CLAUDE.md) — 동일 호출법. 항상 적용 시 `CLAUDE.always.md` 반영 |
| 상황별 적용 | description/globs + Skill (호출·해당 작업 시) | Skill description (해당 작업 시) |
| **검증자 분리** | **가능 — reviewer 서브에이전트 (`.cursor/agents/`)** | **가능 — reviewer 서브에이전트 (`.claude/agents/`)** |
| **루프 계획** | **orchestrator 플래너 서브에이전트 (`.cursor/agents/`)** | **orchestrator 플래너 서브에이전트 (`.claude/agents/`)** |
| **세션 간 기억** | `.cursor/memory/` | `.claude/memory/` |
| **루프 상태표** | `.cursor/state/` | `.claude/state/` |
| 검증 루프 | 작성→검증→수정→재검증 루프 가능 | 작성→검증→수정→재검증 루프 가능 |
| 세부 차이 | 서브에이전트 도구 제한은 `readonly` 필드 | `tools:` 화이트리스트로 도구 세분 지정 |

두 버전 모두 reviewer 서브에이전트로 **검증자 분리**(작성자와 별도 컨텍스트에서 산출물만 판정)를 지원한다. **3-에이전트 루프**(오케스트레이터·Worker·승인자)에서 **실제로 루프를 도는 주체는 메인 에이전트**다: 메인 에이전트가 오케스트레이터 역할을 맡아 Worker를 진행시키고 reviewer 서브에이전트로 검증한다. orchestrator 서브에이전트는 이 루프를 대신 돌리는 게 아니라, **작업 분해·위험 등급·라우팅 계획을 메인 에이전트에 반환하는 플래너**로 쓴다. (서브에이전트가 다른 서브에이전트를 직접 호출하는 것은 플랫폼·버전에 따라 불안정하거나 불가능하므로 의존하지 않는다.)

## 빠른 시작

### 명령어로 설치

```bash
git clone https://github.com/geunsu-son/agent-thinking-guidelines.git

# 공통 — @docs/ 호출용 SSOT (필수)
mkdir -p <your-project>/docs
cp <this-repo>/docs/agent-thinking-guidelines.md <your-project>/docs/
# (선택) 시스템 개요·멀티에이전트 규약도 필요하면 docs/ 전체를 복사

# Cursor
cp -r <this-repo>/cursor/.cursor <your-project>/.cursor

# Claude Code
cp <this-repo>/claude/CLAUDE.md <your-project>/CLAUDE.md
cp <this-repo>/claude/CLAUDE.always.md <your-project>/CLAUDE.always.md
cp -r <this-repo>/claude/skills <your-project>/.claude/skills
cp -r <this-repo>/claude/agents <your-project>/.claude/agents
cp -r <this-repo>/claude/.claude/memory <your-project>/.claude/memory
cp -r <this-repo>/claude/.claude/state <your-project>/.claude/state
```

설치 후 기본은 **옵트인**. 호출법과 "항상 적용할까요?" 안내는 아래 AI 설치 프롬프트의 완료 단계를 따른다.

### AI 채팅으로 설치 (권장)

터미널 명령어 대신, **대상 프로젝트를 연 채팅**에 아래 프롬프트를 붙여넣으면 된다. 에이전트가 repo를 가져와 복사·병합하고, 적용 결과를 보고한다.

**공통 준비**
- 대상 프로젝트 루트를 Cursor / Claude Code로 연다.
- 아래 프롬프트의 `<your-project>`는 현재 워크스페이스 루트로 이해하면 된다.

---

**Cursor** — Agent 채팅에 붙여넣기:

```
이 프로젝트에 Agent Thinking Guidelines를 설치해줘.

소스: https://github.com/geunsu-son/agent-thinking-guidelines
Cursor 버전만 사용해. `cursor/.cursor/` 아래 내용을 이 프로젝트 루트의 `.cursor/`에 반영해.
기본 모드는 옵트인(호출 시에만 적용)이다. core-principles·worker-conduct의 alwaysApply는 false로 둔다.

작업 순서:
1. repo를 임시로 clone한 뒤 `cursor/.cursor/`와 `docs/` 구조를 확인해
2. `docs/agent-thinking-guidelines.md`를 이 프로젝트 `docs/`에 복사 (없으면 디렉터리 생성). @docs/ 호출용 SSOT다
3. 이미 `.cursor/`가 있으면 덮어쓰지 말고 병합해
   - `rules/`, `skills/`, `agents/`, `memory/`, `state/`는 없는 것만 추가
   - 같은 이름의 rule·skill·agent가 있으면 차이를 요약하고 내 확인 후 처리해
4. 복사 대상:
   - rules/ (core-principles, worker-conduct, analysis-protocol, design-protocol) — alwaysApply는 upstream 그대로(기본 false)
   - skills/ (analysis-protocol, design-protocol)
   - agents/ (orchestrator, reviewer)
   - memory/ (README 및 예시)
   - state/ (README 및 loop-status.example — 런타임 loop-status.md는 생성만, 복사 안 함)
5. 파일 반영이 끝나면 설치 요약을 짧게 보고한 뒤, 아래를 **그대로** 안내하고 마지막 질문을 반드시 해:
   - 기본 모드는 호출 시에만 지침을 씀 (토큰 절약)
   - 호출 방법1: @docs/agent-thinking-guidelines.md
   - 호출 방법2: /reviewer … (검증), /orchestrator … (계획)
   - 질문: "항상 적용되도록 적용할까요?"
   - 사용자가 예라고 하면 core-principles.mdc·worker-conduct.mdc의 alwaysApply를 true로 바꾸고 확인 보고
   - 아니오(또는 미응답)면 옵트인 유지
```

---

**Claude Code** — 채팅에 붙여넣기:

```
이 프로젝트에 Agent Thinking Guidelines를 설치해줘.

소스: https://github.com/geunsu-son/agent-thinking-guidelines
Claude Code 버전만 사용해. `claude/` 아래 내용을 이 프로젝트에 반영해.
기본 모드는 옵트인(호출 시에만 적용)이다. CLAUDE.md는 스텁(호출 안내)을 쓰고, 전문은 CLAUDE.always.md에 둔다.

작업 순서:
1. repo를 임시로 clone한 뒤 `claude/`와 `docs/` 구조를 확인해
2. `docs/agent-thinking-guidelines.md`를 이 프로젝트 `docs/`에 복사 (없으면 디렉터리 생성). @docs/ 호출용 SSOT다
3. 파일별 반영:
   - `CLAUDE.md` → 프로젝트 루트에 옵트인 스텁으로 설치. 이미 있으면 덮어쓰지 말고 옵트인 안내·호출법을 병합해 (전문 원칙을 자동으로 항상 로드하지 말 것)
   - `CLAUDE.always.md` → 프로젝트 루트에 복사 (항상 적용 전환용)
   - `skills/` → `.claude/skills/` (없는 skill만 추가, 동일 이름은 차이 요약 후 확인)
   - `agents/` → `.claude/agents/` (orchestrator, reviewer)
   - `.claude/memory/` → `.claude/memory/` (디렉터리·README)
   - `.claude/state/` → `.claude/state/` (디렉터리·README·example)
4. 개인 전역(`~/.claude/`)에는 설치하지 말고, 이 프로젝트에만 적용해
5. 파일 반영이 끝나면 설치 요약을 짧게 보고한 뒤, 아래를 **그대로** 안내하고 마지막 질문을 반드시 해:
   - 기본 모드는 호출 시에만 지침을 씀 (토큰 절약)
   - 호출 방법1: @docs/agent-thinking-guidelines.md
   - 호출 방법2: /reviewer … (검증), /orchestrator … (계획)
   - 질문: "항상 적용되도록 적용할까요?"
   - 사용자가 예라고 하면 CLAUDE.always.md 내용을 CLAUDE.md에 반영(교체 또는 병합)하고 확인 보고
   - 아니오(또는 미응답)면 옵트인 스텁 유지
```

---

**이미 설치했는지 모를 때** — 사용 중인 도구에 맞는 한 줄:

| 도구 | 확인 프롬프트 |
|---|---|
| Cursor | `이 프로젝트에 agent-thinking-guidelines(Cursor 버전)가 적용돼 있는지 확인해줘. docs/agent-thinking-guidelines.md, .cursor/rules, skills, agents, memory, state를 점검하고 빠진 항목만 설치해. 끝나면 호출법 안내 후 "항상 적용되도록 적용할까요?"를 물어.` |
| Claude Code | `이 프로젝트에 agent-thinking-guidelines(Claude Code 버전)가 적용돼 있는지 확인해줘. docs/agent-thinking-guidelines.md, CLAUDE.md, CLAUDE.always.md, .claude/skills, agents, memory, state를 점검하고 빠진 항목만 설치해. 끝나면 호출법 안내 후 "항상 적용되도록 적용할까요?"를 물어.` |

설치 후 작업 지시(프롬프트 템플릿)는 각 버전 README 참조 — [cursor/README.md](cursor/README.md), [claude/README.md](claude/README.md). **3-에이전트 루프** 통합 템플릿도 포함되어 있다.

### 업데이트 (버전 반영)

upstream repo가 갱신되면, 설치 때와 같이 **명령어** 또는 **AI 채팅**으로 반영한다. 원칙은 동일하다: 지침 파일은 upstream 기준으로 맞추되, **프로젝트 전용 내용은 보존**한다.

**보존 (덮어쓰지 않음)**
- `.cursor/memory/`, `.claude/memory/` 안의 **프로젝트 교훈 파일** (upstream에 없는 파일)
- `.cursor/state/loop-status.md`, `.claude/state/loop-status.md` (루프 런타임 상태)
- 팀이 직접 추가한 rule·skill·agent (upstream 목록에 없는 이름)

**갱신 (upstream과 동기화)**
- `core-principles`, `worker-conduct`, `analysis-protocol`, `design-protocol` (rule·skill)
- `orchestrator`, `reviewer` (agent)
- `memory/README.md` (규약만 — 교훈 본문은 유지)
- `state/README.md`, `state/loop-status.example.md` (규약·형식만 — 런타임 `loop-status.md`는 유지)

#### 명령어로 업데이트

```bash
# 최신 upstream 가져오기
git clone https://github.com/geunsu-son/agent-thinking-guidelines.git /tmp/agent-thinking-guidelines
# 이미 clone 해 두었다면: cd /tmp/agent-thinking-guidelines && git pull

# docs SSOT
cp /tmp/agent-thinking-guidelines/docs/agent-thinking-guidelines.md <your-project>/docs/

# Cursor — 파일 단위로 덮어쓰기 (memory/ 교훈 파일은 제외하고 직접 확인)
# alwaysApply를 프로젝트에서 true로 바꿔 둔 경우, 덮어쓴 뒤 다시 true로 맞출지 확인
cp /tmp/agent-thinking-guidelines/cursor/.cursor/rules/*.mdc <your-project>/.cursor/rules/
cp -r /tmp/agent-thinking-guidelines/cursor/.cursor/skills/* <your-project>/.cursor/skills/
cp /tmp/agent-thinking-guidelines/cursor/.cursor/agents/*.md <your-project>/.cursor/agents/
cp /tmp/agent-thinking-guidelines/cursor/.cursor/memory/README.md <your-project>/.cursor/memory/
cp /tmp/agent-thinking-guidelines/cursor/.cursor/state/README.md <your-project>/.cursor/state/
cp /tmp/agent-thinking-guidelines/cursor/.cursor/state/loop-status.example.md <your-project>/.cursor/state/

# Claude Code — CLAUDE.md는 옵트인 스텁 vs 항상적용 여부를 diff로 확인 후 병합
diff <your-project>/CLAUDE.md /tmp/agent-thinking-guidelines/claude/CLAUDE.md
cp /tmp/agent-thinking-guidelines/claude/CLAUDE.always.md <your-project>/CLAUDE.always.md
cp -r /tmp/agent-thinking-guidelines/claude/skills/* <your-project>/.claude/skills/
cp /tmp/agent-thinking-guidelines/claude/agents/*.md <your-project>/.claude/agents/
cp /tmp/agent-thinking-guidelines/claude/.claude/memory/README.md <your-project>/.claude/memory/
cp /tmp/agent-thinking-guidelines/claude/.claude/state/README.md <your-project>/.claude/state/
cp /tmp/agent-thinking-guidelines/claude/.claude/state/loop-status.example.md <your-project>/.claude/state/
```

#### AI 채팅으로 업데이트 (권장)

**Cursor** — Agent 채팅에 붙여넣기:

```
이 프로젝트의 Agent Thinking Guidelines를 최신 upstream으로 업데이트해줘.

소스: https://github.com/geunsu-son/agent-thinking-guidelines (최신 master 기준)
Cursor 버전만 대상으로 해.

작업 순서:
1. repo를 임시로 clone하거나 pull해서 최신 `cursor/.cursor/`와 `docs/`를 확인해
2. 이 프로젝트의 `docs/`, `.cursor/`와 upstream을 파일별로 비교해
3. 변경 요약을 먼저 보여줘 (추가·수정·삭제된 upstream 파일)
4. 반영 규칙:
   - `docs/agent-thinking-guidelines.md` → upstream 기준으로 갱신
   - upstream에 있는 rules/, skills/, agents/ → 내용을 upstream 기준으로 갱신
   - 단, 이 프로젝트에서 alwaysApply를 true로 바꿔 둔 rule은 내용 갱신 후 alwaysApply 설정을 유지할지 확인
   - `.cursor/memory/`의 프로젝트 교훈 파일(upstream에 없는 .md) → 건드리지 않음
   - `memory/README.md`만 upstream 규약으로 갱신
   - `.cursor/state/loop-status.md`(런타임) → 건드리지 않음
   - `state/README.md`, `state/loop-status.example.md`만 upstream 규약으로 갱신
   - 이 프로젝트에만 있는 rule·skill·agent → 유지
   - 같은 이름 파일인데 우리가 커스텀했을 수 있으면 diff를 보여주고 내 확인 후 반영
5. 완료 후: 갱신·보존 파일 목록을 보고하고, 호출법(@docs/…, /reviewer, /orchestrator)을 다시 안내한 뒤 "항상 적용되도록 적용할까요?"를 물어.
   - 예 → core-principles·worker-conduct의 alwaysApply를 true로
   - 아니오 → 옵트인(false) 유지. 사용자가 이미 항상 적용 중이면 그 설정을 유지할지 확인
```

**Claude Code** — 채팅에 붙여넣기:

```
이 프로젝트의 Agent Thinking Guidelines를 최신 upstream으로 업데이트해줘.

소스: https://github.com/geunsu-son/agent-thinking-guidelines (최신 master 기준)
Claude Code 버전만 대상으로 해.

작업 순서:
1. repo를 임시로 clone하거나 pull해서 최신 `claude/`와 `docs/`를 확인해
2. 이 프로젝트의 docs/, CLAUDE.md, CLAUDE.always.md, `.claude/skills`, `agents`, `memory`, `state`와 upstream을 비교해
3. 변경 요약을 먼저 보여줘
4. 반영 규칙:
   - `docs/agent-thinking-guidelines.md` → upstream 기준으로 갱신
   - `skills/`, `agents/` → upstream 기준으로 갱신
   - `CLAUDE.always.md` → upstream 기준으로 갱신
   - `CLAUDE.md` → 현재가 옵트인 스텁이면 스텁을 upstream 기준으로 갱신. 이미 항상 적용(전문)이면 덮어쓰지 말고 CLAUDE.always.md 변경분만 병합할지 확인
   - `.claude/memory/`의 프로젝트 교훈 파일 → 건드리지 않음
   - `memory/README.md`만 upstream 규약으로 갱신
   - `.claude/state/loop-status.md`(런타임) → 건드리지 않음
   - `state/README.md`, `state/loop-status.example.md`만 upstream 규약으로 갱신
   - 이 프로젝트에만 있는 skill·agent → 유지
5. 개인 전역(`~/.claude/`)은 수정하지 말 것
6. 완료 후: 갱신·보존 파일 목록을 보고하고, 호출법(@docs/…, /reviewer, /orchestrator)을 다시 안내한 뒤 "항상 적용되도록 적용할까요?"를 물어.
   - 예 → CLAUDE.always.md를 CLAUDE.md에 반영
   - 아니오 → 옵트인 유지
```

| 도구 | 버전 확인 프롬프트 |
|---|---|
| Cursor | `agent-thinking-guidelines upstream과 이 프로젝트 .cursor/를 비교해줘. 빠지거나 오래된 지침 파일만 골라 업데이트안을 보여줘.` |
| Claude Code | `agent-thinking-guidelines upstream과 이 프로젝트 CLAUDE.md·.claude/를 비교해줘. 빠지거나 오래된 지침만 골라 업데이트안을 보여줘.` |

## 요구 사항

- **Cursor**: 서브에이전트(`.cursor/agents/`) 사용 시 **Cursor 2.4+** ([Subagents 문서](https://cursor.com/docs/agent/subagents))
- **Claude Code**: CLAUDE.md, skills, agents 경로 지원 버전

## SSOT 동기화 체크리스트

`docs/`를 수정할 때 함께 갱신할 대상:

| SSOT 변경 | Cursor | Claude Code |
|---|---|---|
| 핵심 원칙·금지 행동 | `cursor/.cursor/rules/core-principles.mdc` | `claude/CLAUDE.always.md` (+ 항상 적용 시 `CLAUDE.md`) |
| 작업 규율·진행 보고 | `cursor/.cursor/rules/worker-conduct.mdc` | `claude/CLAUDE.always.md` (+ 항상 적용 시 `CLAUDE.md`) |
| 옵트인 안내·호출법 | (rule frontmatter `alwaysApply: false`) | `claude/CLAUDE.md` (스텁) |
| 분석 프로토콜 | `cursor/.cursor/skills/analysis-protocol/SKILL.md` (rule `.mdc`는 트리거 포인터 — 내용 수정 불필요) | `claude/skills/analysis-protocol/SKILL.md` |
| 설계 프로토콜 | `cursor/.cursor/skills/design-protocol/SKILL.md` (rule `.mdc`는 트리거 포인터 — 내용 수정 불필요) | `claude/skills/design-protocol/SKILL.md` |
| 오케스트레이터 | `cursor/.cursor/agents/orchestrator.md` | `claude/agents/orchestrator.md` |
| 승인자 | `cursor/.cursor/agents/reviewer.md` | `claude/agents/reviewer.md` |
| 메모리 규약 | `cursor/.cursor/memory/README.md` | `claude/.claude/memory/README.md` |
| 루프 상태표 | `cursor/.cursor/state/README.md` | `claude/.claude/state/README.md` |

## 확장 (선택 — 앱/하네스 레벨)

장기·비동기 에이전트를 **직접 구현**할 때 참고 (이 repo의 정적 파일만으로는 제공하지 않음):

- **send-to-user tool**: 작업이 끝나기 전에 사용자에게 **원문 그대로** 중간 결과·진행을 전달하는 클라이언트 도구. Fable 5 장기 실행 패턴과 동일. [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) 참조.
- Cursor/Claude Code IDE 사용 시에는 채팅 응답으로 충분한 경우가 많다. 자체 에이전트 API·백그라운드 루프를 만들 때만 도입을 검토한다.

## 운영 원칙

1. **SSOT**: 기준 변경은 `docs/` 원본에 먼저 반영하고 각 버전으로 내려보낸다.
2. **어긴 항목은 예시로 승격**: 에이전트가 반복해서 어기는 규칙은 나쁨/좋음 대비 예시로 추가한다. 규칙 서술보다 예시가 준수율을 가장 많이 올린다. 추가 위치: 원칙은 `docs/` + `core-principles.mdc`/`CLAUDE.always.md`, 상황 한정(분석·설계 등)은 해당 skill.
3. **짧게 유지**: rule/CLAUDE.md가 길수록 준수율이 떨어진다. 추가할 때마다 제거를 함께 고려. 기본은 옵트인을 유지하고, 항상 적용은 사용자가 선택한 프로젝트에만.
4. **사람 샘플 감사**: 검증을 통과한 산출물도 주기적으로 사람이 검수하고, 오판 사례를 체크리스트에 반영한다. 되돌리기 어려운 작업은 항상 사람이 최종 승인한다.

## License

MIT