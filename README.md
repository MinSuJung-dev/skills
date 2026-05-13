# Skills

AI coding agent skills built to fix specific failure modes — not to own your workflow.

Small, composable, and stack-agnostic. Works with Claude Code, Codex, Cursor, or any agent that reads markdown.

---

## Quickstart

### Claude Code

마켓플레이스를 먼저 추가한 후 설치:

```bash
claude plugin marketplace add MinSuJung-dev/skills
claude plugin install skills
```

설치 후 대화에서 스킬 이름으로 호출:

```
/integration-audit
/investigate
```

### Cursor / Windsurf

원하는 스킬의 `SKILL.md` 내용을 `.cursor/rules` 또는 `.windsurfrules`에 붙여넣는다.

### Codex / 기타 AI

시스템 프롬프트 또는 대화 시작 시 `SKILL.md` 내용을 직접 첨부한다.

---

## 이 스킬들이 존재하는 이유

AI 에이전트는 코드를 빠르게 만든다. 하지만 두 가지 실패 패턴이 반복된다.

### #1: 만들었는데 동작하지 않는다

AI가 화면을 그리고, 컴포넌트를 만들고, 파일을 생성한다. 그런데 버튼을 누르면 아무 일도 일어나지 않는다.

핸들러는 비어 있다. 라우트는 등록됐지만 어디서도 진입할 수 없다. API 연결 없이 하드코딩된 더미 데이터가 그럴싸하게 채워져 있다. 구현이 된 것처럼 보이지만 실제로는 껍데기다.

에이전트는 코드를 쓰는 것과 동작하는 것을 구분하지 못한다. PR 전에, 머지 전에, 혹은 "왜 안 되지?" 싶을 때 — **[`/integration-audit`](./skills/engineering/integration-audit/SKILL.md)** 가 실행 경로를 끝까지 추적해서 끊긴 지점을 찾아낸다.

### #2: 같은 버그를 두 번 고친다

에이전트가 버그를 잡으려 한다. 몇 가지 시도를 한다. 잘 안 된다. 세션이 바뀌면 이전에 시도한 것을 잊어버리고 같은 방법을 다시 시도한다.

피드백 루프가 없으면 디버깅이 아니라 추측이다. 재현 신호도 없이 코드를 바꾸는 건 소음이다.

**[`/investigate`](./skills/engineering/investigate/SKILL.md)** 는 먼저 재현 가능한 패스/페일 신호를 만들고, 반증 가능한 가설을 세우고, 한 번에 하나씩 검증한다. `.bugs/` 아래에 버그 카드를 유지해서 세션이 바뀌어도 시도한 것과 반증된 가설을 잃지 않는다.

---

## Skills

### Engineering

| Skill | 설명 |
|-------|------|
| [integration-audit](./skills/engineering/integration-audit/SKILL.md) | 구현이 실제로 완성됐는지 감사 — 빈 핸들러, 끊긴 라우트, 반쪽짜리 뮤테이션, 더미 데이터 탐지 |
| [investigate](./skills/engineering/investigate/SKILL.md) | 재현 루프 기반 버그 디버깅 — 가설 추적, 세션 간 버그 카드 유지 |

---

## 기여

스킬은 `skills/<category>/<skill-name>/SKILL.md` 구조를 따른다.

```markdown
---
name: skill-name
description: AI가 트리거 여부를 판단하는 조건. 짧고 구체적으로.
---

# 스킬 내용
```

`description`은 에이전트가 자동으로 이 스킬을 쓸지 결정하는 데 사용된다.
