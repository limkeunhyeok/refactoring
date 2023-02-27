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

### 11.7 세터 제거하기(Remove Setting Method)

```javascript
// before
class Person {
  get name() { ... }
  set name(aString) { ... }
}

// after
class Person {
  get name() { ... }
}
```

> 세터 메서드가 있다는 거는 필드가 수정될 수 있다는 뜻이다. 객체 생성 후에 수정되지 않길 원하는 필드라면, 세터를 제거하여 의도를 명백히 하고 변경될 가능성을 막는다. 사람들이 무조건 접근자 메서드를 통해서만 필드를 다루거나 클라이언트에서 생성 스크립트를 사용해 객체를 생성할 때 세터를 제거한다.

#### 절차

1. 설정해야 할 값을 생성자에서 받지 않는다면 그 값을 받을 매개변수를 생성자에 추가한다. 그런 다음 생성자 안에서 적절한 세터를 호출한다.
2. 생성자 밖에서 세터를 호출하는 곳을 찾아 제거하고, 대신 새로운 생성자를 사용하도록 한다. 하나 수정할 때마다 테스트한다.
3. 세터 메서드를 인라인한다. 가능하다면 해당 필드를 불변으로 만든다.
4. 테스트한다.

### 11.8 생성자를 팩터리 함수로 바꾸기(Replace Constructor with Factory Function)

```javascript
// before
leadEngineer = new Employee(document.leadEngineer, "E");

// after
leadEngineer = createEmployee(document.leadEngineer);
```

> 객체 지향에서 제공하는 생성자는 객체를 초기화하는 특별한 용도의 함수다. 생성자에는 일반 함수에는 없는 이상한 제약이 따라 붙기도 한다.

#### 절차

1. 팩터리 함수를 만든다. 팩터리 함수의 본문에서는 원래의 생성자를 호출한다.
2. 생성자를 호출하던 코드를 팩터리 함수 호출로 바꾼다.
3. 하나씩 수정할 때마다 테스트한다.
4. 생성자의 가시 범위가 최소가 되도록 제한한다.

### 11.9 함수를 명령으로 바꾸기(Replace Function with Command)

```javascript
// before
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  // ...
}

// after
class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this._candidate = candidate;
    this._medicalExam = medicalExam;
    this._scoringGuide = scoringGuide;
  }

  execute() {
    this._reuslt = 0;
    this._healthLevel = 0;
    // ...
  }
}
```

> 함수를 그 함수만을 위한 객체 안으로 캡슐화하는 객체를 '명령 객체' 또는 단순히 '명령(command)'라고 한다. 명령은 평범한 함수 메커니즘보다 훨씬 유연하게 함수를 제어하고 표현할 수 있다. 단, 유연성은 복잡성을 키울 수 있다. 일급 함수와 명령 중 선택해야 한다면 보통은 일급 함수를 선택한다.

#### 절차

1. 대상 함수의 기능을 옮길 빈 클래스를 만든다. 클래스 이름은 함수 이름에 기초해 짓는다.
2. 방금 생성한 빈 클래스로 함수를 옮긴다.
3. 함수의 인수들 각각은 명령의 필드로 만들어 생성자를 통해 설정할지 고민해본다.

### 11.10 명령을 함수로 바꾸기(Replace Command with Function)

```javascript
// before
class ChargeCalculator {
  construnctor(customer, usage) {
    this._customer = customer;
    this._usage = usage;
  }

  execute() {
    return this._customer.rate * this._usage;
  }
}

// after
function charge(customer, usage) {
  return customer.rate * usage;
}
```

> 명령 객체는 복잡한 연산을 다룰 수 있는 강력한 메커니즘을 제공한다. 다만, 명령은 그저 함수를 하나 호출해 정해진 일을 수행하는 용도로 주로 쓰이며, 로직이 크게 복잡하지 않다면 평범한 함수로 바꿔주는 게 낫다.

#### 절차

1. 명령을 생성하는 코드와 명령의 실행 메서드를 호출하는 코드를 함께 함수로 추출한다.
2. 명령의 실행 함수가 호출하는 보조 메서드들 각각을 인라인한다.
3. 함수 선언 바꾸기를 적용하여 생성자의 매개변수 모두를 명령의 실행 메서드를 옮긴다.
4. 명령의 실행 메서드에서 참조하는 필드를 대신 대응하는 매개변수를 사용하게끔 바꾼다. 하나씩 수정할 때마다 테스트한다.
5. 생성자 호출과 명령의 실행 메서드 호출을 호출자 안으로 인라인한다.
6. 테스트한다.
7. 죽은 코드 제거하기로 명령 클래스를 없앤다.

### 11.11 수정된 값 반환하기(Return Modified Value)

```javascript
// before
let totalAscent = 0;
calculateAscent();

function calculateAscent() {
  for (let i = 0; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    totalAscent += verticalChange > 0 ? vertical : 0;
  }
}

// after
const totalAscent = calculateAscent();

function calculateAscent() {
  let result = 0;
  for (let i = 0; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    result += verticalChange > 0 ? vertical : 0;
  }
  return result;
}
```

> 데이터가 어떻게 수정되는지를 추적하는 일은 코드에서 이해하기 가장 어려운 부분 중 하나다. 데이터가 수정된다면 그 사실을 명확히 알려주어, 어느 함수가 무슨 일을 하는지 쉽게 알 수 있게 하는 일이 중요하다. 이 리팩터링은 값 하나를 계산한다는 분명한 목적이 있는 함수들에 가장 효과적이고, 여러 개를 갱신하는 함수에는 효과적이지 않다.

#### 절차

1. 함수가 수정된 값을 반환하게 하여 호출자가 그 값을 자신의 변수에 저장하게 한다.
2. 테스트한다.
3. 피호출 함수 안에 반환할 값을 가리키는 새로운 변수를 선언한다.
4. 테스트한다.
5. 계산이 선언과 동시에 이뤄지도록 통합한다.
6. 테스트한다.
7. 피호출 함수의 변수 이름을 새 역할에 어울리도록 바꿔준다.
8. 테스트한다.

### 11.12 오류 코드를 예외로 바꾸기(Replace Error Code with Exception)

```javascript
// before
if (data) {
  return new ShippingRules(data);
} else {
  return -23;
}

// after
if (data) {
  return new ShippingRules(data);
} else {
  throw new OrderProcessingError(-23);
}
```

> 예외는 프로그래밍 언어에서 제공하는 독립적인 오류 처리 메커니즘이다. 예외를 사용하면 오류 코드를 일일이 검사하거나 오류를 식별해 콜스택 위로 던지는 일을 신경 쓰지 않아도 된다. 예외는 정확히 예상 밖의 동작일 때만 쓰여야 한다.

#### 절차

1. 콜스택 상위에 해당 예외를 처리할 예외 핸들러를 작성한다.
2. 테스트한다.
3. 해당 오류 코드를 대체할 예외와 그 밖의 예외를 구분할 식별 방법을 찾는다.
4. 정적 검사를 수행한다.
5. catch절을 수정하여 직접 처리할 수 있는 예외는 적절히 대처하고 그렇지 않은 예외는 다시 던진다.
6. 테스트한다.
7. 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정한다. 하나씩 수정할 때마다 테스트한다.
8. 모두 수정했다면 그 오류 코드를 콜 스택 위로 전달하는 코드를 모두 제거한다. 하나씩 수정할 때마다 테스트한다.

### 11.13 예외를 사전확인으로 바꾸기(Replace Exception with Precheck)

```java
// before
double getValueForPeriod(int periodNumber) {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}

// after
double getValueForPeriod(int periodNumber) {
  return (periodNumber >= values.length) ? 0 : values[periodNumber];
}
```

> 예외는 뜻밖의 오류일 때만 사용해야 한다. 함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는 곳에서 조건을 검사하도록 해야 한다.

#### 절차

1. 예외를 유발하는 상황을 검사할 수 있는 조건문을 추가한다. catch 블록의 코드를 조건문의 조건절 중 하나로 옮기고, 남은 try 블록의 코드를 다른 조건절로 옮긴다.
2. catch 블록에 어서션을 추가하고 테스트한다.
3. try 문과 catch 블록을 제거한다.
4. 테스트한다.
