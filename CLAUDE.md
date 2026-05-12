# ERP 주문 승인 미니 시스템

이 저장소는 ERP 주문 승인 미니 시스템의 base repo다. 다음 규칙을 강하게 적용한다.

## 작업 절차

- 항상 Plan mode로 먼저 계획을 제시한다. 파일 수정은 plan 승인 이후에만 한다.
- 구현 전에 변경 파일, 테스트 전략, 위험 요소를 plan에 명시한다.
- 비즈니스 규칙은 `src/core/domain/`과 `src/core/application/`에 둔다. UI나 repository에 비즈니스 로직을 섞지 않는다.

## 범위 제한

- 외부 hosted DB (Supabase, Postgres 등), 인증, 결제, 외부 API, 배포 자동화 기능을 추가하지 않는다.
- 새 npm 패키지를 설치하기 전에 이유와 대안을 먼저 제시한다.
- repository는 in-memory 구현체를 사용한다. 영속성/마이그레이션/RLS는 이 프로젝트 범위 밖이다.

## 검증

- 완료 보고 전에 `npm test`와 `npm run lint`를 실행한다.
- 테스트가 실패하면 통과로 보고하지 않는다. 원인과 다음 수정 계획을 먼저 보고한다.
- 추측 구현하지 않는다. 불확실한 요구사항은 질문한다.

## 더 자세한 규칙

- [`docs/rules/architecture.md`](docs/rules/architecture.md): Clean Architecture 의존성 방향, 계층 책임, repository interface, DTO
- [`docs/rules/nextjs-framework.md`](docs/rules/nextjs-framework.md): App Router, Server Component 우선, Server Action 정책
- [`docs/rules/tdd.md`](docs/rules/tdd.md): Red-Green-Refactor 사이클
- [`docs/rules/tech-stack.md`](docs/rules/tech-stack.md): 이 프로젝트의 스택과 in-memory repository 정책

## 채워야 할 TODO 두 군데

- `src/core/domain/policies/DiscountPolicy.ts`
- `src/core/application/use-cases/ApproveOrderUseCase.ts`

`src/core/domain/entities/Order.ts`의 상태 전이 로직은 이미 구현되어 있다.
