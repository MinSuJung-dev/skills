# Skills

AI-agnostic coding agent skills for serious engineering work.
Each skill is a structured instruction set that any AI agent can follow — Claude Code, Codex, Cursor, or anything that reads markdown.

---

## Skills

### Engineering

| Skill | Trigger |
|-------|---------|
| [integration-audit](./skills/engineering/integration-audit/SKILL.md) | "audit this", "verify before PR", "should work but doesn't" |
| [investigate](./skills/engineering/investigate/SKILL.md) | "debug this", "it's broken again", "same error", performance regression |

---

## Installation

### Claude Code (plugin)

```bash
claude plugin install github:MinSuJung-dev/skills
```

설치 후 대화에서 스킬 이름으로 호출:

```
/integration-audit
/investigate
```

### Claude Code (로컬 개발용)

```bash
claude plugin install /path/to/skills
```

### Cursor / Windsurf (Rules)

1. 원하는 스킬의 `SKILL.md` 파일을 연다
2. 전체 내용을 복사한다
3. `.cursor/rules` 또는 `.windsurfrules`에 붙여넣는다

### Codex / 기타 AI

시스템 프롬프트 또는 대화 시작 시 `SKILL.md` 내용을 직접 첨부한다.

---

## 스킬 설명

### `integration-audit`

> 최근 구현한 기능이 실제로 완성됐는지 감사한다.

"동작하는 것처럼 보이지만 실제로는 안 되는" 상태를 찾아낸다.

**찾아내는 것:**
- `() => {}` 빈 핸들러, `console.log`만 하는 핸들러
- TODO / FIXME / `UnimplementedError`로 막힌 스텁
- 라우터에 등록됐지만 어디서도 진입할 수 없는 화면
- API 호출 없이 하드코딩된 더미 데이터
- UI 피드백 없는 반쪽짜리 뮤테이션
- dispose 없는 StreamSubscription (lifecycle leak)

**동작 방식:**
1. 범위 확정 (PR, 커밋 범위, 디렉토리)
2. UI / 라우팅 / 상태 / 플랫폼 레이어 인벤토리 추출
3. 런타임 등록 코드 선별 (DI, codegen → 신뢰도 낮춤)
4. 각 요소의 실행 경로 끝까지 추적
5. severity + confidence 기준으로 리포트 출력
6. 명시적 승인 없이는 코드 수정 금지

**지원 스택:** Flutter / React / Vue / Next.js / Backend API

---

### `investigate`

> 재현 가능한 피드백 루프를 먼저 만들고, 그 위에서 디버깅한다.

"같은 버그를 같은 방식으로 두 번 고치는" AI 실패 패턴을 방지한다.
`.bugs/` 아래 버그 카드 파일을 유지해 세션이 바뀌어도 이미 시도한 것과 반증된 가설을 잃지 않는다.

**언제 쓰나:**
- 버그 리포트, "디버그해줘", "왜 안 돼"
- "또 같은 에러", "이미 시도해봤어", "고쳤는데 다시 터짐"
- 성능 회귀

**동작 방식:**
1. `.bugs/<slug>.md` 버그 카드 생성 또는 재개
2. 재현 가능한 패스/페일 신호 구축 (테스트, curl, 헤드리스 브라우저 등)
3. 반증 가능한 가설 3–5개 생성 후 사용자에게 순위 공유
4. 한 번에 하나씩 계측 → 결과를 카드에 기록
5. 수정 후 회귀 테스트
6. 디버그 로그 제거, 카드에 원인 기록

---

## 기여

스킬은 `skills/<category>/<skill-name>/SKILL.md` 구조를 따른다.

```markdown
---
name: skill-name
description: AI가 언제 이 스킬을 쓸지 판단하는 트리거 조건 (간결하게)
---

# 스킬 내용...
```

`description`은 AI 에이전트가 자동으로 트리거 여부를 판단하는 데 쓰인다. 짧고 구체적인 트리거 조건으로만 작성한다.
