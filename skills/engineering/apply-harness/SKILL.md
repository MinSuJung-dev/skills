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

## Phase 3: 하네스 설계

패턴 파일에서 이 프로젝트에 맞는 구조를 선택한다.

**에이전트 설계 원칙** (`patterns/orchestration.md` 참조):
- 메인 = 라우터/판단자, 구현 = 전문 에이전트 위임
- 작업 유형 1개당 전문가 에이전트 1개
- 관련 예시 하네스(`patterns/examples/`)를 참고해 역할 분담

**스킬 설계 원칙**:
- 반복 워크플로우 1개당 스킬 1개
- 각 스킬은 Phase 구조 (진단 → 구현 → 검증)
- 탈출구(escape hatch) 규칙 반드시 포함

## Phase 4: 산출물 생성

다음 파일을 생성한다:

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
