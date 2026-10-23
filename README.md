# My AI Learning

Claude Code를 개인 프로그래밍 선생님으로 쓰는 장기 학습 저장소.
특정 기술에 묶이지 않으며, 새 주제는 언제든 대화로 시작할 수 있다.

## 구조

| 경로 | 역할 |
|---|---|
| `CLAUDE.md` | 공통 운영 지침. 매 세션 자동 로드되며, `learning/PROGRESS.md`를 import한다 |
| `.claude/agents/tutor.md` | 깊이 있는 개념 설명, 내부 동작 원리, 오해 교정 |
| `.claude/agents/researcher.md` | 버전, 최신 동작, 주장 검증 (웹 검색은 가능할 때만) |
| `.claude/agents/learning-coach.md` | 선수 지식, 학습 공백, 개념 연결, 다음 단계 |
| `.claude/agents/examiner.md` | 이해도 확인 문제 출제와 채점 |
| `learning/PROGRESS.md` | 전체 현황 대시보드 (집중 주제, 주제 목록, 우선순위) |
| `learning/topics/_TEMPLATE.md` | 주제 파일 템플릿 |
| `learning/topics/<slug>.md` | 주제별 학습 기록 (공부를 시작하면 자동 생성) |

## 사용법

별도 설정 없이 대화로 시작한다.

- 새 주제: "Python asyncio 공부하고 싶어" → 기록 확인 → `learning/topics/python-asyncio.md` 생성 → 학습 시작
- 이어서 공부: "asyncio 이어서 하자" → 해당 파일만 읽고 지난번 위치부터 이어 감
- 무엇을 공부할지 모를 때: "다음에 뭘 공부하면 좋을까?"
- 이해도 확인: "내가 이해했는지 확인해줘" → examiner가 문제를 내고, 한 문제씩 답한다
- 에이전트 직접 지정: "researcher로 확인해줘" 또는 `@agent-researcher ...`
- 기록 저장: "지금까지 내용 기록해줘"

## 기록 원칙

- 실제 대화에서 일어난 일만 기록한다.
- "학습함"은 이해의 증거가 아니다. "확인됨"은 원리를 설명하거나 적용해 맞힌 근거가 있을 때만 기록한다.
- 주제 파일 생성과 사실 기록(다룬 개념, 공백, 질문)은 자동으로 저장한다. 삭제, 완료 처리, 우선순위 변경은 먼저 묻는다.

## 기록 보존 (중요)

기록은 git에 커밋해야 남는다. Claude Code on the web(클라우드 세션)은 매번 저장소를 새로 clone하므로,
커밋/푸시하지 않은 기록은 세션이 끝나면 사라진다. 새 세션은 보통 기본 브랜치(`main`)에서 시작하므로,
다음 세션에서 기록을 보려면 변경 사항이 `main`에 반영되어 있어야 한다.
