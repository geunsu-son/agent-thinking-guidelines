# Agent Thinking Guidelines

AI 에이전트가 설계·분석 작업을 정확하고 깊이 있게 수행하도록 만드는 지침 세트.
동일한 원본 지침(`docs/`)을 **Cursor 버전**과 **Claude Code 버전**으로 각각 매핑했다.

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
│   ├── 에이전트_사고행동_지침.md
│   └── 멀티에이전트_오케스트레이션_지침.md
├── cursor/                                # Cursor 버전
│   ├── .cursor/rules/                     #   4개 rule (.mdc)
│   ├── .cursor/skills/                    #   상황별 프로토콜 (analysis / design)
│   ├── .cursor/agents/                    #   orchestrator + reviewer 서브에이전트
│   ├── .cursor/memory/                    #   세션 간 교훈 축적
│   └── README.md                          #   설치·프롬프트 가이드
└── claude/                                # Claude Code 버전
    ├── CLAUDE.md                          #   항상 로드되는 핵심 원칙
    ├── skills/                            #   상황별 프로토콜 (analysis / design)
    ├── agents/                            #   orchestrator + reviewer 서브에이전트
    ├── .claude/memory/                    #   세션 간 교훈 축적
    └── README.md                          #   설치·프롬프트 가이드
```

## 버전 선택

| | Cursor 버전 | Claude Code 버전 |
|---|---|---|
| 매핑 | Rules (.mdc) + Skills + Subagent | CLAUDE.md + Skills + Subagent |
| 상황별 적용 | globs + description + Skill 트리거 | Skill description 자동 트리거 |
| **검증자 분리** | **가능 — reviewer 서브에이전트 (`.cursor/agents/`)** | **가능 — reviewer 서브에이전트 (`.claude/agents/`)** |
| **루프 관리** | **가능 — orchestrator 서브에이전트 (`.cursor/agents/`)** | **가능 — orchestrator 서브에이전트 (`.claude/agents/`)** |
| **세션 간 기억** | `.cursor/memory/` | `.claude/memory/` |
| 검증 루프 | 작성→검증→수정→재검증 루프 가능 | 작성→검증→수정→재검증 루프 가능 |
| 세부 차이 | 서브에이전트 도구 제한은 `readonly` 필드 | `tools:` 화이트리스트로 도구 세분 지정 |

두 버전 모두 서브에이전트(orchestrator, reviewer)를 지원하므로 **3-에이전트 루프**(오케스트레이터·Worker·승인자 분리)와 **검증자 분리**(작성자와 별도 컨텍스트에서 산출물만 판정)가 가능하다. 남는 차이는 매핑 형식과 서브에이전트 세부 설정 필드 정도이며, 어느 쪽을 쓰든 검증 루프의 효과는 동일하다.

## 빠른 시작

```bash
git clone https://github.com/geunsu-son/agent-thinking-guidelines.git

# Cursor
cp -r <this-repo>/cursor/.cursor <your-project>/.cursor

# Claude Code
cp <this-repo>/claude/CLAUDE.md <your-project>/CLAUDE.md
cp -r <this-repo>/claude/skills <your-project>/.claude/skills
cp -r <this-repo>/claude/agents <your-project>/.claude/agents
cp -r <this-repo>/claude/.claude/memory <your-project>/.claude/memory
```

지시하는 법(프롬프트 템플릿)은 각 버전의 README 참조. **3-에이전트 루프** 통합 템플릿도 포함되어 있다.

## 요구 사항

- **Cursor**: 서브에이전트(`.cursor/agents/`) 사용 시 **Cursor 2.4+** ([Subagents 문서](https://cursor.com/docs/agent/subagents))
- **Claude Code**: CLAUDE.md, skills, agents 경로 지원 버전

## SSOT 동기화 체크리스트

`docs/`를 수정할 때 함께 갱신할 대상:

| SSOT 변경 | Cursor | Claude Code |
|---|---|---|
| 핵심 원칙·금지 행동 | `cursor/.cursor/rules/core-principles.mdc` | `claude/CLAUDE.md` |
| 작업 규율·진행 보고 | `cursor/.cursor/rules/worker-conduct.mdc` | `claude/CLAUDE.md` |
| 분석 프로토콜 | `cursor/.cursor/rules/analysis-protocol.mdc`, `cursor/.cursor/skills/analysis-protocol/SKILL.md` | `claude/skills/analysis-protocol/SKILL.md` |
| 설계 프로토콜 | `cursor/.cursor/rules/design-protocol.mdc`, `cursor/.cursor/skills/design-protocol/SKILL.md` | `claude/skills/design-protocol/SKILL.md` |
| 오케스트레이터 | `cursor/.cursor/agents/orchestrator.md` | `claude/agents/orchestrator.md` |
| 승인자 | `cursor/.cursor/agents/reviewer.md` | `claude/agents/reviewer.md` |
| 메모리 규약 | `cursor/.cursor/memory/README.md` | `claude/.claude/memory/README.md` |

## 확장 (선택 — 앱/하네스 레벨)

장기·비동기 에이전트를 **직접 구현**할 때 참고 (이 repo의 정적 파일만으로는 제공하지 않음):

- **send-to-user tool**: 작업이 끝나기 전에 사용자에게 **원문 그대로** 중간 결과·진행을 전달하는 클라이언트 도구. Fable 5 장기 실행 패턴과 동일. [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) 참조.
- Cursor/Claude Code IDE 사용 시에는 채팅 응답으로 충분한 경우가 많다. 자체 에이전트 API·백그라운드 루프를 만들 때만 도입을 검토한다.

## 운영 원칙

1. **SSOT**: 기준 변경은 `docs/` 원본에 먼저 반영하고 각 버전으로 내려보낸다.
2. **어긴 항목은 예시로 승격**: 에이전트가 반복해서 어기는 규칙은 나쁨/좋음 대비 예시로 추가한다. 규칙 서술보다 예시가 준수율을 가장 많이 올린다.
3. **짧게 유지**: rule/CLAUDE.md가 길수록 준수율이 떨어진다. 추가할 때마다 제거를 함께 고려.
4. **사람 샘플 감사**: 검증을 통과한 산출물도 주기적으로 사람이 검수하고, 오판 사례를 체크리스트에 반영한다. 되돌리기 어려운 작업은 항상 사람이 최종 승인한다.
