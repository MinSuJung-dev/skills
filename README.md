# Skills

AI 코딩 에이전트의 반복 실패 패턴을 잡는 스킬 모음.

---

## 설치 (Claude Code)

마켓플레이스를 먼저 추가한다:

```bash
claude plugin marketplace add MinSuJung-dev/skills
```

### 전체 설치

```bash
claude plugin install skills
```

### 개별 설치

필요한 스킬만 골라서 설치:

```bash
claude plugin install investigate
claude plugin install integration-audit
claude plugin install diagnose
claude plugin install knowledge-prune
claude plugin install apply-harness
claude plugin install grill-me
```

설치 후 대화에서 바로 호출:

```
/investigate
/integration-audit
/diagnose
/knowledge-prune
/apply-harness
/grill-me
```

---

## 스킬 목록

### `/investigate` — 버그 두 번 안 고치기

버그를 잡으려 시도했는데 세션이 바뀌고 같은 걸 또 시도하는 패턴을 끊는다.

- `.bugs/` 디렉토리에 **버그 카드**를 만들어 세션이 바뀌어도 시도 내역이 남는다
- 재현 가능한 패스/페일 신호를 먼저 만든 다음 가설을 세운다
- 반증 가능한 가설 3–5개를 한 번에 하나씩 검증한다
- 버그를 닫을 때마다 패턴 라이브러리에 교훈을 남겨 다음번에 더 빠르게 잡는다

**언제 쓰나**: 버그 신고, "여전히 안 돼", "이미 시도해봤어", "또 터졌어"

---

### `/integration-audit` — 껍데기 구현 잡기

화면은 그려졌는데 버튼을 누르면 아무 일도 일어나지 않는 경우를 잡는다. 빈 핸들러, 끊긴 라우트, 하드코딩 더미 데이터를 추적한다.

- UI → 라우팅 → 상태/데이터 → 플랫폼 레이어까지 실행 경로를 끝까지 추적
- 발견 내용을 심각도/신뢰도 포함해 보고하며, **코드는 직접 고치지 않는다**
- 승인 후에만 수정

**언제 쓰나**: PR 전, 머지 전, "동작해야 하는데 왜 안 되지"

---

### `/diagnose` — 체계적인 진단 루프

버그/성능 회귀를 체계적으로 짚는다. `investigate`와 달리 지식 베이스를 유지하지 않으며 단발성 진단에 집중한다.

재현 → 최소화 → 가설 3–5개 → 계측 → 수정 → 회귀 테스트 순서로 진행.

**언제 쓰나**: "진단해줘", "디버그해줘", 처음 마주치는 버그나 성능 문제

---

### `/knowledge-prune` — 지식 베이스 정리

`/investigate`가 쌓은 `.bugs/` 디렉토리를 정리한다. 오래된 카드, 중복 버그, 반증된 패턴 규칙이 쌓이면 다음 조사를 잘못된 방향으로 이끈다.

- Index 무결성, 중복, 방치된 카드, 패턴 통합, 미팔로업 위험 5가지를 감사
- **변경 계획을 먼저 보고하고 승인 후 적용**
- 삭제 없이 `.bugs/archive/`로 이동

**언제 쓰나**: `investigate`를 오래 써서 `.bugs/`가 엉킨 느낌이 들 때

---

### `/grill-me` — 계획/설계 검증 인터뷰

계획이나 설계를 말하면 결정 트리의 모든 분기를 해소할 때까지 질문 하나씩 파고든다. 각 질문마다 추천 답변도 제시한다. 코드베이스로 답할 수 있는 질문은 직접 탐색한다.

**언제 쓰나**: 설계를 검증하고 싶을 때, "날카롭게 물어봐줘", "grill me"

---

### `/apply-harness` — 새 프로젝트에 에이전트 체계 구축

검증된 오케스트레이터+전문가 에이전트 구조를 새 프로젝트에 맞게 적용한다.

1. 기술 스택과 모듈 구조 분석
2. 반복 작업 유형 식별 (기능 개발, 버그 수정, 정리 등)
3. 옵션 선택 (아래 참고)
4. `CLAUDE.md`, `.claude/agents/`, `.claude/skills/` 자동 생성

**옵션 A — 지식 베이스 관리** (기본값: 활성화)

bugfix 에이전트가 `investigate` 프로토콜을 따른다. 버그 작업마다 `.bugs/` 에 카드가 생성되고, 수정 후 패턴 라이브러리에 교훈이 누적된다. 10개 이상 쌓이면 `/knowledge-prune` 실행을 안내한다.

비활성화하면 표준 bugfix-pipeline 사용.

**옵션 B — 구현 완성도 감사** (기본값: 활성화)

feature 에이전트가 작업 완료 선언 전 `integration-audit` 프로토콜을 실행한다. 빈 핸들러, 끊긴 라우트, 더미 데이터를 자동으로 탐지하고 보고한다.

비활성화하면 표준 feature-pipeline 사용.

**언제 쓰나**: 새 프로젝트에 에이전트 자동화 체계를 처음 세울 때

---

## Cursor / Windsurf / 기타 AI

원하는 스킬의 `SKILL.md` 내용을 `.cursor/rules`, `.windsurfrules`, 또는 시스템 프롬프트에 붙여넣는다.

```
skills/engineering/investigate/SKILL.md
skills/engineering/integration-audit/SKILL.md
skills/engineering/diagnose/SKILL.md
skills/engineering/knowledge-prune/SKILL.md
skills/productivity/grill-me/SKILL.md
skills/engineering/apply-harness/SKILL.md
```

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

`description`은 에이전트가 이 스킬을 자동으로 쓸지 결정하는 데 사용된다.
