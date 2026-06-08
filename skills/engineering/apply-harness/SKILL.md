---
name: apply-harness
description: 검증된 하네스 패턴을 새 프로젝트에 적용한다. "하네스 적용해줘", "에이전트 팀 구성해줘", "이 프로젝트에도 같은 시스템 적용해줘", "자동화 체계 만들어줘" 등 신규 프로젝트에 오케스트레이터+전문가 에이전트 구조를 구축할 때 사용.
---

# Apply Harness

검증된 하네스 패턴을 이 프로젝트에 맞게 적용한다.

## Phase 1: 패턴 파일 로드

```bash
find ~/.claude/plugins/cache/skills -path "*/apply-harness/patterns/*.md" 2>/dev/null | sort
```

위 명령으로 패턴 파일 경로를 찾아 전부 Read한다. 없으면 로컬 개발 경로 `~/project/skills/skills/engineering/apply-harness/patterns/` 에서 찾는다.

See [patterns/orchestration.md](patterns/orchestration.md) — 오케스트레이션 원칙
See [patterns/feature-pipeline.md](patterns/feature-pipeline.md) — 기능 개발 파이프라인
See [patterns/bugfix-pipeline.md](patterns/bugfix-pipeline.md) — 버그 수정 파이프라인
See [patterns/cleanup-pipeline.md](patterns/cleanup-pipeline.md) — 코드 정리 파이프라인
See [patterns/escape-hatch.md](patterns/escape-hatch.md) — 직접 수정 vs 위임 규칙
See [patterns/examples/flutter-app.md](patterns/examples/flutter-app.md) — Flutter 앱 예시
See [patterns/examples/python-backend.md](patterns/examples/python-backend.md) — Python 백엔드 예시

## Phase 2: 프로젝트 분석

다음 순서로 탐색한다:

1. **기술 스택 감지**
   - `pubspec.yaml` → Flutter/Dart
   - `package.json` → Node.js / React / Next.js
   - `requirements.txt` / `pyproject.toml` → Python
   - `build.gradle` / `*.xcodeproj` → Android / iOS 네이티브
   - `go.mod` → Go

2. **모듈 구조 파악** — `find . -maxdepth 4 -name "*.dart" -o -name "*.py" -o -name "*.ts" | head -60`

3. **기존 작업 유형 식별** — 어떤 반복 작업이 자동화 대상인가:
   - 신규 기능/화면 개발
   - 버그 수정
   - 코드 정리 / 품질 관리
   - API/모델 생성
   - 테스트 / 검증

4. **기존 에이전트/스킬 확인** — `.claude/agents/`, `.claude/skills/` 존재 여부 확인

## Phase 3: 옵션 선택

프로젝트 분석 결과를 요약해 보여준 뒤, 사용자에게 다음 옵션을 묻는다. **각 항목을 yes/no로 선택받는다.**

---

**[옵션 A] 지식 베이스 관리** (기본값: yes)

버그 수정 에이전트가 `investigate` 스킬 프로토콜을 따른다.
- 모든 버그 작업 시작 시 `.bugs/INDEX.md`를 먼저 조회
- 버그 카드(`.bugs/bugs/BUG-NNN.md`) 생성 및 세션 간 유지
- 버그 종료 시 패턴 라이브러리(`.bugs/patterns/`) 자동 업데이트
- `.bugs/` 카드가 10개 이상 누적되면 `/knowledge-prune` 실행 안내

yes → Phase 4에서 bugfix 에이전트에 investigate 워크플로우 wiring, CLAUDE.md에 지식 베이스 섹션 추가
no → 표준 bugfix-pipeline 사용

---

**[옵션 B] 구현 완성도 자동 감사** (기본값: yes)

기능 개발 에이전트가 작업 완료 전 `integration-audit` 프로토콜을 실행한다.
- 빈 핸들러, 끊긴 라우트, 더미 데이터 자동 탐지
- 발견 시 사용자에게 보고 후 승인받아 수정

yes → Phase 4에서 feature 에이전트에 integration-audit 단계 추가
no → 표준 feature-pipeline 사용

---

선택 결과를 Phase 4 산출물 생성에 반영한다.

## Phase 4: 하네스 설계 및 산출물 생성

패턴 파일에서 이 프로젝트에 맞는 구조를 선택한다.

**에이전트 설계 원칙** (`patterns/orchestration.md` 참조):
- 메인 = 라우터/판단자, 구현 = 전문 에이전트 위임
- 작업 유형 1개당 전문가 에이전트 1개
- 관련 예시 하네스(`patterns/examples/`)를 참고해 역할 분담

**스킬 설계 원칙**:
- 반복 워크플로우 1개당 스킬 1개
- 각 스킬은 Phase 구조 (진단 → 구현 → 검증)
- 탈출구(escape hatch) 규칙 반드시 포함

### CLAUDE.md (프로젝트 루트)

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

### .claude/agents/{name}.md

각 전문 에이전트 정의:

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

**옵션 A(지식 베이스)가 yes면** bugfix 에이전트에 다음 섹션을 추가한다:

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

**옵션 B(구현 완성도 감사)가 yes면** feature 에이전트에 다음 섹션을 추가한다:

```markdown
## 완성도 감사 프로토콜

기능 구현 완료를 선언하기 전에 반드시 실행한다.

1. UI 레이어 — onClick/onTap/onPressed 핸들러가 실제로 연결되어 있는가
2. 라우팅 레이어 — 새 라우트가 등록됐고 진입점이 존재하는가
3. 상태/데이터 레이어 — API 연결이 실제로 이루어졌는가, 더미 데이터가 남아 있지 않은가
4. 발견 사항을 심각도/신뢰도 포함해 보고하고, 승인 후 수정한다
```

### .claude/skills/{name}/skill.md

각 워크플로우 스킬:

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

생성 완료 후 CLAUDE.md를 읽어 구조가 일관성 있는지 검토하고, 필요 시 조정 후 사용자에게 보고한다.
