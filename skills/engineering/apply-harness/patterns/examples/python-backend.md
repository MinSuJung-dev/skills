# 예시: Python 백엔드 하네스

## 기술 스택
Flask/FastAPI + Python + (선택) Ollama/ML 모델 + SQLite/PostgreSQL

## 에이전트 구성 예시 (분야별)

### 웹 API 서비스 (3개)
| 에이전트 | 역할 |
|---------|------|
| `{app}-dev` | Flask/FastAPI 라우트, 서비스 레이어, DB 모델 구현 |
| `{app}-qa` | 엔드포인트 검증, 에러 핸들링, 보안 취약점 검토 |
| `{app}-devops` | Dockerfile, 의존성, 환경변수 관리 |

### ML/AI 서비스 (4개)
| 에이전트 | 역할 |
|---------|------|
| `{app}-engineer` | 핵심 알고리즘/분석 엔진 구현 |
| `{app}-model-optimizer` | 모델/프롬프트 최적화 |
| `{app}-fullstack-dev` | API 레이어 + UI |
| `{app}-validator` | 정확도 측정, 테스트 케이스 관리 |

## 스킬 구성

### 웹 API 서비스
| 스킬 | 트리거 |
|-----|--------|
| `{app}-feature` | 신규 엔드포인트/기능 개발 |
| `{app}-bugfix` | 버그 수정 |
| `{app}-test` | 테스트 커버리지 추가 |

### ML/AI 서비스
| 스킬 | 트리거 |
|-----|--------|
| `{app}-orchestrator` | 복합 작업 조율 (새 기법, 모델 개선, 리팩토링) |
| `{app}-technique` | 새 분석/처리 기법 추가 |
| `{app}-tuning` | 모델/프롬프트 최적화 |
| `{app}-test` | 정확도 검증 |

## 정적 분석 명령

```bash
# Python
flake8 {파일} --max-line-length 100
mypy {파일}

# 테스트
pytest tests/ -v
python -m pytest {테스트파일} -k "{테스트명}"
```

## CLAUDE.md 핵심 파일 위치 표 예시

```markdown
| 역할 | 경로 |
|------|------|
| 메인 앱 | `app.py` |
| 분석 엔진 | `forensics.py` / `analyzer.py` |
| 모델 정의 | `Modelfile` |
| 테스트 이미지 | `test_images/` |
| 의존성 | `requirements.txt` |
```

## 에이전트 간 프로토콜 예시 (ML 서비스)

```markdown
## 입력/출력 계약
- engineer → validator: 분석 함수 구현 완료 알림 (SendMessage)
- validator → engineer: 정확도 수치 + 실패 케이스 반환
- optimizer → engineer: 점수 가중치 변경 시 사전 협의 필요
```
