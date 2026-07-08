# 루프 상태표

3-에이전트 루프(orchestrator + Worker + reviewer) 실행 시 반려 횟수·작업 상태를 파일로 영속화한다. SSOT는 `docs/multi-agent-orchestration.md` 3.4.

## 파일

- **런타임**: `loop-status.md` — 메인 에이전트가 루프마다 생성·갱신 (gitignore 대상)
- **형식 참고**: `loop-status.example.md`

## 규칙

1. 설치 시 이 디렉터리를 프로젝트 `.cursor/state/`에 복사한다 (`README.md`, `loop-status.example.md` — 런타임 `loop-status.md`는 루프 시작 시 생성).
2. 루프 시작 시 `loop-status.md`를 생성한다 (없으면 example을 참고).
3. **매 판정**(통과/반려/에스컬레이션) 후 즉시 갱신한다.
4. 세션 재개·컨텍스트 압축 후에는 컨텍스트보다 이 파일을 먼저 읽는다.
5. 반려 3회·동일 사유 2연속 반려 에스컬레이션 판단은 이 파일의 반려횟수를 기준으로 한다.
