# 아키텍처

## 디렉토리 구조

```
harness_framework/
├── scripts/
│   ├── execute.py          # 단일 파일 실행 엔진 (분리 금지)
│   └── test_execute.py     # execute.py 테스트
├── phases/                 # 실행 산출물 (소스 아님, git에서 output.json 제외)
│   ├── index.json          # 전체 phase 현황 (top-level)
│   └── {task-name}/
│       ├── index.json      # task 상세 + step 상태
│       ├── step{N}.md      # step 지시 파일
│       └── step{N}-output.json  # Claude 실행 결과 (gitignore)
├── docs/                   # 가드레일 문서 (_load_guardrails가 자동 주입)
│   ├── ARCHITECTURE.md     # 이 파일
│   └── ADR.md              # 기술 결정 기록
└── .claude/
    ├── settings.json       # hooks (Stop: pytest, PreToolUse: 위험 명령 차단)
    └── skills/
        ├── harness/SKILL.md   # /harness 스킬 워크플로우
        └── review/SKILL.md    # /review 스킬 체크리스트
```

## 설계 패턴

- **단일 파일 원칙**: `execute.py`는 표준 라이브러리만 사용, 외부 헬퍼 모듈 없음. `python3 scripts/execute.py`로 즉시 실행 가능.
- **상태 기반 실행**: `phases/{task-name}/index.json`으로 step 상태 추적. 중단 후 재시작 가능.
- **가드레일 주입**: `CLAUDE.md` + `docs/*.md` 내용을 모든 step 프롬프트에 포함해 아키텍처 일관성을 강제.
- **2단계 커밋**: 코드 변경(`feat`)과 메타데이터 변경(`chore`)을 분리 커밋.

## 데이터 흐름

```
execute.py 실행
  → index.json 읽기 (pending step 탐색)
  → CLAUDE.md + docs/*.md 로딩 (가드레일)
  → 이전 completed step summary 누적 (컨텍스트)
  → preamble + step{N}.md 조합 → claude -p 호출
  → index.json에서 status 확인 (completed / error / blocked)
  → git commit (feat + chore)
  → 다음 pending step으로 반복
```
