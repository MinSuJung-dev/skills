# 산출물 템플릿

섹션 이름은 원 캔버스(ddd-crew)의 영어 이름을 유지한다.

## `docs/domain/glossary.md`

```markdown
# 용어집

| 용어 | 코드명 | 정의 (1–2문장) | 쓰지 말 것 | 컨텍스트 | 상태 |
|------|--------|---------------|-----------|---------|------|
| 주문 | Order | 고객이 상품 구매를 요청한 기록. | 오더, 구매건 | 주문 | 확정 |
| 고객 | Customer | … | 사용자, 회원 | 주문 | 충돌: 결제 컨텍스트는 "회원" 사용 |
```

- 같은 단어가 컨텍스트마다 다른 뜻이면 행을 나눈다. 억지로 하나로 합치지 않는다.
- 구현 세부(테이블명, API 경로)는 적지 않는다.

## 이벤트 후보 → 확정

스토리의 `이벤트 후보`에서 반응하는 쪽이 확인된 것만 컨텍스트 캔버스의 Outbound Communication과 애그리거트의 Created Events로 옮긴다.

## `docs/domain/context-map.md`

```markdown
# 컨텍스트 맵

## 서브도메인
| 서브도메인 | 분류 | 근거 (경계 단서) | 컨텍스트 |
|-----------|------|-----------------|---------|
| 주문 | core | 트리거 차이 (고객 요청) | ordering |

## 관계
| upstream | downstream | 패턴 | 메모 |
|----------|-----------|------|------|
| ordering | shipping | Customer/Supplier | 주문 확정 이벤트 발행 |
| 외부 PG | payment | Anticorruption Layer | PG 응답 모델을 내부 모델로 변환 |
```

패턴 목록: Partnership · Shared Kernel · Customer/Supplier · Conformist · Anticorruption Layer · Open Host Service · Published Language · Separate Ways.

## `docs/domain/contexts/<name>.md` — Bounded Context Canvas

```markdown
# <Name> 컨텍스트

## Purpose
## Strategic Classification
- Domain: core | supporting | generic
- Business model: revenue | engagement | compliance | cost reduction
- Evolution: genesis | custom | product | commodity
## Domain Roles
## Inbound Communication
| 보내는 쪽 | 메시지 (command/query/event) |
## Outbound Communication
| 받는 쪽 | 메시지 |
## Ubiquitous Language
→ glossary.md에서 컨텍스트 = <name> 인 용어
## Business Decisions
## Assumptions
## Verification Metrics
## Open Questions

## Aggregates
### <AggregateName>
- Description:
- 경계 근거: (REVIEW.md 애그리거트 경계 질문 답)
- Entities / Value Objects:
- State Transitions: 초안 → 확정 → 취소
- Enforced Invariants: (규칙과 예시의 출처 스토리 번호 표기)
- Corrective Policies: (불변식을 한 트랜잭션에서 못 지킬 때 사후 처리)
- Handled Commands:
- Created Events: (과거형 도메인 용어, 반응하는 쪽이 있는 것만. Domain / Integration 표시)
- Throughput: 동시 수정 가능성 낮음/중간/높음
- Size: 인스턴스당 이벤트 수 추정
```

## 유스케이스 (contexts/<name>.md 안에)

```markdown
### <UseCaseName> (도메인 동사: PlaceOrder, CancelOrder)
- 행위자:
- 출처 스토리: stories/NNN (문장 번호)
- 입력:
- 흐름: 로드 <Aggregate> → <Aggregate>.<method>() → 저장 → <Event> 발행
- 지키는 규칙: (규칙과 예시에서)
- 외부 의존 (포트):
- 트랜잭션: 애그리거트 1개 | 이유 있는 예외
- 실패 경우:
```

애그리거트 경계 질문은 REVIEW.md, 로직 위치·이벤트 규칙은 ARCHITECTURE.md를 따른다.
