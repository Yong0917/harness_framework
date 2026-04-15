# 프로젝트: Harness Framework

Claude Code로 복잡한 기능을 여러 step으로 나누어 자동 실행하는 오케스트레이션 프레임워크.
`execute.py`가 각 step을 순차 실행하고, 실패 시 자동 재시도하며, 상태를 JSON으로 추적한다.

## 기술 스택
- Python 3 (표준 라이브러리만 사용, 외부 의존성 없음)
- pytest (테스트)
- Claude Code CLI (`claude -p`)

## 아키텍처 규칙
- CRITICAL: `execute.py`는 단일 파일로 유지한다. 헬퍼 모듈로 분리하지 마라.
- CRITICAL: `phases/` 하위 JSON 스키마(index.json, step output)를 임의로 변경하지 마라. execute.py와 스키마가 항상 동기화되어야 한다.
- `scripts/` 디렉토리에만 실행 스크립트를 둔다.
- `phases/{task-name}/` 디렉토리는 harness 실행 산출물이므로 소스로 취급하지 않는다.

## 개발 프로세스
- CRITICAL: 새 기능 구현 시 반드시 `test_execute.py`에 테스트를 먼저 작성하고, 테스트가 통과하는 구현을 작성할 것 (TDD)
- 커밋 메시지는 conventional commits 형식을 따를 것 (feat:, fix:, docs:, refactor:)

## 명령어
python3 scripts/execute.py {phase}         # phase 실행
python3 scripts/execute.py {phase} --push  # 실행 후 push
python3 -m pytest scripts/test_execute.py  # 테스트
