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
/diagnose
/integration-audit
/investigate
/knowledge-prune
/grill-me
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

**[`/investigate`](./skills/engineering/investigate/SKILL.md)** 는 먼저 재현 가능한 패스/페일 신호를 만들고, 반증 가능한 가설을 세우고, 한 번에 하나씩 검증한다. `.bugs/` 아래에 버그 카드를 유지해서 세션이 바뀌어도 시도한 것과 반증된 가설을 잃지 않는다. 버그를 닫을 때마다 교훈을 패턴 라이브러리에 전파해서 같은 종류의 버그가 다음에 더 빨리 잡힌다.

### #3: 지식 베이스가 쌓일수록 지저분해진다

버그를 계속 추적하다 보면 `.bugs/` 디렉토리가 엉킨다. 같은 버그가 두 개의 카드로 나뉘어 있다. 수개월 전에 시작했다가 방치된 `investigating` 카드가 남아 있다. 패턴 파일에는 이미 반증된 규칙이 섞여 있다.

오래된 지식 베이스는 없는 것보다 나쁘다 — 다음 조사를 잘못된 방향으로 이끈다.

**[`/knowledge-prune`](./skills/engineering/knowledge-prune/SKILL.md)** 는 지식 베이스를 전체 감사한 뒤 변경 계획을 먼저 보고하고, 승인 후에만 적용한다. 중복 레코드를 합치고, 방치된 카드를 아카이브하고, 비슷한 패턴 규칙을 하나로 통합하고, 과거 버그에서 지적된 인접 위험을 팔로업한다.

---

## Skills

### Engineering

| Skill | 설명 |
|-------|------|
| [diagnose](./skills/engineering/diagnose/SKILL.md) | 버그/성능 회귀 진단 루프 — 재현 → 가설 → 계측 → 수정 → 회귀 테스트 |
| [integration-audit](./skills/engineering/integration-audit/SKILL.md) | 구현이 실제로 완성됐는지 감사 — 빈 핸들러, 끊긴 라우트, 반쪽짜리 뮤테이션, 더미 데이터 탐지 |
| [investigate](./skills/engineering/investigate/SKILL.md) | 재현 루프 기반 버그 디버깅 — 가설 추적, 세션 간 버그 카드 및 패턴 라이브러리 유지 |
| [knowledge-prune](./skills/engineering/knowledge-prune/SKILL.md) | .bugs/ 지식 베이스 정리 및 최적화 — 중복 탐지, 패턴 통합, 오래된 레코드 정리 |

### Productivity

| Skill | 설명 |
|-------|------|
| [grill-me](./skills/productivity/grill-me/SKILL.md) | 계획/설계를 집요하게 인터뷰해서 결정 트리의 모든 분기를 해소 |

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
