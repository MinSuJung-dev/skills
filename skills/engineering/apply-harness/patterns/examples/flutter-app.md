# 예시: Flutter 앱 하네스

## 기술 스택
Flutter + GetX (상태관리/라우팅) + Retrofit (API) + Hive/ObjectBox (로컬 DB) + flutter_screenutil

## 에이전트 구성 (4개)

| 에이전트 | 역할 |
|---------|------|
| `{app}-architect` | 신규 기능 설계, 모듈 구조 설계, _workspace/01_spec.md 작성 |
| `{app}-dev` | spec 기반 GetX 컨트롤러/뷰/바인딩 구현, Retrofit API 클라이언트, DTO 생성 |
| `{app}-qa` | 절대 규칙 준수 검토, GetX 메모리 누수, Analytics 누락, 패턴 일관성 |
| `{app}-entropy` | 미사용 파일, 문서-코드 불일치, 중복 패턴, 고착된 TODO 탐지 |

## 스킬 구성 (5개)

| 스킬 | 트리거 | 파이프라인 |
|-----|--------|----------|
| `{app}-feature` | 신규 기능/화면 개발 | architect → dev → qa |
| `{app}-bugfix` | 버그 수정 | phase0 분류 → 조사 → (위임) → 검증 |
| `{app}-cleanup` | 코드 정리 | entropy → report → 승인 → clean |
| `{app}-api` | API/DTO 생성 | 스펙 분석 → 생성 → build_runner |
| `{app}-module` | 모듈 뼈대 생성 | binding+controller+view 스캐폴딩 |

## 절대 규칙 예시 (Flutter/GetX)

```markdown
## 절대 규칙 (위반 시 QA NEEDS_FIX)
1. 컨트롤러는 **`{BaseController}` 상속** 필수
2. 크기 단위 **`.w` `.h` `.sp`** 필수 (하드코딩 숫자 금지)
3. 민감 데이터 → **`flutter_secure_storage`**
4. DTO/API 파일 생성 후 → **`dart run build_runner build`**
5. Analytics 이벤트명 → **등록 파일에 추가**
```

## CLAUDE.md 핵심 파일 위치 표 예시

```markdown
| 역할 | 경로 |
|------|------|
| 베이스 컨트롤러 | `lib/config/base_controller.dart` |
| Dio 클라이언트 | `lib/config/api_client.dart` |
| 라우트 이름 | `lib/app/routes/route_names.dart` |
| 라우트 등록 | `lib/app/routes/app_pages.dart` |
| Analytics 이벤트명 | `lib/app/routes/analytics_names.dart` |
```

## dev 에이전트 모드 B (버그 수정 위임) 핵심 규칙

```markdown
1. prompt에 명시된 라인만 수정
2. 범위 외 리팩토링 절대 금지
3. 완료 후 `flutter analyze {파일}` 실행 및 결과 보고
4. prompt가 모호하면 즉시 메인에 되묻기
```
