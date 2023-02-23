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

### 11.3 플래그 인수 제거하기(Remove Flag Argument)

```javascript
// before
function setDimension(name, value) {
  if (name === "height") {
    this._height = value;
    return;
  }
  if (name === "width") {
    this._width = value;
    return;
  }
}

// after
function setHeight(value) {
  this._height = value;
}
function setWidth(value) {
  this._width = value;
}
```

> 플래그 인수란 호출되는 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 인수다. 플래그 인수가 있으면 함수들의 기능 차이가 잘 드러나지 않는다. 플래그 인수가 둘 이상이면 함수 하나가 너무 많은 일을 처리하고 있다는 신호이다.

#### 절차

1. 매개 변수로 주어질 수 있는 값 각각에 대응하는 명시적 함수들을 생성한다.
2. 원래 함수를 호출하는 코드들을 모두 찾아서 각 리터럴 값에 대응되는 명시적 함수를 호출하도록 수정한다.

### 11.4 객체 통째로 넘기기(Preserve Whole Object)

```javascript
// before
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.withinRange(low, high))

// after
if (aPlan.withinRange(aRoom.daysTempRange))
```

> 레코드를 통째로 넘기면 변화에 대응하기 쉽다. 또한 매개변수 목록이 짧아져서 일반적으로는 함수 사용법을 이해하기 쉬워진다. 다만 함수가 레코드 자체에 의존하기를 원치 않을 때는 이 리팩터링을 수행하지 않는데, 레코드와 함수가 서로 다른 모듈에 속한 상황이라면 특히 더 그렇다.

#### 절차

1. 매개변수들을 원하는 형태로 받는 빈 함수를 만든다.
2. 새 함수의 본문에서는 원래 함수를 호출하도록 하며, 새 매개변수와 원래 함수의 매개변수를 매핑한다.
3. 정적 검사를 수행한다.
4. 모든 호출자가 새 함수를 사용하게 수정한다. 하나씩 수정하며 테스트한다.
5. 호출자를 모두 수정했다면 원래 함수를 인라인한다.
6. 새 함수의 이름을 적절히 수정하고 모든 호출자에 반영한다.

### 11.5 매개변수를 질의 함수로 바꾸기

```javascript
// before
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
  ...
};

// after
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee) {
  const grade = anEmployee.grade;

  ...
};
```

> 매개변수 목록은 함수의 동작에 변화를 줄 수 있는 일차적인 수단이다. 피호출 함수가 쉽게 결정할 수 있는 값을 매개변수로 건네는 것도 일종의 중복이다. 제거하려는 매개변수의 값을 다른 매개변수에 질의해서 얻을 수 있다면 안심하고 질의 함수로 바꿀 수 있다. 단, 대상 함수가 참조 투명(referential transparency)해야 한다. 참조 투명이란 함수에 똑같은 값을 건네 호출하면 항상 똑같이 동작한다는 것을 의미한다.

#### 절차

1. 필요하다면 대상 매개변수의 값을 계산하는 코드를 별도 함수로 추출해 놓는다.
2. 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 그 매개변수의 값을 만들어주는 표현식을 참조하도록 바꾼다. 하나 수정할 때마다 테스트한다.
3. 함수 선언 바꾸기로 대상 매개변수를 없앤다.

### 11.6 질의 함수를 매개변수로 바꾸기(Replace Query with Parameter)

```javascript
// before
targetTemperature(aPlan);

function targetTemperature(aPlan) {
  currentTemperature = thermostat.currentTemperature;
  ...
}

// after
targetTemperature(aPlan, thermostat.currentTemperature);

function targetTemperature(aPlan, currentTemperature) {
  ...
}
```

> 전역 변수를 참조한다거나 제거하길 원하는 원소를 참조하는 경우, 해당 참조를 매개변수로 바꿔 해결할 수 있다. 똑같은 값을 건네면 매번 똑같은 결과를 내는 함수는 다루기 쉽다. 모듈을 참조 투명하게 만들어 얻는 장점은 대체로 아주 크다. 이에 모듈을 개발할 때 순수 함수들을 따로 구분하고, 프로그램의 입출력과 기타 가변 원소들을 다루는 로직으로 순수 함수들의 겉을 감싸는 패턴을 많이 활용한다.

#### 절차

1. 변수 추출하기로 질의 코드를 함수 본문의 나머지 코드와 분리한다.
2. 함수 본문 중 해당 질의를 호출하지 않는 코드들을 별도 함수로 추출한다.
3. 방금 만든 변수를 인라인하여 제거한다.
4. 원래 함수도 인라인한다.
5. 새 함수의 이름을 원래 함수의 이름으로 고쳐준다.
