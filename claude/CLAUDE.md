# Agent Thinking Guidelines (옵트인)

이 프로젝트에 Agent Thinking Guidelines가 설치되어 있다.

**기본값: 호출 시에만 적용.** 토큰·리소스 비용이 크므로 매 턴 자동으로 전문 지침을 따르지 않는다.
사용자가 아래 방법으로 호출하거나, "항상 적용"으로 전환을 요청한 뒤에만 지침을 적용한다.

## 호출 방법

1. **전문 지침 첨부**: `@docs/agent-thinking-guidelines.md` 를 채팅에 첨부한 뒤 작업 지시
2. **검증 서브에이전트**: `/reviewer …` (또는 "reviewer로 검증해줘")
3. **계획 서브에이전트**: `/orchestrator …` (또는 "orchestrator로 작업 분해해줘")

상황별 프로토콜(분석·설계)은 `skills/`에 있으며, 해당 작업을 **명시적으로** 요청하거나 skill을 호출할 때 사용한다.

## 항상 적용으로 전환

사용자가 항상 적용을 원하면:
1. 같은 디렉터리의 `CLAUDE.always.md` 내용을 이 파일(`CLAUDE.md`)에 반영(교체 또는 병합)한다.
2. 반영 후 상단에 "항상 적용 중"임을 한 줄로 남긴다.

전문 SSOT는 항상 `docs/agent-thinking-guidelines.md`다. 기준 변경은 docs → `CLAUDE.always.md` 순으로 고친다.

## 설치 직후 안내 (에이전트용)

설치·업데이트를 마친 에이전트는 사용자에게 다음을 **반드시** 안내한 뒤, 아래 질문을 한다:

```
설치 완료. 기본 모드는 호출 시에만 지침을 씁니다 (토큰 절약).

호출 방법:
1. @docs/agent-thinking-guidelines.md
2. /reviewer …  (검증)
3. /orchestrator …  (계획)

항상 적용되도록 적용할까요?
(예: core 원칙을 매 세션 자동 로드 / 아니오: 지금처럼 호출할 때만)
```
