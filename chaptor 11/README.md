# 11. API 리팩터링

## 1. 읽기함수와 변경함수 분리하기

### 배경

- 좋은 API는 데이터를 조회만 하는 함수와 갱신하는 함수를 명확히 구분한다
- 부수효과가 없는 함수를 추구해야 한다
- 이런함수는 호출하는 함수 안 어디든 옮겨도 되며, 테스트 하기도 쉽다

### 절차

1. 대상 함수를 복제하고, 읽기 목적에 충실한 이름을 짓는다
2. 새 읽기함수에서 부수효과(변경 등)을 모두 제거한다
3. 정적검사 수행
4. 기존 함수 사용되는 곳을 모두 확인, 반환값을 사용한다면 새로운 읽기함수로 변경, 원래 함수 호출 코드 아래에 새로운 읽기함수 추가
5. 기존 함수에서 읽기관련 코드 제거
6. 테스트

### 예시

```jsx
// BEFORE
function getTotalOutstandingAndSendBill() {
    const result = customer.invoices.reduce((total, each) => each.amount + total, 0)
    sendBill();
    return result;
}
```

```jsx
// AFTER
function getTotalOutstanding() {
    return result = customer.invoices.reduce((total, each) => each.amount + total, 0)
}

function sendBill() {
    emailGateway.send(formatBill(customer));
}
```

## 2. 함수 매개변수화 하기

### 배경

- 값 하나 때문에 여러개로 나뉜 함수는 하나로 합칠 수 있다
- 이렇게 하면 매개변수 값만 바꿔서 여러곳에서 쓸수 있어, 함수의 유용성이 커진다

### 절차

1. 비슷한 함수중 하나를 선택한다
2. 함수 선언 바꾸기로 리터럴들을 매개변수로 추가한다
3. 이 함수를 호출하는 곳 모두에 적절한 리터럴 값을 추가한다
4. 테스트한다
5. 매개변수로 받은 값을 사용하도록 함수 본문을 수정한다. 하나 수정할 때마다 테스트 한다
6. 비슷한 다른 함수를 호출하는 코드를 찾아 매개변수화된 함수를 호출하도록 하나씩 수정한다. 하나 수정할 때마다 테스트한다

### 예시

```jsx
// BEFORE
function tenPercentRaise(aPerson) {
    aPerson.salary = aPerson.salary.multiply(1.1);
}

function fivePercentRaise(aPerson) {
    aPerson.salary = aPerson.salary.multiply(1.05);
}
```

```jsx
// AFTER
function raise(aPerson, factor) {
    aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

## 3. 플래그 인수 제거하기

### 배경

- 어떤 매개변수는 그저 함수의 동작 모드를 전환하는 용도로만 쓰이는데, 이럴때는 플래그 인수를 제거하면 좋다
- 플래그 인수가 있으면 호출할 수 있는 함수들이 무엇이고, 어떻게 호출해야 하는지 이해하기 어려워진다
- 또한 함수들의 기능차이가 잘 드러나지 않는다

### 절차

1. 매개변수로 주어질 수 있는 값 각각에 대응하는 명시적 함수들을 생성한다
2. 원래 함수를 호출하는 코드들을 모두 찾아서 각 리터럴 값에 대응되는 명시적 함수를 호출하도록 수정한다

### 예시

```jsx
// BEFORE
function setDimension(name, value) {
    if(name == "height") {
        this._height = value;
        return;
    }
    if(name == "width") {
        this._width = value;
        return;
    }
}
```

```jsx
// AFTER
function setHeight(value) {this._height = value;}
function setWidth(value) {this._width = value;}
```

 

## 4. 객체 통째로 넘기기


### 배경

 어떤 객체로부터 값 몇개를 얻은 후 그 값들만으로 무언가를 하는 로직이 있다면, 그 로직을 객체 안으로 집어넣어야 하는 신호로 알아야 한다.

### 절차

 1. 매개변수들을 원하는 형태로 받는 빈 함수를 만든다

1. 새 함수의 본문에서는 원래 함수를 호출하도록 하며, 새 매개변수와 우너래 함수의 매개변수를 매핑한다.
2. 정적 검사
3. 모든 호출자가 새 함수를 사용하게 수정
4. 원래 함수를 인라인
5. 새 함수 이름 수정 후 모든 호출자에 반영

### 예시

```jsx
const low = aRoom.daysTempRange.low; //해당 부분 필요 없어짐
const high = aRoom.daysTempRange.high; // 해당 부분 필요없어짐
if(!aPlan.xxNEWwighinRagne(aRoom.daysTempRange))
	alert.push('이 방온도가 지정 범위를 벗어났습니다')

xxNEWwithinRange(aNumberRange){
	return this.withinRange(aNumberRange.low, aNumberRange.high)
}

xxNEWwithinRange(aNumberRange){
	return(aNumberRange.low >= this._temperatureRange.low) &&
				(aNumberRange.high <=this._temperatureRange.high);
}

if(!aPlan.withinRange(aRoom.daysTempRange))
	alert.push('이 방온도가 지정 범위를 벗어났습니다')
```

### 새 함수를 다른 방식으로 만들기

```jsx
const low = aRoom.daysTempRange.low; 
const high = aRoom.daysTempRange.high
const isWithinRange = xxNEWwithinRange(aPlan, tempRange)
if(!isWithinRange) alert.push('방 온도가 지정 범위를 벗어났습니다.')

function xxNEWwithinRange(aPlan, tempRange){
	const low = tempRange.log;
	const high = tempRange.high;
	const isWithinRange = aPlan.withinRange(low, high)
	return isWithinRange;
}
```

## 5. 매개변수를 질의함수로 바꾸기

```jsx
availableVacation(anEmployee, anEmployee.gragde)

//변경후
availableVation(employee)

function availableVacation(anEmployee){
	const grade = anEmployee.grade;

```

### 절차

1. 대상 매개변수값을 계산하는 코드를 별도 함수로 추출해놓는다
2. 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 그 매개변수의 값을 만들어주는 표현식을 참조하도록 바꾼다.
3. 함수 선언 바꾸기로 대상 매개변수를 없앤다.

## 6. 질의 함수를 매개변수로 바꾸기

### 배경

똑같은 값을 건네면 매번 똑같은 결과를 내는 함수 ‘참조 투명성’

### 절차

1. 변수 추출하기로 질의 코드를 함수본문의 나머지 코드와 분리한다
2. 함수 본문 중 해당 질의를 호출하지 않는 코드들을 별도 함수로 추출한다.
3. 방금 만든 변수를 인라인하여 제거한다.
4. 원래 함수도 인라인 한다
5. 새 함수의 이름을 원래 함수의 이름으로 고쳐준다.

```jsx
get targetTemperature(){
	const selectedTemperature = thermostat.selectedTemperature;
	if(selectedTemperature > this._max) return this._max;
	else if(selectedTemeperature < this._min) return this._min;
	else return selectedTemperature;

//변경후
get targetTemperature(){
		return this.xxNEWtargetTemperature(thermostat.selectedTemperature;);
}

xxNEWtargetTemperature(selectedTemperature){
	if(selectedTemperature > this._max) return this._max;
	else if(selectedTemeperature < this._min) return this._min;
	else return selectedTemperature;
}

//selectedTemperature 를 xxNEWtargetTemperature(thermostat.selectedTemperature)
//로 변경하기
```

## 7.세터 제거하기

```jsx
class Person 
{
	get name() {...}
	set name(aString) {...}
}
```

🔽

```jsx
class Person 
{
	get name() {...}
}
```

### 절차

1. 설정해야 할 값을 생성자에서 받지 않는다면 그 값을 받을 매개변수를 생성자에 추가한다.
그런 다음 생성자 안에 적절한 세터를 호출한다.
2. 생성자 밖에서 세터를 호출하는 곳을 찾아 제거하고, 대신 새로운 생성자를 사용하도록 한다. 하나 수정할 때마다 테스트한다.
3. 세터 메서드를 인라인한다. 가능하면 필드를 불변으로 만든다.
4. 테스트한다.

## 8.생성자를 팩터리 함수로 바꾸기

```jsx
leadEngineer = new Employee(document.leadEngineer, 'E');
```

🔽

```jsx
leadEngineer = createEngineer(document.leadEngineer);
```

### 절차

1. 팩터리 함수를 만든다. 팩터리 함수의 본문에서는 원래 생성자를 호출한다.
2. 생성자를 호출한던 코드를 팩터리 함수로 바꾼다.
3. 하나씩 수정할 때마다 테스트한다.
4. 생성자의 가시 범위가 되도록 제한한다.

## 9.함수를 명령으로 바꾸기

```jsx
function score(candidatem, medicalExam, socringGuide)
{
	let result = 0;
	let healthLevel = 0;
	// 긴 코드 생략
}
```

🔽

```jsx
class Scorer
{
	this._candidatem = candidatem;
	this._medicalExam = medicalExam;
	this._socringGuide ******=****** socringGuide;
}

execute()
{
	this._result  = result;
	this._healthLevel = healthLevel;
	// 긴 코드 생략
}
```

### 배경

### 절차

1. 대상 함수의 기능을 옮길 빈 클래스를 만든다. (클래스 이름은 함수 이름에 기초해 짓는다)
2. 방금 생성한 빈 클래스로 함수를 옮긴다.
3. 함수의 인수들 각각은 명령의 필드로 만들어 생성자를 통해 설정할 지 고민해본다.

---

## 10. 명령을 함수로 바꾸기

```jsx
class ChargeCalculator {
  constructor(customer, usage) {
    this._customer = customer;
    this._usage = usage;
  }

  execute() {
    return this._customer.rate * this._usage;
  }
}
```

🔽

```jsx
function charge(customer, usage) {
  return customer.rate * usage;
}
```

### 배경

명령 객체는 복잡한 연산을 다룰 수 있는 강력한 메커니즘을 제공한다. 하지만 명령의 이런 능력은 공짜가 아니다. 그저 함수를 하나 호출해 정해진 일을 수행하는 용도로만 쓰이고 로직이 크게 복잡하지 않다면 명령 객체는 장점보다 단점이 크니 평범한 함수로 바꿔주는 게 낫다.

### 절차

1. 명령을 생성하는 코드와 명령의 실행 메서드를 호출하는 코드를 함께 함수로 추출(6.1절)한다.
→ 이 함수가 바로 명령을 대체할 함수다.
2. 명령의 실행 함수가 호출하는 보조 메서드들 각각을 인라인(6.2절)한다.
→ 보조 메서드가 값을 반환한다면 함수 인라인에 앞서 변수 추출하기(6.3절)를 적용한다.
3. 함수 선언 바꾸기(6.5절)를 적용하여 생성자의 매개변수 모두를 명령의 실행 메서드로 옮긴다.
4. 명령의 실행 메서드에서 참조하는 필드들 대신 대응하는 매개변수를 사용하게끔 바꾼다. 하나씩 수정할 때마다 테스트한다.
5. 생성자 호출과 명령의 실행 메서드 호출을 호출자(대체 함수) 안으로 인라인한다.
6. 테스트한다.
7. 죽은 코드 제거하기(8.9절)로 명령 클래스를 없앤다.

---

## 11. 수정된 값 반환하기

```jsx
let totalAscent = 0;
calculateAscent();

function calculateAscent() {
  for (let i = 1; i < points.length; i += 1) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
  }
}
```

🔽

```jsx
const totalAscent = calculateAscent();

function calculateAscent() {
  let result = 0;
  for (let i = 1; i < points.length; i += 1) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    result += (verticalChange > 0) ? verticalChange : 0;
  }
  return result;
}
```

### 배경

데이터가 수정된다면 그 사실을 명확히 알려주어서, 어느 함수가 무슨 일을 하는지 쉽게 알 수 있게 하는 일이 대단히 중요하다.

데이터가 수정됨을 알려주는 좋은 방법은 변수를 갱신하는 함수라면 수정된 값을 반환하여 호출자가 그 값을 변수에 담아두도록 하는 것이다.

이 리팩터링은 값 하나를 계산한다는 분명한 목적이 있는 함수들에 가장 효과적이고, 반대로 값 여러 개를 갱신하는 함수에는 효과적이지 않다. 한편, 함수 옮기기(8.1절)의 준비 작업으로 적용하기 좋은 리팩터링이다.

### 절차

1. 함수가 수정된 값을 반환하게 하여 호출자가 그 값을 자신의 변수에 저장하게 한다.
2. 테스트한다.
3. 피호출 함수 안에 반환할 값을 가리키는 새로운 변수를 선언한다.
→ 이 작업이 의도대로 이뤄졌는지 검사하고 싶다면 호출자에서 초깃값을 수정해보자. 제대로 처리했다면 수정된 값이 무시된다.
4. 테스트한다.
5. 계산이 선언과 동시에 이뤄지도록 통합한다(즉, 선언 시점에 계산 로직을 바로 실행해 대입한다).
→ 프로그래밍 언어에서 지원한다면 이 변수를 불변으로 지정하자.
6. 테스트한다.
7. 피호출 함수의 변수 이름을 새 역할에 어울리도록 바꿔준다.
8. 테스트한다.

---

## 12. 오류 코드를 예외로 바꾸기

```jsx
if (data) {
  return new ShippingRules(data);
}
return -23;
```

🔽

```jsx
if (data) {
  return new ShippingRules(data);
}
throw new OrderProcessingError(-23);
```

### 배경

예외에는 독자적인 흐름이 있어서 프로그램의 나머지에서는 오류 발생에 따른 복잡한 상황에 대처하는 코드를 작성하거나 읽을 일이 없게 해준다.

예외는 정확히 예상 밖의 동작일 때만 쓰여야 한다. 달리 말하면 프로그램의 정상 동작 범주에 들지 않는 오류를 나타낼 때만 쓰여아 한다. 괜찮은 경험 법칙이 하나 있다. 예외를 던지는 코드를 프로그램 종료 코드로 바꿔도 프로그램이 여전히 정상 동작할지를 따져보는 것이다. 정상 동작하지 않을 것 같다면 예외를 사용하지 말라는 신호다. 예외 대신 오류를 검출하여 프로그램을 정상 흐름으로 되돌리게끔 처리해야 한다.

### 절차

1. 콜스택 상위에 해당 예외를 처리할 예외 핸들러를 작성한다.
→ 이 핸들러는 처음에는 모든 예외를 다시 던지게 해둔다.
→ 적절한 처리를 해주는 핸들러가 이미 있다면 지금의 콜스택도 처리할 수 있도록 확장한다.
2. 테스트한다.
3. 해당 오류 코드를 대체할 예외와 그 밖의 예외를 구분할 식별 방법을 찾는다.
→ 사용하는 프로그래밍 언어에 맞게 선택하면 된다. 대부분 언어에서는 서브클래스를 사용하면 될 것이다.
4. 정적 검사를 수행한다.
5. `catch` 절을 수정하여 직접 처리할 수 있는 예외는 적절히 대처하고 그렇지 않은 예외는 다시 던진다.
6. 테스트한다.
7. 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정한다. 하나씩 수정할 때마다 테스트한다.
8. 모두 수정했다면 그 오류 코드를 콜스택 위로 전달하는 코드를 모두 제거한다. 하나씩 수정할 때마다 테스트한다.
→ 먼저 오류 코드를 검사하는 부분을 함정(trap)으로 바꾼 다음, 함정에 걸려들지 않는지 테스트한 후 제거하는 전략을 권한다. 함정에 걸려드는 곳이 있다면 오류 코드를 검사하는 코드가 아직 남아 있다는 뜻이다. 함정을 무사히 피했다면 안심하고 본문을 정리하자(죽은 코드 제거하기(8.9절)).

---

## 13. 예외를 사전확인으로 바꾸기

```jsx
function getLocalStorage() {
  try {
    return window.localStorage;
  } catch (err) {
    if (err instanceof ReferenceError) {
      return null;
    }
  }
}
```

🔽

```jsx
function getLocalStorage() {
  return (typeof window === 'undefined') ? null : window.localStorage;
}
```

### 배경

예외는 ‘뜻밖의 오류’라는, 말 그대로 예외적으로 동작할 때만 쓰여야 한다. 함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는 곳에서 조건을 검사하도록 해야 한다.

### 절차

1. 예외를 유발하는 상황을 검사할 수 있는 조건문을 추가한다. `catch` 블록의 코드를 조건문의 조건절 중 하나로 옮기고, 남은 `try` 블록의 코드를 다른 조건절로 옮긴다.
2. `catch` 블록에 어서션을 추가하고 테스트한다.
3. `try` 문과 `catch` 블록을 제거한다.
4. 테스트한다.