---
name: agent-thinking-guidelines
description: 에이전트 사고·행동 지침 적용. 설계, 대규모 분석, 중요 산출물, 되돌리기 어려운 변경 작업 시 사용자가 명시 호출할 때 사용한다. "지침 적용", "thinking guidelines", "에이전트 지침" 등으로 요청받을 때도 사용한다.
disable-model-invocation: true
---

# 에이전트 사고·행동 지침

이 스킬이 호출되면 `docs/agent-thinking-guidelines.md` **1~7장**(에이전트용 기준)을 읽고 그 기준으로 작업한다.

## 적용 순서

1. `docs/agent-thinking-guidelines.md`를 Read 도구로 읽는다 (1~7장. 0장은 사용자 가이드, 8장은 메모리 규약).
2. 핵심 원칙·작업 규율을 이번 작업 전반에 적용한다.
3. 상황에 맞으면 추가 프로토콜을 따른다:
   - 데이터 분석·집계·대량 처리 → `analysis-protocol` 스킬
   - 설계·구조 변경·스키마·기획 문서 → `design-protocol` 스킬
4. 중요 산출물 완료 후 검증이 필요하면 `reviewer` 서브에이전트로 검증한다.

## SSOT

- 전문 SSOT: `docs/agent-thinking-guidelines.md`
- 내용이 갈라지면 `docs/`가 우선한다. 이 스킬은 **호출 진입점**이며 전문을 복제하지 않는다.

## 제출 시

산출물과 함께 가정 목록·미해결 사항을 자기신고한다 (`worker-conduct` 규율).
