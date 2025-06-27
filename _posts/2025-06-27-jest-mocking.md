---
title: Jest Mocking & Spy 
categories: [Frontend, Testcode]
tags: [frontend, testcode, jest, mock, spy]
layout: post
---

## 🎯 **Mocking 이란?**

> 테스트하려는 코드가 의존하는 부분을 가짜(Mock) 객체로 대체하는 기술이다. 이를 통해 외부 의존성을 제어하고, 테스트가 예측 가능한 환경에서 항상 동일한 결과를 내도록 보장할 수 있다.
{: .prompt-info}

### 의존성 이란?
> 쉽게 말해 코드의 대화 상대를 의미한다. 프론트엔드로 치면 내 코드가 백엔드의 API와 통신해서 데이터를 가져오기 때문에 백엔드 API가 수정이 되면  내 코드의 테스트가 실패하게 되는것이다. 이러한 경우를 **의존**한다고 한다.
{: .prompt-tip}


## 1. Mock 함수 기본: `jest.fn()` - 가짜 함수 만들기

`jest.fn()`는 실제 구현은 없지만, 함수가 어떻게 호출되었는지, 어떤 인자를 받았는지, 몇 번 호출되었는지 등 모든 것을 기억한다.

- .`toHaveBeenCalled()`: 함수가 호출되었는지 여부를 확인한다.
- .`toHaveBeenCalledTimes(횟수)`: 함수가 **몇 번** 호출되었는지 확인한다.
- .`toHaveBeenCalledWith(인자)`: **특정 인자**가 함수에 전달되었는지 확인한다.

### 호출 추적

가짜 함수가 어떻게 사용되었는지 추적한다.

- `.mockCallback`: 테스트할 때 사용하는 가짜 함수

```typescript
it("Mock 함수는 호출 여부와 인자를 추적한다", () => {
  // Arrange: Mock 함수 생성
  const mockCallback = jest.fn();

  // Act: Mock 함수 호출
  mockCallback("첫번째 호출");
  mockCallback("두번째 호출", 123);

  // Assert: 호출 검증
  expect(mockCallback).toHaveBeenCalled(); // 호출되었는가?
  expect(mockCallback).toHaveBeenCalledTimes(2); // 몇 번 호출되었는가?
  expect(mockCallback).toHaveBeenCalledWith("첫번째 호출"); // 특정 인자로 호출되었는가?
  expect(mockCallback).toHaveBeenNthCalledWith(2, "두번째 호출", 123); // n번째 호출의 인자는?
});
```

### 반환값 설정

Mock 함수가 특정 값을 반환하도록 설정할 수 있다. API 요청 결과를 시뮬레이션할 때 유용하다.

- `.mockReturnValue(value)`: 항상 특정(`value`) 값을 반환한다.
- `.mockReturnValueOnce(value)`: 딱 한 번만 특정(`value`) 값을 반환하고, 그 이후에는 기본 반환값(설정했다면)이나 `undefined`를 반환한다.

```typescript
it("Mock 함수는 반환값을 설정할 수 있다", () => {
    //Arrange
  const mockFunction = jest.fn();
  mockFunction.mockReturnValue("기본 반환값");

    //Act & Assert
  expect(mockFunction()).toBe("기본 반환값");

  // 일회성 반환값 설정
  mockFunction.mockReturnValueOnce("한번만 반환");
  expect(mockFunction()).toBe("한번만 반환");
  
  // 다시 기본값으로 돌아온다
  expect(mockFunction()).toBe("기본 반환값");
});
```

### 커스텀 구현 제공

`jest.fn()`에 직접 함수를 전달하여 실제 로직을 심을 수도 있다.

```typescript
it("Mock 함수는 사용자 정의 구현을 가질 수 있다", () => {
  // Arrange
  const mockAdd = jest.fn((a: number, b: number) => a + b + 100);

  // Act
  const result = mockAdd(5, 3);

  // Assert
  expect(result).toBe(108); // 5 + 3 + 100
  expect(mockAdd).toHaveBeenCalledWith(5, 3);
});
```
### 비동기 처리
`async/await` 구문과 자연스럽게 동작하도록 Promise를 반환하는 Mock 함수를 만들 수 있다.

- `.mockResolvedValue(value)`: 성공적으로 `resolve`되는 Promise를 반환한다.
- `.mockRejectedValue(error)`: `reject`되는 Promise를 반환한다.

```typescript
it("비동기 Mock 함수는 Promise를 반환할 수 있다", async () => {
  const mockAsyncFunction = jest.fn();
  
  // 성공 케이스
  mockAsyncFunction.mockResolvedValue("비동기 성공 결과");
  await expect(mockAsyncFunction()).resolves.toBe("비동기 성공 결과");

  // 실패 케이스
  mockAsyncFunction.mockRejectedValue(new Error("비동기 에러"));
  await expect(mockAsyncFunction()).rejects.toThrow("비동기 에러");
});
```

---

## 2. Spy 기본: `jest.spyOn()`

`jest.fn()`이 가짜 함수를 만드는 것이라면, `jest.spyOn()`은 **이미 존재하는 객체의 메소드를 감시(Spying)**하는 역할을 한다.

**Mock vs. Spy**

> - **Mock**: 가짜 함수. 원래 구현이 없다.
- **Spy**: 실제 함수를 감싸는 Wrapper. 기본적으로 원래 구현을 그대로 실행하면서 호출 여부를 감시한다.
{: .prompt-tip}

### 기존 동작 유지하며 감시하기
실제 메소드가 실행되도록 내버려 두면서 호출 정보만 기록한다.

```ts
it("Spy는 기존 메서드를 감시하면서 원래 동작을 유지한다", () => {
  const calculator = new Calculator();
  const addSpy = jest.spyOn(calculator, "add");

  const result = calculator.add(2, 3);

  expect(result).toBe(5); // 원래 동작 확인
  expect(addSpy).toHaveBeenCalledWith(2, 3); // spy 호출 확인

  // 테스트가 끝나면 원래대로 복구하는 것이 좋다
  addSpy.mockRestore();
});
```
addSpy는 add함수가 2와 3을 인자로 호출되었다는 사실을 기록한다.

### 기존 동작 변경하기
Spy도 Mock처럼 동작을 바꿀 수 있다. `mockReturnValue`나` mockImplementation` 등을 사용하면 된다.

```ts
it("Spy는 원래 동작을 변경할 수 있다", () => {
  const calculator = new Calculator();
  const multiplySpy = jest.spyOn(calculator, "multiply");
  multiplySpy.mockReturnValue(999); // 원래 곱셈 대신 999를 반환

  const result = calculator.multiply(5, 10);

  expect(result).toBe(999); // 변경된 동작 확인
  expect(multiplySpy).toHaveBeenCalledWith(5, 10);

  multiplySpy.mockRestore();
});
```

---

## 3. 의존성 Mocking: 실제 코드처럼 테스트하기

대부분의 서비스 클래스는 다른 서비스나 모듈에 의존한다.  
예를 들어 `UserService`는 API 통신을 담당하는 `ApiClient`에 의존할 수 있다.

```typescript
// 테스트 대상인 UserService는 ApiClient를 주입받는다.
// constructor(private apiClient: ApiClient) {}

it("API 클라이언트를 mocking하여 UserService 테스트", async () => {
  // Arrange: 가짜 API 클라이언트(의존성) 생성
  const mockApiClient = {
    get: jest.fn().mockResolvedValue({
      id: 1,
      name: "김개발",
      email: "kim@test.com",
    }),
  };

  // Act: 의존성을 주입하여 서비스 테스트
  const userService = new UserService(mockApiClient as any);
  const user = await userService.getUser(1);

  // Assert: 결과와 호출 확인
  expect(user.name).toBe("김개발");
  expect(mockApiClient.get).toHaveBeenCalledWith("/api/users/1");
});
```

위 코드의 핵심은 실제 `ApiClient`가 아니라, 우리가 제어할 수 있는 가짜 `mockApiClient`를 `UserService`에 주입했다는 점이다. 덕분에 네트워크 연결 없이도 `UserService`가 `ApiClient`를 올바르게 사용하는지 검증할 수 있다.

잘 이해가 안가서 AI에게 설명을 요구했더니 알아듣기 쉽게 설명해줬다

<details>
<summary><b>의존성</b></summary>
<br>
<b>의존성 Mocking - 조립 로봇 테스트하기</b><br>
현실의 코드는 여러 부품(모듈, 서비스)을 조립해서 만듭니다.<br>
'주문 서비스'는 <b>사용자 정보 모듈</b>, <b>결제 모듈</b>, <b>이메일 발송 모듈</b>을 가져와 조립한 결과물일 수 있습니다.<br><br>

우리가 테스트하고 싶은 것은 <b>'주문 서비스'가 이 부품들을 올바른 순서로 잘 조립해서 사용하는가'</b>이지, 각 부품의 성능이 아닙니다.<br>
<i>(부품 테스트는 각자 따로 진행해야 합니다.)</i>
<hr>
<b>설명</b><br>
UserService를 테스트하는데, 이 서비스는 <b>ApiClient</b>라는 'API 통신 부품'을 사용합니다.<br>
우리는 <code>new UserService(mockApiClient)</code>처럼 가짜 부품을 끼워서 UserService를 조립합니다.<br>
그리고 <code>userService.getUser(1)</code>를 실행했을 때, UserService가 <code>mockApiClient.get</code>이라는 부품의 기능을 올바른 규격(<code>/api/users/1</code>)에 맞춰 잘 사용하는지만 확인합니다.<br>
<hr>
<b>복잡한 서비스도 마찬가지</b><br>
OrderService가 UserService, EmailService, Utils라는 3개의 부품을 쓴다면, 그냥 가짜 부품 3개를 다 만들어서 끼워주면 됩니다.<br>
그리고 주문 생성 로직을 실행했을 때,<br>
1. UserService의 <code>getUser</code>를 먼저 호출하고<br>
2. 그다음 EmailService의 <code>sendEmail</code>을 호출하는 등의 <b>조립 순서</b>가 올바른지 검증하는 것입니다.<br>
</details>

### 복잡한 서비스 Mocking

하나가 아닌 여러 의존성을 가질 때도 각 의존성을 모두 Mock으로 만들어서 주입하면 된다.

```ts
it("주문 서비스는 사용자, 이메일, 유틸 서비스를 모두 사용한다", async () => {
  // Arrange: 모든 의존성 mock 설정
  const mockUserService = { 
    getUser: jest.fn().mockResolvedValue({
        id: 1,
        email: "customer@test.com",
        }),
        /* ... */ };
  const mockEmailService = { 
    sendEmail: jest.fn().mockResolvedValue(true),
    /* ... */ };
  const mockUtils = {
    generateId: jest.fn().mockReturnValue("order-123"),
    /* ... */
  };

  // Act: 모든 의존성을 주입하여 OrderService 생성
  const orderService = new OrderService(mockUserService as any, mockEmailService, mockUtils as any);
  const order = await orderService.createOrder(1, [{ name: "상품A", price: 10000, quantity: 2 }]);

  // Assert: 주문 생성 프로세스 전체 검증
  expect(order.id).toBe("order-123");
  expect(mockUserService.getUser).toHaveBeenCalledWith(1);
  expect(mockEmailService.sendEmail).toHaveBeenCalledWith(
    "customer@test.com",
    "주문 확인",
    "주문 번호 order-123가 접수되었습니다."
  );
  expect(mockUserService.updateUserActivity).toHaveBeenCalledWith(1, "ORDER_CREATED");
});
```
이 테스트는 `OrderService`가 주문 생성이라는 비즈니스 로직을 수행할 때, 각 의존성(`UserService`, `EmailService`, `Utils`)의 메소드를 올바른 순서와 인자로 호출하는지를 검증한다. 각 의존성의 내부 동작은 신경 쓸 필요가 없다.

---

## 4. 실시간 함수 Mocking

테스트는 언제나 예측 가능해야 하지만 코드에는 `new Date()`나 `Math.random()`처럼 실행할 때마다 결과가 달라지는 예측 불가능한 요소들이 존재한다. 이는 `jest.spyOn`을 사용하여 제어할 수 있다.

- 시간 고정하기: `jest.spyOn(Date, 'now').mockReturnValue(fixedTime)`
- 랜덤값 고정하기: `jest.spyOn(Math, 'random').mockReturnValue(0.5)`

```ts
it("Date.now()를 고정하여 시간 의존적 코드 테스트", () => {
    // Arrange: 시간을 고정
    const fixedTime = 1609459200000; // 2021-01-01
    const dateNowSpy = jest.spyOn(Date, "now").mockReturnValue(fixedTime);

    // Act
    const timestamp = Date.now();

    // Assert
    expect(timestamp).toBe(fixedTime);

    // Clean up
    dateNowSpy.mockRestore();
});
```

## 5. Mock 상태 관리와 테스트 격리

테스트는 서로에게 영향을 주지 않고 독립적으로 실행되어야 한다(Independent). 한 테스트에서 변경한 Mock의 상태가 다른 테스트에 영향을 미치면 안 된다.

### Mock 초기화
Jest는 Mock의 상태를 초기화하는 여러 방법을 제공한다.

- `mockClear()`: 호출 횟수(`mock.calls`)와 결과(`mock.results`)만 초기화. `mockReturnValue` 등으로 설정한 구현은 유지된다.
- `mockReset()`: `mockClear()`를 포함하여, `mockReturnValue` 같은 구현까지 모두 초기화하여 완전한 맨 처음 상태(`jest.fn()`)로 되돌린다.
- `mockRestore()`: `jest.spyOn`으로 감시하던 메소드를 원래 구현으로 완전히 복구한다.` mockReset()`의 효과를 포함한다.

    **언제 무엇을 써야 할까?**
    `beforeEach`나 `afterEach`에서 모든 Mock을 깔끔하게 정리하는 것이 좋다.
    `jest.clearAllMocks()`: 모든 mock의 `.mock.calls`와 `.mock.results`를 초기화. (`mockClear`와 동일)
    `jest.resetAllMocks()`: 모든 mock을 리셋. (`mockReset`과 동일)
    `jest.restoreAllMocks()`: 모든 spy를 복구. (`mockRestore`와 동일)

> `beforeEach`에 `jest.clearAllMocks()`를 넣어 테스트 간의 호출 기록이 꼬이는 것을 방지한다.
{: .prompt-warning}

```ts
describe("🧹 Mock 상태 관리", () => {
  // 각 테스트 전에 모든 mock의 호출 기록을 초기화한다.
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it("beforeEach에서 모든 mock을 초기화하여 테스트 격리", () => {
    const mockFunction = jest.fn();
    // 다른 테스트에서 이 함수를 몇 번 호출했든 상관없이,
    // 이 테스트가 시작될 때는 항상 0에서 시작한다.
    expect(mockFunction).toHaveBeenCalledTimes(0);
  });
});
```

## 6. 실무에서 자주 쓰는 Mocking 패턴

### 에러 상황 테스트
API 요청이 실패하거나 로직에서 에러가 발생하는 상황을 테스트하기 위해 `mockRejectedValue`나 `mockImplementation`을 사용하여 의도적으로 에러를 발생시킬 수 있다.

```ts
it("API 에러 상황 테스트", async () => {
  // Arrange: API 실패 시나리오
  const mockApiClient = {
    get: jest.fn().mockRejectedValue(new Error("네트워크 오류")),
  };
  const userService = new UserService(mockApiClient as any);

  // Act & Assert
  // userService.getUser는 내부적으로 에러를 잡아서 "사용자를 찾을 수 없습니다" 예외를 던진다고 가정
  await expect(userService.getUser(1)).rejects.toThrow("사용자를 찾을 수 없습니다");
});
```

### 연속 호출에 다른 값 반환

API 재시도 로직 등을 테스트할 때 유용하다. `mockReturnValueOnce`를 체이닝하여 호출할 때마다 다른 값을 반환하게 만들 수 있다.

```ts
it("연속 호출에서 다른 값 반환하기", () => {
  const mockRetry = jest.fn();
  mockRetry
    .mockReturnValueOnce({ success: false, error: "첫 번째 실패" })
    .mockReturnValueOnce({ success: false, error: "두 번째 실패" })
    .mockReturnValue({ success: true, data: "세 번째 성공" }); // 마지막은 Once가 아님

  expect(mockRetry()).toEqual({ success: false, error: "첫 번째 실패" });
  expect(mockRetry()).toEqual({ success: false, error: "두 번째 실패" });
  expect(mockRetry()).toEqual({ success: true, data: "세 번째 성공" });
  expect(mockRetry()).toEqual({ success: true, data: "세 번째 성공" }); // 계속 마지막 값 반환
});
```