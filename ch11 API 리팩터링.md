# Refactoring

## CH 11 API 리팩터링

### 11.1 조건문 분해하기(Decompose Conditional)

```javascript
// before
function getTotalOutstandingAndSendBill() {
  const result = customer.invoices.reduce(
    (total, each) => each.amount + total,
    0
  );
  sendBill();
  return result;
}

// after
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);
}

function sendBill() {
  emailGateway.send(formatBill(customer));
}
```

> 겉보기 부수효과(observable side effect)가 전혀 없이 값을 반환해주는 함수를 추구해야 한다. 이런 함수는 어느 때건 원하는 만큼 호출해도 아무 문제가 없다. 겉보기 부수효과가 있는 함수와 없는 함수는 명확히 구분짓는 것이 좋다. 이를 위한 한 가지 방법은 질의 함수(읽기 함수)는 모두 부수효과가 없어야 한다. 이를 명령-질의 분리(command-query separation)이라 한다.

#### 절차

1. 대상 함수를 복제하고 질의 목적에 충실한 이름을 짓는다.
2. 새 질의 함수에서 부수효과를 모두 제거한다.
3. 정적 검사를 수행한다.
4. 원래 함수를 호출하는 곳을 모두 찾는다. 호출하는 고세서 반환 값을 사용한다면 질의 함수를 바꾸고, 원래 함수를 호출하는 코드를 바로 아래 줄에 새로 추가한다. 하나 수정할 때마다 테스트한다.
5. 원래 함수에서 질의 관련 코드를 제거한다.
6. 테스트한다.

### 11.2 함수 매개변수화하기(Parameterize Function)

```javascript
// before
function tenPercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.05);
}

// after
function raise(aPerson, factor) {
  aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

> 두 함수의 로직이 단순히 리터럴 값만 다르다면, 다른 값만 매개변수로 받아 처리하는 함수 하나로 합쳐서 중복을 없앨 수 있다.

#### 절차

1. 비슷한 함수 중 하나를 선택한다.
2. 함수 선언 바꾸기로 리터럴들을 매개변수로 추가한다.
3. 이 함수를 호출하는 곳 모두에 적절한 리터럴 값을 추가한다.
4. 테스트한다.
5. 매개변수로 받으 값을 사용하도록 함수 본문을 수정한다. 하나 수정할 때마다 테스트한다.
6. 비슷한 다른 함수를 호출하는 코드를 찾아 매개변수화된 함수를 호출하도록 하나씩 수정한다. 하나 수정할 때마다 테스트한다.
