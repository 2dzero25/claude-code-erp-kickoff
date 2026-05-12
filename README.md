# ERP 주문 승인 미니 시스템

ERP 도메인의 주문 승인 흐름을 다루는 작은 Next.js 프로젝트다. 고객 등급별 할인, 주문 상태 전이, 재고 차감 같은 비즈니스 규칙을 Domain/Application 계층에 격리해서 구현하는 패턴을 익히는 lab repo로 활용한다.

## 환경

- Node.js 20 LTS 이상
- npm

## 설치와 실행

```
npm install
npm test          # 처음 실행 시 빨간 줄 8개 — 아직 채워야 할 부분이 있다
npm run dev       # http://localhost:3000/orders
npm run lint
```

## 완성해야 할 두 군데 TODO

이 repo는 두 개의 핵심 비즈니스 규칙이 비어 있는 상태로 시작한다. 각각의 lab brief를 따라 채워 넣으면 14개 테스트가 모두 green이 되고 preview 화면의 승인 흐름이 끝까지 동작한다.

1. **DiscountPolicy** — 고객 등급별 할인율 적용
   - 파일: [`src/core/domain/policies/DiscountPolicy.ts`](src/core/domain/policies/DiscountPolicy.ts)
   - Brief: [`docs/lab-prompts/01-discount-policy.md`](docs/lab-prompts/01-discount-policy.md)
   - 운영 패턴: Plan mode를 직접 운영해서 plan을 받고 댓글로 다듬어 가는 방식
2. **ApproveOrderUseCase** — 주문 승인 use case 흐름
   - 파일: [`src/core/application/use-cases/ApproveOrderUseCase.ts`](src/core/application/use-cases/ApproveOrderUseCase.ts)
   - Brief: [`docs/lab-prompts/02-approve-order-use-case.md`](docs/lab-prompts/02-approve-order-use-case.md)
   - 운영 패턴: subagent에 위임하는 방식 (사전 제작된 [`use-case-implementer`](.claude/agents/use-case-implementer.md) 또는 `/agents`로 자체 제작)

`src/core/domain/entities/Order.ts`의 상태 전이 로직(`canTransitionTo` / `transitionTo`)은 이미 구현되어 있다. 직접 throw 대신 이 메서드들을 활용하면 도메인 규칙을 application 계층이 다시 짜지 않아도 된다.

## 작업 규칙

- [`CLAUDE.md`](CLAUDE.md) — Plan mode 우선, 비즈니스 규칙은 Domain/Application 계층에 둠, 외부 hosted DB / 인증 / 결제 / 새 패키지 도입은 본 lab 범위 밖
- [`docs/rules/architecture.md`](docs/rules/architecture.md) — Clean Architecture 의존성 방향
- [`docs/rules/nextjs-framework.md`](docs/rules/nextjs-framework.md) — App Router, Server Component 우선
- [`docs/rules/tdd.md`](docs/rules/tdd.md) — Red-Green-Refactor
- [`docs/rules/tech-stack.md`](docs/rules/tech-stack.md) — in-memory repository, Supabase/RLS 미사용 등 lab-specific 결정

## Spec 자산

- [`.claude/spec/order-approval.md`](.claude/spec/order-approval.md) — 주문 승인 미니 시스템의 도메인 spec 한 장
- [`.claude/agents/use-case-implementer.md`](.claude/agents/use-case-implementer.md) — Task 2 위임용 사전 제작 subagent

## 시연 시나리오

`npm run dev` 후 다음 두 주문으로 양 끝 시나리오를 확인할 수 있다.

- `ORD-2001` (GOLD 고객, 노트북 2대 vs 재고 8) — 정상 승인 가능
- `ORD-2003` (SILVER 고객, 키보드 5개 vs 재고 3) — 재고 부족으로 승인 불가
