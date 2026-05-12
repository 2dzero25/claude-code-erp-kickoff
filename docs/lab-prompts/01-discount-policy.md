# Task 1 — DiscountPolicy 구현

이 ERP 시스템에는 고객 등급별 할인 규칙이 아직 비어 있다. `src/core/domain/policies/DiscountPolicy.ts` 의 `apply()` 메서드를 구현해서 등급별 할인 정책을 완성한다.

## 목표

`DiscountPolicy.apply(originalAmount, grade)` 가 등급별 할인율을 적용한 최종 금액을 반환하도록 한다.

- `NORMAL` 고객: 할인 없음
- `SILVER` 고객: 5% 할인
- `GOLD` 고객: 10% 할인
- 음수 금액 입력은 허용하지 않는다

## 도메인 컨텍스트

- 위치: `src/core/domain/policies/DiscountPolicy.ts` (Domain layer)
- 의존성: 없음. Domain 계층은 외부 라이브러리/프레임워크를 import하지 않는다 (`docs/rules/architecture.md` 참고)
- 사용처: 추후 `ApproveOrderUseCase` (Task 2)에서 호출된다. 본 task 시점에는 아직 호출처가 없어도 무방
- 관련 타입: `CustomerGrade` (`src/core/domain/entities/Customer.ts`)

## 제약

- 새 npm 패키지를 설치하지 않는다
- `src/core/domain/policies/DiscountPolicy.ts` 외 파일은 수정하지 않는다 (테스트 파일 포함)
- 기존 테스트 5개를 그대로 통과시킨다. 테스트를 추가/수정하지 않는다
- `docs/rules/architecture.md`, `docs/rules/tdd.md` 의 규칙을 따른다
- `CLAUDE.md` 의 작업 절차를 따른다 (Plan mode 우선)

## 테스트 contract

```
tests/domain/DiscountPolicy.test.ts   (5 케이스)
```

검증 명령:

```
npm test -- DiscountPolicy
```

5/5 통과를 목표로 한다.

## 운영 가이드

이 task는 **Plan mode**로 진행한다.

1. Claude Code Desktop에서 권한 모드를 **Plan**으로 토글
2. 아래 Step 1 prompt 입력 → Claude가 plan 작성 (파일은 수정 안 됨)
3. plan을 읽고, 모호하거나 손대고 싶은 지점에 댓글로 follow-up 작성
   - 예: "반올림은 `Math.round`로 통일해줘", "음수 입력은 throw로 명확하게"
4. plan이 만족스러우면 Step 2 prompt로 구현 진입
5. 터미널에서 `npm test -- DiscountPolicy` 실행 후 5/5 green 확인

---

## 복사용 prompt body

### Step 1 — Plan mode 진입

```
src/core/domain/policies/DiscountPolicy.ts 의 apply() 메서드를 구현하려고 한다.

Plan mode로만 진행해줘. 아직 파일을 수정하지 마.

먼저 다음을 알려줘.

1. 변경할 파일 후보 (의도된 단일 파일이 맞는지 확인)
2. Domain layer 안에서의 책임 위치 (다른 계층으로 새지 않는지)
3. 먼저 통과시킬 테스트 케이스 순서
4. 새 패키지 없이 가능한 구현 전략
5. 위험 요소 또는 모호한 지점 (반올림 방향, 잘못된 입력 처리 등)

도메인 규칙은 다음과 같다.
- NORMAL 고객은 할인 없음
- SILVER 고객은 5% 할인
- GOLD 고객은 10% 할인

테스트는 tests/domain/DiscountPolicy.test.ts 5개를 그대로 통과해야 한다.
```

### Step 2 — 구현 진입

plan 댓글 반영이 끝난 뒤:

```
위에서 합의된 plan대로 src/core/domain/policies/DiscountPolicy.ts 만 구현해줘.

지키는 것
- 다른 파일은 수정하지 않는다 (테스트 포함)
- 새 패키지를 추가하지 않는다
- Domain layer 의존성 규칙을 위반하지 않는다 (외부 import 없음)

구현이 끝나면 npm test -- DiscountPolicy 를 실행해서 5/5 green을 보고해줘. 실패하면 원인과 수정 계획을 먼저 보고하고, 추측 수정은 하지 마.
```
