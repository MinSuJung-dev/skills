---
name: apply-harness
description: 검증된 하네스 패턴을 새 프로젝트에 적용한다. "하네스 적용해줘", "에이전트 팀 구성해줘", "이 프로젝트에도 같은 시스템 적용해줘", "자동화 체계 만들어줘" 등 신규 프로젝트에 오케스트레이터+전문가 에이전트 구조를 구축할 때 사용.
---

# Apply Harness

검증된 하네스 패턴을 이 프로젝트에 맞게 적용한다.

## Phase 1: 패턴 파일 로드

아래 패턴 파일을 전부 Read한다 (경로는 이 스킬 디렉토리 기준).

See [patterns/orchestration.md](patterns/orchestration.md) — 오케스트레이션 원칙
See [patterns/feature-pipeline.md](patterns/feature-pipeline.md) — 기능 개발 파이프라인
See [patterns/bugfix-pipeline.md](patterns/bugfix-pipeline.md) — 버그 수정 파이프라인
See [patterns/cleanup-pipeline.md](patterns/cleanup-pipeline.md) — 코드 정리 파이프라인
See [patterns/escape-hatch.md](patterns/escape-hatch.md) — 직접 수정 vs 위임 규칙
See [patterns/examples/flutter-app.md](patterns/examples/flutter-app.md) — Flutter 앱 예시
See [patterns/examples/python-backend.md](patterns/examples/python-backend.md) — Python 백엔드 예시
See [patterns/templates.md](patterns/templates.md) — 산출물 템플릿 (CLAUDE.md / agents / skills)

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

프로젝트 분석 결과를 요약해 보여준 뒤, 사용자에게 다음 옵션을 **각각 yes/no로** 묻는다.

**[옵션 A] 지식 베이스 관리** (기본값: yes)
bugfix 에이전트가 `investigate` 프로토콜을 따른다. 버그 작업마다 `.bugs/INDEX.md` 조회 → 버그 카드 생성·유지 → 종료 시 패턴 라이브러리 갱신. 카드 10개 이상이면 `/knowledge-prune` 안내.
yes → bugfix 에이전트에 investigate 워크플로우, CLAUDE.md에 지식 베이스 섹션 / no → 표준 bugfix-pipeline

**[옵션 B] 구현 완성도 자동 감사** (기본값: yes)
feature 에이전트가 완료 선언 전 `integration-audit` 프로토콜로 빈 핸들러·끊긴 라우트·더미 데이터를 탐지하고, 보고 후 승인받아 수정한다.
yes → feature 에이전트에 감사 단계 / no → 표준 feature-pipeline

**[옵션 C] DDD 도메인 엔지니어링** (기본값: no — 복잡한 비즈니스 규칙이 보이거나 `docs/domain/`이 있으면 yes 추천)
기능 개발이 `ddd` 스킬 프로토콜을 따른다. architect는 `design` 모드(설계 검증 게이트 통과 후에만 spec), dev는 `implement` 규칙, qa는 `review` 모드. 단순 CRUD 기능은 architect가 판정하고 게이트를 건너뛴다.
yes → `ddd` 스킬 설치 확인 (없으면 `claude plugin install ddd` 안내), architect/dev/qa 에이전트와 CLAUDE.md에 DDD 섹션 / no → 추가 없음

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

### 산출물

[patterns/templates.md](patterns/templates.md)의 템플릿으로 생성한다. 옵션 A/B/C 선택에 따라 해당 섹션을 추가한다.

- `CLAUDE.md` (프로젝트 루트)
- `.claude/agents/{name}.md` — 전문 에이전트마다 1개
- `.claude/skills/{name}/SKILL.md` — 워크플로우마다 1개 (파일명 대문자)

생성 완료 후 CLAUDE.md를 읽어 구조가 일관성 있는지 검토하고, 필요 시 조정 후 사용자에게 보고한다.
