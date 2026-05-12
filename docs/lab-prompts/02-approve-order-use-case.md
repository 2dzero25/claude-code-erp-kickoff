# Task 2 — ApproveOrderUseCase 구현 (subagent 위임)

이 ERP 시스템의 주문 승인 흐름이 아직 비어 있다. `src/core/application/use-cases/ApproveOrderUseCase.ts` 의 `execute()` 메서드를 구현해서 주문 승인 use case를 완성한다.

이 task는 **subagent에 위임**해서 완료한다. Task 1과 달리 직접 Plan mode를 운영하지 않고, 위임 brief를 작성해서 subagent가 plan-then-implement를 수행하게 한다.

## 전제

- **Task 1 (DiscountPolicy) 가 완료된 상태**여야 한다. 본 use case가 `DiscountPolicy.apply()` 를 호출한다.

## 목표

`ApproveOrderUseCase.execute({ orderId })` 가 다음 흐름을 수행해야 한다.

1. `orderId` 로 주문 조회. 없으면 `OrderNotFoundError`
2. 주문 상태가 `PENDING_APPROVAL` 인지 확인. 아니면 `InvalidOrderStatusTransitionError`
3. 각 라인의 재고 확인. 부족하면 `OutOfStockError` (productId/requested/available 포함). 상태/재고는 그대로 유지
4. `DiscountPolicy.apply` 로 할인 적용한 최종 금액 계산
5. 라인별 재고 차감
6. 주문 상태를 `APPROVED` 로 전이 (`Order.transitionTo()` 활용)
7. 주문 저장
8. `{ orderId, finalAmount, approvedAt }` 형태의 `OrderApprovalResult` 반환

## 도메인 컨텍스트

- 위치: `src/core/application/use-cases/ApproveOrderUseCase.ts` (Application layer)
- 사용 가능한 도메인 자산
  - `Order` 엔티티 (`src/core/domain/entities/Order.ts`) — `status`, `lines`, `customerGrade`, `originalAmount()`, `transitionTo(target)`
  - `OrderRepository` 인터페이스 (`src/core/application/interfaces/OrderRepository.ts`)
  - `InventoryRepository` 인터페이스 (`src/core/application/interfaces/InventoryRepository.ts`)
  - `DiscountPolicy.apply` (`src/core/domain/policies/DiscountPolicy.ts`)
  - 에러 클래스들 (`src/core/domain/errors/OrderApprovalError.ts`)
- DTO: `OrderApprovalDto`, `OrderApprovalResult` (`src/core/application/dtos/OrderApprovalDto.ts`)
- 의존성 주입: 생성자에서 `OrderRepository`, `InventoryRepository` 를 주입받음

## 제약

- 새 npm 패키지를 설치하지 않는다
- `src/core/application/use-cases/ApproveOrderUseCase.ts` 외 파일은 수정하지 않는다 (도메인 엔티티/정책/에러, infrastructure repository, UI 모두 그대로)
- 기존 테스트 4개를 그대로 통과시킨다. 테스트를 추가/수정하지 않는다
- `docs/rules/architecture.md`, `docs/rules/tdd.md` 의 규칙을 따른다

## 테스트 contract

```
tests/application/ApproveOrderUseCase.test.ts   (4 케이스)
```

검증 명령:

```
npm test
```

전체 테스트 14개가 모두 green이어야 한다 (도메인 5 + 상태 전이 5 + use case 4).

## 운영 가이드 — Path A (default): 사전 제작된 subagent 사용

본 repo에는 `.claude/agents/use-case-implementer.md` 가 이미 박혀 있다. Claude Code Desktop이 자동으로 인식한다.

> `.claude/agents/` 디렉토리는 Claude Code **세션 시작 시점에만** 스캔된다. clone 직후 첫 세션에서는 바로 인식되지만, 이미 세션이 열려있던 상태에서 파일이 추가/수정되면 세션 재시작이 필요하다.

진행 순서
1. 아래 "복사용 위임 prompt"를 Claude에게 입력. Claude는 적절한 subagent로 위임을 검토함
2. 또는 명시적으로 `Use the use-case-implementer subagent to ...` 형태로 호출
3. subagent가 plan을 출력 → 검토 → 댓글로 의견 반영
4. subagent가 구현 → `npm test` 실행 → 결과 보고
5. 사용자가 main session에서 결과 검토

## 운영 가이드 — Path B: 직접 subagent 만들기

`/agents` 명령으로 자체 제작도 가능. 학습용으로 권장.

진행 순서
1. `/agents` 입력 → 에이전트 관리 화면
2. "Create new agent" 선택
3. 입력 항목
   - **Name**: `use-case-implementer` (또는 본인이 정한 이름)
   - **Description**: 한 줄 요약 — "Implements Clean Architecture application-layer use cases against existing failing tests"
   - **Tools**: `Read, Edit, Write, Bash, Glob, Grep`
   - **System prompt**: 아래 부록 system prompt template 그대로 붙여넣기
4. 저장 후 `.claude/agents/<name>.md` 가 생성된 것 확인
5. 같은 위임 prompt를 사용해 위임

## 복사용 위임 prompt

```
이 repo의 src/core/application/use-cases/ApproveOrderUseCase.ts 의 execute() 메서드를 구현해야 해. use-case-implementer subagent에 위임해서 완료해줘.

위임 brief:

목표
- ApproveOrderUseCase.execute({ orderId }) 가 주문 승인 use case 흐름을 수행하도록 구현한다
- 흐름: 조회 -> 상태 검증 -> 재고 검사 -> 할인 적용 (DiscountPolicy.apply 호출) -> 재고 차감 -> 상태 전이 (Order.transitionTo) -> 저장 -> OrderApprovalResult 반환
- 실패 분기: OrderNotFoundError, InvalidOrderStatusTransitionError, OutOfStockError (productId/requested/available 포함)
- 재고 부족 시 상태/재고 모두 변경 없음

scope 제약
- src/core/application/use-cases/ApproveOrderUseCase.ts 만 수정한다
- 다른 도메인/인프라/UI 파일은 손대지 않는다
- 새 npm 패키지를 추가하지 않는다
- 새 테스트를 추가하지 않는다 (기존 4개만 통과)

테스트 contract
- tests/application/ApproveOrderUseCase.test.ts 4 케이스 모두 통과
- 전체 npm test 14/14 통과

출력 형식 (Plan-then-Implement)
1. 변경 파일 / 책임 위치 / 흐름 단계 / 위험 요소 / 확인 질문 형태의 plan 먼저 출력
2. plan에 응답하면 그 plan대로 구현
3. npm test 실행 후 결과 보고. 실패 시 원인 분석 후 다시 진행
```

## 부록 — `/agents` wizard에 붙여넣을 system prompt template

`use-case-implementer` 자체 제작 시 이 본문을 system prompt로 붙여넣는다. 본 repo에 이미 박힌 `.claude/agents/use-case-implementer.md` 와 동일하다.

```
You are a Clean Architecture application-layer implementer. You receive a brief describing an application use case to build, and you complete it against existing failing tests.

Discipline:

1. Plan first. Output a plan covering changed files, responsibility placement, flow steps, risks, and any clarifying questions. Wait for approval before editing code.
2. Read before writing. Always read CLAUDE.md, docs/rules/architecture.md, docs/rules/tdd.md, and the target use case file plus its test file before producing the plan.
3. Implement minimally. Write the smallest code that makes the existing failing tests pass. Do not refactor unrelated code. Do not add new tests.
4. Respect dependency direction. Application depends only on Domain. Do not import infrastructure or framework code in the use case.
5. Do not add npm packages. If a brief seems to need one, ask first.
6. Verify before claiming done. After implementing, run npm test and report the actual output. Never claim a passing run you did not perform.
7. Stay in scope. Edit only the use case file unless the brief says otherwise. If the brief asks you to touch domain entities, repositories, infrastructure, or UI, push back first.
8. When uncertain, ask. Do not guess at unclear requirements.

Output format:

- Plan section: bullet list of "changed files", "flow", "errors raised", "risks", "questions"
- Implementation section: actual code edits
- Verification section: command run + result + a one-line conclusion

You may use these tools: Read, Edit, Write, Bash, Glob, Grep.
```
