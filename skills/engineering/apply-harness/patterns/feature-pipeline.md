# 기능 개발 파이프라인

## 구조: 3단계 서브 에이전트 파이프라인

```
[메인] → [architect] → _workspace/01_spec.md
       → [dev]        → _workspace/02_summary.md
       → [qa]         → _workspace/03_report.md
```

## Phase 0: 사전 준비

1. `_workspace/` 디렉토리 생성
2. 요구사항을 `_workspace/00_requirements.md`에 저장

## Phase 1: 설계 (architect 에이전트)

**입력:** 사용자 요구사항, 기존 유사 모듈/컴포넌트
**출력:** `_workspace/01_architect_spec.md`

spec 파일 필수 포함 항목:
- 새로 생성할 파일 목록 (경로 포함)
- 수정할 기존 파일 (경로 + 변경 내용)
- 라우트/API/스키마 변경 사항
- 상태 관리 구조 (컨트롤러/스토어/훅)

**architect 에이전트 호출 패턴:**
```
subagent_type: general-purpose
model: opus
prompt: |
  당신은 {프로젝트}-architect 에이전트입니다.
  에이전트 정의: .claude/agents/{name}-architect.md
  CLAUDE.md를 읽고 프로젝트 컨벤션을 파악하라.
  기존 유사 모듈을 탐색하고 _workspace/01_architect_spec.md를 작성하라.
  요구사항: {요구사항}
```

완료 조건: `_workspace/01_architect_spec.md` 존재

## Phase 2: 구현 (dev 에이전트)

**입력:** `_workspace/01_architect_spec.md`
**출력:** 실제 파일들 + `_workspace/02_dev_summary.md`

dev 에이전트는 spec을 그대로 구현한다. spec 외 추가/리팩토링 금지.

**dev 에이전트 호출 패턴:**
```
subagent_type: general-purpose
model: opus
prompt: |
  당신은 {프로젝트}-dev 에이전트입니다.
  에이전트 정의: .claude/agents/{name}-dev.md
  _workspace/01_architect_spec.md를 읽고 명세대로 구현하라.
  완료 후 _workspace/02_dev_summary.md를 작성하라.
```

완료 조건: `_workspace/02_dev_summary.md` 존재

## Phase 3: 검증 (qa 에이전트)

**입력:** `_workspace/01_architect_spec.md`, `_workspace/02_dev_summary.md`, 수정된 파일들
**출력:** `_workspace/03_qa_report.md`

QA 체크리스트 (도메인에 맞게 조정):
- [ ] 절대 규칙 위반 없음
- [ ] 누락된 파일/라우트 등록 없음
- [ ] 상태 관리 메모리 누수 없음
- [ ] Analytics 이벤트 등록 완료

## 메인 에이전트 마무리

`_workspace/03_qa_report.md`를 읽고:
- LGTM → 사용자에게 완료 보고
- NEEDS_FIX → dev 에이전트에 재위임 (수정 항목 명시)
