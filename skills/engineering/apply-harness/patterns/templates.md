# 산출물 템플릿

## CLAUDE.md (프로젝트 루트)

```
# {프로젝트명} — 프로젝트 지도
## 기술 스택
## 모듈 구조
## 절대 규칙 (위반 시 QA NEEDS_FIX)
## 핵심 파일 위치 (표)
## 스킬 목록 (표)
## 오케스트레이션 원칙
```

옵션 A(지식 베이스)가 yes면 다음 섹션을 추가한다:

```
## 지식 베이스
- 모든 버그 작업은 .bugs/INDEX.md 조회로 시작한다
- 버그 카드 위치: .bugs/bugs/BUG-NNN.md
- 패턴 라이브러리: .bugs/patterns/<category>.md
- 카드 10개 이상 누적 시 /knowledge-prune 실행
```

옵션 C(DDD)가 yes면 다음 섹션을 추가한다:

```
## 도메인 (ddd 스킬)
- 도메인 산출물: docs/domain/ (glossary.md, context-map.md, stories/, contexts/, reviews/)
- 클래스·메서드·이벤트 이름은 docs/domain/glossary.md 코드명을 따른다
- 문서·코드에 없는 비즈니스 규칙은 추측하지 않는다 → 질문 또는 `가정:` 명시
- 모듈형 모놀리스 기본, 컨텍스트 = 모듈. MSA는 ADR 근거가 있을 때만
- 도메인 모델링 요청 → /ddd discover, 설계 → /ddd design, DDD 리뷰 → /ddd review
```

## .claude/agents/{name}.md

```yaml
---
name: {name}
description: {트리거 조건 포함한 역할 설명}
model: opus
---
# {Name}
## 역할
## 작업 원칙
## 입력/출력
```

### 옵션 A(지식 베이스)가 yes면 — bugfix 에이전트에 추가

```markdown
## 지식 베이스 프로토콜

모든 버그 작업은 아래 순서를 따른다. 건너뛰지 않는다.

1. **조회** — `.bugs/INDEX.md`에서 동일/유사 버그 확인. 기존 카드가 있으면 읽고 시작.
2. **카드 생성** — 없으면 다음 BUG-NNN id 부여, `.bugs/bugs/BUG-NNN-{slug}.md` 생성, INDEX.md에 행 추가.
3. **재현 신호 구축** — 패스/페일을 판별하는 실행 가능한 신호를 먼저 만든다.
4. **가설 수립** — 반증 가능한 가설 3–5개를 세우고 한 번에 하나씩 검증한다.
5. **수정 + 회귀 테스트** — 범위 밖 파일 수정은 사용자 승인 후.
6. **카드 종료** — 루트 원인, 수정 내용, 인접 위험 기록. `.bugs/patterns/<category>.md`에 예방 규칙 추가.
```

### 옵션 B(구현 완성도 감사)가 yes면 — feature 에이전트에 추가

```markdown
## 완성도 감사 프로토콜

기능 구현 완료를 선언하기 전에 반드시 실행한다.

1. UI 레이어 — onClick/onTap/onPressed 핸들러가 실제로 연결되어 있는가
2. 라우팅 레이어 — 새 라우트가 등록됐고 진입점이 존재하는가
3. 상태/데이터 레이어 — API 연결이 실제로 이루어졌는가, 더미 데이터가 남아 있지 않은가
4. 발견 사항을 심각도/신뢰도 포함해 보고하고, 승인 후 수정한다
```

### 옵션 C(DDD)가 yes면 — architect / dev / qa 에이전트에 추가

architect:

```markdown
## DDD 설계 프로토콜 (ddd 스킬 design 모드)

1. `docs/domain/glossary.md`, `context-map.md`를 먼저 읽는다.
2. 기능이 단순 CRUD/generic인지 판정하고 근거를 spec에 적는다. CRUD면 3–4를 건너뛴다.
3. 아니면 `ddd` 스킬(Skill 도구로 호출)의 Phase 3–6을 수행한다. 서브에이전트는 사용자와 대화할 수 없으므로:
   - 스토리·규칙이 없으면 → "discover 필요"로 반환 (인터뷰는 메인이 한다)
   - 경계·core 분류·연동 방식·to-be 결정 등 사용자 결정이 필요하면 → 추천안과 함께 "결정 필요"로 반환
4. 설계 검증 게이트(`docs/domain/reviews/NNN-*-design.md`)가 `통과`일 때만 `_workspace/01_architect_spec.md`를 쓰고, spec에 해당 리뷰 파일·컨텍스트·애그리거트·유스케이스를 링크한다.
5. spec의 이름은 전부 glossary 코드명. glossary에 없는 개념이 필요하면 spec을 쓰지 않고 질문으로 반환한다.
```

dev:

```markdown
## DDD 구현 규칙 (ddd 스킬 implement 모드)

- spec이 링크한 유스케이스의 규칙을 Domain Rule 테스트(Given-When-Then)로 먼저 쓴다.
- 계층 분리: Domain(규칙) / Application(유스케이스 조율) / Infrastructure(DB·외부). 도메인에 프레임워크 의존 추가 금지 (기존 컨벤션이 다르면 따른다).
- `create/update/delete/find` 모양 서비스 금지, 명령은 도메인 동사.
- spec에 없는 비즈니스 조건을 추가하지 않는다. 필요하면 메인에 되묻는다.
```

qa:

```markdown
## DDD 리뷰 (ddd 스킬 review 모드)

- 기존 QA 체크에 더해 ddd 스킬 Phase 8 체크를 수행하고 `docs/domain/reviews/NNN-*-review.md`에 저장한다.
- HIGH 위반 또는 "추측된 규칙"이 있으면 NEEDS_FIX.
- 코드는 수정하지 않는다. dev가 작성한 맥락을 이어받지 않고 spec·docs/domain·diff만 보고 판단한다.
```

## .claude/skills/{name}/SKILL.md

파일명은 반드시 대문자 `SKILL.md`.

```yaml
---
name: {name}
description: {자연어 트리거 조건 명시}
---
# {Name}
## 실행 모드
## Phase 1: ...
## Phase N: ...
```
