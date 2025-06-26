---
title: "테스트코드 - Jest Matchers"
date: 2025-06-26
categories: [Frontend, Testcode]
tags: [frontend, testcode, jest, matcher]
layout: post
---

> Jest에서는 `Assert`단계에서 `expect` 함수와 함께 다양한 **Matcher**를 사용하여 검증 과정을 수행한다. Matcher는 값을 특정 조건과 비교해주는 함수들이다.
{: .prompt-tip }

이번 포스팅에서는 Matcher들을 종류별로 알아 보겠습니다.

## ✅ 기본 비교: **toBe**와 **toEqual**

* `toBe`: 숫자, 문자열, 불린 등 **원시 타입(Primitive type)** 값을 비교할 때 사용한다. `Object.is` (거의 `===` 와 동일)로 동작하여 정확한 일치 여부를 확인한다.
* `toEqual`: 객체나 배열의 **내용**이 같은지 재귀적으로 확인한다. 객체나 배열을 테스트할 땐 `toEqual`을 사용해야 한다.

> 객체나 배열은 참조 타입이므로, 내용이 같아도 메모리 주소가 다르면 `toBe` 비교는 실패한다.
{: .prompt-warning }

```typescript
it("toBe와 toEqual의 차이점을 이해한다", () => {
  const user1 = { name: "김개발" };
  const user2 = { name: "김개발" }; // 내용은 같지만 다른 객체
  const user3 = user1; // 같은 객체를 참조

  expect(user1).toBe(user2); // ❌ 다른 참조(메모리 주소)라서 실패
  expect(user1).not.toBe(user2); // ✅

  expect(user1).toEqual(user2); // ✅ 내용은 같으므로 성공
  expect(user1).toBe(user3); // ✅ 같은 참조이므로 성공
});
```

## ✅ 불린 검증: **toBeTruthy** 와 **toBeFalsy**

`true`/`false` 뿐만 아니라, JavaScript의 Truthy/Falsy 값을 검증할 때 유용하다.
* `toBeTruthy`: 값이 Truthy인지 확인한다 (`true`, 숫자(0제외), 문자열, 객체, 배열 등).
* `toBeFalsy`: 값이 Falsy인지 확인한다 (`false`, `0`, `""`, `null`, `undefined`, `NaN`).
* `toBeNull`, `toBeUndefined`: 값이 정확히 `null` 또는 `undefined`인지 확인할 때 사용한다.

```typescript
it("toBeTruthy는 truthy 값을, toBeFalsy는 falsy 값을 확인한다", () => {
  expect("hello").toBeTruthy();
  expect(42).toBeTruthy();
  expect({}).toBeTruthy();

  expect("").toBeFalsy();
  expect(0).toBeFalsy();
  expect(null).toBeFalsy();
});
```
```ts
it("toBeNull과 toBeUndefined로 특정 값을 확인한다", () => {
      // Arrange
      const inactiveUser = { isActive: false };
      const userWithoutStatus = { name: "test" };
      const nullUser = null;

      // Act & Assert
      expect(checkLoginStatus(nullUser)).toBeNull();
      expect(checkLoginStatus(userWithoutStatus)).toBeUndefined();
      expect(checkLoginStatus(inactiveUser)).not.toBeNull();
      expect(checkLoginStatus(inactiveUser)).not.toBeUndefined();
    });
```

## ✅ 숫자 비교: **toBeGreaterThan**, **toBeLessThan**, **toBeCloseTo**

숫자 값을 비교할 때는 더 직관적인 Matcher들을 사용할 수 있다.

* `toBeGreaterThan(숫자)`: 더 큰지 확인
* `toBeLessThan(숫자)`: 더 작은지 확인
* `toBeCloseTo`: 부동소수점 비교
    > 컴퓨터에서 `0.1 + 0.2`는 `0.3`이 아니다. 부동소수점 연산의 미세한 오차 때문에 `toBe(0.3)`은 실패할 수 있다. 이럴 땐 `toBeCloseTo`를 사용해 근사치를 비교해야 한다.
    {: .prompt-tip }

```typescript
it("toBeCloseTo로 부동소수점 수를 비교한다", () => {
  const result = 0.1 + 0.2;

  expect(result).not.toBe(0.3); // ❌ 실패!
  expect(result).toBeCloseTo(0.3); // ✅ 성공!
});
```

## ✅ 문자열 비교: `toContain`과 `toMatch`

* `toContain`: 특정 문자열이 포함되어 있는지 간단히 확인할 때 사용한다.
* `toMatch`: **정규표현식**을 사용하여 복잡한 패턴을 검증할 때 사용한다. 이메일 형식, 비밀번호 규칙 등을 테스트할 때 유용하다.

```typescript
it("toMatch로 정규표DOKDO식 패턴을 확인한다", () => {
  const email = "test.user123@example.com";
  
  // 이메일 형식 검증
  expect(email).toMatch(/^[\w.]+@[\w.]+\.\w+$/);
  
  // 'test' 라는 문자열 포함 여부 검증
  expect(email).toMatch(/test/);
});
```

## ✅ 배열/객체 비교: `toContain`, `toHaveLength`, `toHaveProperty`

### 배열 Matchers
* `toHaveLength(숫자)`: 배열의 길이를 확인할 때 사용한다.
* `toContain(요소)`: 배열에 특정 요소가 포함되어 있는지 확인한다.

```typescript
it("배열에서 toContain과 toHaveLength를 사용한다", () => {
  const tags = ["react", "javascript", "jest"];
  
  expect(tags).toHaveLength(3);
  expect(tags).toContain("react");
  expect(tags).not.toContain("python"); // 포함되지 않았는지 확인
});
```

### 객체 Matchers
* `toHaveProperty`: 객체가 특정 프로퍼티(키)를 가지고 있는지 확인한다. 값까지 비교할 수 있다.

```ts
it("객체에서 toHaveProperty를 사용한다", () => {
  const product = { id: 1, name: "MacBook Pro", inStock: true };

  expect(product).toHaveProperty("id"); // 'id' 키가 있는가?
  expect(product).toHaveProperty("name", "MacBook Pro"); // 'name' 키의 값이 일치하는가?
});
```

## ✅ 에러 처리 : `toThrow`
특정 함수가 의도대로 에러를 발생시키는지 검증한다.


> 에러를 발생시키는 함수는 반드시 `expect` 안에서 화살표 함수 `() =>` 로 감싸주어야 한다. 그렇지 않으면 테스트 자체가 에러로 인해 멈춰버린다.
{: .prompt-warning }

```typescript
const divideByZero = () => {
  throw new Error("0으로 나눌 수 없습니다");
};

it("함수가 에러를 발생시키는지 확인한다", () => {
  // ✅ 올바른 사용법
  expect(() => divideByZero()).toThrow();
  expect(() => divideByZero()).toThrow("0으로 나눌 수 없습니다");

  // ❌ 잘못된 사용법: 테스트가 그냥 멈춤
  // expect(divideByZero()).toThrow(); 
});
```

## ✅ 비동기 처리 : `resolves`, `rejects`

Promise를 반환하는 비동기 함수를 테스트할 때는 `resolves`와 `rejects` Matcher를 사용한다. **`expect` 앞에 `await` 키워드를 붙이는 것이 핵심이다.**

* `resolves`: Promise가 성공적으로 `resolve`될 때 사용.
* `rejects`: Promise가 `reject`될 때 사용.

```typescript
it("Promise가 성공적으로 resolve되는지 확인한다", async () => {
  // fetchUserData(1)은 성공적으로 사용자 데이터를 담은 Promise를 반환
  await expect(fetchUserData(1)).resolves.toEqual({
    id: 1,
    name: "김개발",
    email: "kim@example.com",
  });
});

it("Promise가 reject되는지 확인한다", async () => {
  // fetchUserData(999)는 에러를 담은 Promise를 반환
  await expect(fetchUserData(999)).rejects.toThrow("사용자를 찾을 수 없습니다");
});
```