# 6. 기본적인 리팩토링

## 1. 함수 추출하기

- 코드 조각을 찾아 무슨일을 하는지 파악한 다음, 독립된 함수로 추출하고 목적에 맞는 이름을 붙인다.
- 언제 독립된 함수로 나눠야 하나 ? → `목적과 구현을 분리` 코드를 보고 무슨 일을 하는지 파악하는데 시간이 걸린다면 그 부분을 함수로 추출한뒤 `하는일` 에 맞는 이름을 짓는다.
- 함수의 이름을 잘 지어야 한다.
    1. 함수를 새로 만들고 목적을 잘 드러내는 이름을 붙인다. (`무엇을` 하는지가 드러나야 한다 )
        
        → 대상 코드가 함수 호출문 처럼 간단하더라도 함수로 뽑아서 목적이 더 잘 드러나는 이름을 붙일 수 있다면 추출한다. 이런 이름이 떠오르지 않는다면 함수로 추출하면 안 된다는 신호다.
        
    2. 추출할 코드를 원 본 함수에서 복사하여 새 함수에 붙여넣는다.
    3. 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사한다. 있다면 매개변수로 전달한다.
    4. 변수를 다 처리했다면 컴파일 한다.
    5. 원본 함수에서 추출한 코드 부분을  새로 만든 함수를 호출하는 문장으로 바꾼다
    6. 테스트
    7. 다른코드에 방금 추출한 것과 똑같거나 비슷한 코드가 없는지 살핀다. 있다면 방금 추출한 새 함수를 호출하도록 바꿀지 검토한다.
    
## 2. 함수 인라인하기
    
- 리펙터링 과정에서 잘 못 추출된 함수들도 다시 인라인한다.
- 절차
    1. 다형 메서드 인지 확인한다
    2. 인라인할 함수를 호출하는 곳을 모두 찾는다
    3. 각 호출문을 함수 본문으로 교체한다
    4. 하나씩 교체할때마다 테스트한다.
    5. 함수 정의(원래함수) 를 삭제한다.
    
    더 복잡한 경우가 있을 수 있다. 다 설명하지 않는 이유는 상황이 그정도로 복잡하다면 함수 인라인하기를 적용하면 안된다.

## 3. 변수 추출하기

```jsx
return order.quantity * order.itmePrice -
	Math.max(0, order.quantity - 500) * order.itmePrice * 0.05 +
	Math.min(order.quantity * order.itmePrice * 0.1, 100);

const basePrice = order.quantity * order.itmePrice;
const quantityDiscount = Math.max(0, order.quantity - 500)
													* order.itmePrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

### a. 배경

 표현식이 너무 복잡해서 이해하기 어려울 수 있는데 이럴 때 지역 `변수`를 활용하면 표현식을 잘게 쪼개 관리하기 쉽게 만들 수 있다. 그러면 복잡한 로직을 구성하는 단계마다 이름을 붙일 수 있기 떄문에 `코드의 목적`을 훨씬 명확하게 드러낼 수 있다. 

 이 과정에서 추가된 `변수`는 디버거에 중단점을 설정하거나 상태를 출력하는 문장을 추가하는 등 디버깅에도 도움을 준다.

 현재 함수 안에서만 의미가 있다면 `변수`로 추출하는 것이 좋지만 함수를 넘어서 넓은 문맥에서까지 의미가 된다면 그 넓은 범위에서 통용되는 이름을 생각해야 하는데 즉, 주로 `변수가 아닌 함수`로 추출하는 것을 말한다. 이름이 통용되는 문맥을 넓히면 다른 코드에서 사용할 수 있기 때문에 같은 표현식을 중복해서 작성하지 않아도 되며 결과적으로 중복의 확률이 적으면서 의도가 잘 드러나는 코드를 작성 수 있다.

### b. 절차

1. 추출하려는 표현식에 부작용은 없는지 확인한다.
2. 불변 변수 하나를 선언하고 이름을 붙일 표현식의 복제본을 대입한다.
3. 원본 표현식을 새로 만든 변수로 교체한다.
4. 테스트한다.
5. 표현식을 여러 곳에서 사용한다면 각각을 새로 만든 변수로 교체한다. 하나 교체할 때마다 테스트한다.

### c. 참고 리펙터링

**반대 리펙터링**

`**변수 인라인하기**`



## 4. 변수 인라인하기

```jsx
function before() {
	let basePrcie = anOrder.basePrice;
	return basePrice > 1000;
}

function after() {
	return anOrder.basePrice > 1000;
}
```

### a. 배경

 `변수`는 함수 안에서 표현식을 가리키는 이름으로 쓰이고 대체로 긍정적인 효과를 준다.

 하지만 그 이름이 표현식과 다를 바 없는 경우도 있고, 변수가 주변 코드를 리펙터링하는 데 방해가 되기도 하는데 이럴 때 그 `변수`를 인라인하는 것이 좋다.

### b. 절차

1. 대입문의 우변(표현식)에서 부작용이 생기지는 않는지 확인한다.
2. 변수가 불변으로 선언되지 않았다면 불변으로 만든 후 테스트한다.
3. 이 변수를 가장 처음 사용하는 코드를 찾아서 대입문 우변의 코드로 바꾼다
4. 테스트한다.
5. 변수를 사용하는 부분을 모두 교체할 때까지 이 과정을 반복한다.
6. 변수 선언문과 대입문을 지운다.
7. 테스트한다.

### c. 참고 리펙터링

**반대 리펙터링**

`**변수 추출하기**`

## 5. 함수 선언 바꾸기

```jsx
function circum(radius) {
	...
}

function circumference(radius) {
	...
}
```

### a. 배경

 `함수`는 프로그램을 작은 부분으로 나누는 주된 수단이다. 이 `함수`를 잘 정의하면 새로운 기능의 추가가 쉬워지는 반면 잘못 정의한다면 지속적인 방해 요인으로써 작용하며 구조를 어렵게 하기도 한다. 

 그렇기에 `함수`에서 가장 중요한 요소는 `함수의 이름`이라고 할 수 있다. 이름이 좋으면 코드를 살펴 볼 필요 없이 호출문만 보고도 무슨 일을 하는지 파악할 수 있다. 

### b. 절차

간단한 절차

1. 매개변수를 제거하려거든 먼저 함수 본문에서 제거 대상 매개변수를 참조하는 곳은 없는지 확인한다.
2. 매서드 선언을 원하는 형태로 바꾼다.
3. 기존 메서드 선언을 참조하는 부분을 모두 찾아서 바뀐 형태로 수정한다.
4. 테스트한다.

마이그레이션 절차

1. 이어지는 추출 단계를 수월하게 만들어야한다면 함수의 본문을 적절히 리펙터링한다.
2. 함수 본문을 새로운 함수로 추출한다.
3. 추출한 함수에 매개변수를 추가해야 한다며냐 ‘간단한 절차’를 따라 추가한다.
4. 테스트한다.
5. 기존 함수를 인라인한다.
6. 이름을 임시로 붙여뒀다면 함수 선언 바꾸기를 한 번 더 적용해서 원래 이름으로 되돌린다.
7. 테스트한다.

### c. 참고 리펙터링

**다른 이름**

`**함수 이름 바꾸기**`

`**시그니처 바꾸기**`

## 6. 변수 캡슐화 하기

<aside>
💡 데이터의 사용범위가 넓을수록 적절히 캡슐화하는것이 좋다

</aside>

## 배경

변수는 유효범위가 넓어질 수록 다루기가 어려워진다. 전역 데이터가 골칫거리인 이유도 바로 여기에 있다. 그래서 접근할 수 있는 범위가 넓은 데이터는 데이터로의 접근을 독접하는 함수를 만드는 식으로 캡슐화 하는것이 가장 좋은 방법일 때가 많다.

## 절차

1. 변수로의 접근과 갱신을 전담하는 캡슐화 함수들을 만든다.
2. 정적 검사를 수행한다
3. 변수를 직접참조하던 부분을 모두 캡슐화 함수 호출로 바꾼다.
4. 변수의 접근 범위를 제한한다(변수의 이름을 바꿔서 해당 변수를 참조하는 곳을 쉽게 찾을 수 있다)
5. 테스트한다
6. 변경값이 레코드라면 레코드 캡슐화하기를 적용할지 고려해본다

## 예시

### BEFORE

```jsx
let defaultOwner = {fistName: "마틴", lastName: "파울러"}
spaceship.owner = defaultOwner;
```

### AFTER

```jsx
let defaultOwner = {fistName: "마틴", lastName: "파울러"}

// getter
spaceship.owner = getDefaultOwner();
// setter
setDefaultOwner({fistName: "병관", lastName: "김"});

// getter가 새로운 인스턴스를 반환하도록 하여, 수정하더라도 원본에 영향이 없도록 한다
export function getDefaultWoner() {return new Person(defaultOwner)}
export function setDefaultOwner(arg) {defaultOwner = arg;}

class Person {
	constructor(data) {
		this._lastName = data.lastName;
		this._firstName = data.firstName;
	}
	
	get lastName() {return this._lastName;}
	get firstName() {return this._firstName;}

}
```

## 7. 변수 이름 바꾸기

## 배경

변수는 프로그래머가 하려는 일에 관해 많은것을 설명해준다.

## 절차

1. 폭넓게 쓰이는 변수라면 변수 캡술화하기를 고려한다
2. 이름을 바꿀 변수는 참조하는 곳을 모두 찾아서, 하나씩 변경한다
3. 테스트한다

## 예시

### BEFORE

```jsx
let tpHd = "untitled";
tpHd = obj['articleTitle'];
result += `<h1>${tpHd}</h1>`
```

### AFTER

```jsx
function title() {return tpHd;}
function setTitle(arg) {tpHd = arg;}

let tpHd = "untitled";
result += `<h1>${tpHd}</h1>`
```

### BEFORE

```jsx
const cpyNm = "애크미 구스베리";
```

### AFTER

```jsx
// 점진적으로 바꾸는 방법

const companyName = "애크미 구스베리";
const cpyNm = companyName;
```

## 8. 매개변수 객체 만들기

## 배경

데이터 뭉치를 구조로 묶으면 데이터 사이의 관계가 명확해진다. 게다가 묶을 데이터 구조를 이용하면 매개변수의 수가 줄어들게 된다. 이 과정에서 데이터 구조가 문제영역을 훨씬 간결하게 표현하는 새로운 추상개념으로 격상되면서, 코드의 개념적인 그림을 다시 그릴수도 있다. 

## 절차

1. 적당한 데이터 구조가 아직 없다면 새로만든다
2. 테스트 한다
3. 함수선언 바꾸기로 새 데이터 구조를 매개변수로 추가한다
4. 테스트한다
5. 함수 호출 시 새로운 데이터구조의 인스턴스를 넘기도록 수정한다. 하나씩 수정할 때마다 테스트한다
6. 기존 매개변수를 사용하던 코드를 새 데이터 구조의 원소를 사용하도록 변경한다
7. 다 바꿨다면 기존 매개변수를 제거하고 테스트한다.

## 예시

### BEFORE

```jsx
function readingsOutsideRange(station, min, max) {
    return station.readings.filter(r => r.temp < min || r.temp > max);
}
```

### AFTER

```jsx

const range = new NumberRange(operatingPlan.temperatureFloor, operatingPlan.temperatureCeiling)
function readingsOutsideRange(station, range) {
    return station.readings.filter(r => !range.contains(r.temp));
}

class NumberRange {
    constructor(min, max) {
        this._data = {min: min, max: max};
    }

    get min() {return this._data.min};
    get max() {return this._data.max};
    // 단순히 매개변수를 모드는데 그치지 않고, 관련 로직을 클래스내에 구현
    contains(arg) {return (arg >= this.min && arg <= this.max)}
}
```

## 9. 여러 함수를 클래스로 묶기

## 배경

여러 함수를 클래스로 묶으면 이 함수들이 공유하는 공통 환경을 더 명확하게 표현할 수 있고, 각 함수에 전달되는 인수를 줄여서 객체 안에서의 함수 호출을 간결하게 만들 수 있다. 또한 이런 객체를 시스템의 다른 부분에 전달하기 뒤한 참조를 제공할 수 있다.

## 절차

1. 함수들이 공유하는 공통데이터 레코드르르 캡슐화한다
2. 공통 레코드를 사용하는 함수 각각을 새 클래스로 옮긴다.
3. 데이터를 조작하는 로직들은 함수로 추출해서 새 클래스로 옮긴다

## 예시

### BEFORE

```jsx
let reading = {customer: "ivan", quantity: 10, month: 5, year: 2017};

// Client 1
const aReading = acquireReading();
const baseCharge = baseRate(aReading.month, aReading.year) * aReading.quantity;

// Client 2
const aReading = acquireReading();
const base = baseRate(aReading.month, aReading.year) * aReading.quantity;
const taxableCharge = Math.max(0, base - taxThreshold(aReading.year));

// Client 3
const aReading = acquireReading();
const basicChargeAmount = calculateBaseCharge(aReading);

function calculateBaseCharge(aReading) {
    return baseRate(aReading.month, aReading.year) * aReading.quantity;
}
```

### AFTER

```jsx
class Reading {
    constructor(data) {
        this._customer = data.customer;
        this._quantity = data.quantity;
        this._month = data.month;
        this._year = data.year;
    }
    get customer() {return this._customer;}
    get quantity() {return this._quantity;}
    get month() {return this._month;}
    get year() {return this._year;}

    get baseCharge() {
        return baseRate(this.month, this.year) * this.quantity;
    }

    get taxableCharge() {
        return Math.max(0, this.baseCharge - taxThreshold(this.year));
    }
}

// Client 1
const rawReading = acquireReading();
const aReading = new Reading(acquireReading);
const baseCharge = aReading.baseCharge;

// Client 2
const rawReading = acquireReading();
const aReading = new Reading(acquireReading);
const taxableCharge = aReading.taxableCharge;

// Client 3
const rawReading = acquireReading();
const aReading = new Reading(acquireReading);
const basicChargeAmount = aReading.abaseCharge;
```

---

## 10. 여러 함수를 변환 함수로 묶기

```jsx
function base(aReading) { ... }
function taxableCharge(aReading) { ... }
```

🔽

```jsx
function enrichReading(argReading) {
  const aReading = _.cloneDeep(argReading);
  aReading.baseCharge = base(aReading);
  aReading.taxableCharge = taxableCharge(aReading);
  return aReading
}
```

## 배경

데이터를 입력받아서 여러 가지 정보를 도출한 후 이 정보는 여러 곳에서 사용될 수 있는데, 이 정보가 사용되는 곳마다 같은 도출 로직이 반복되기도 한다. 이런 도출 작업들을 한데로 모아두면 검색과 갱신을 일관된 장소에서 처리할 수 있고 로직 중복도 막을 수 있다.

위 같은 경우 이 리팩터링 대신 여러 함수를 클래스로 묶기(6.9절)로 처리해도 된다. 대체로 소프트웨어에 이미 반영된 프로그래밍 스타일을 따르면 되지만 둘 사이에 중요한 차이가 하나 있다. 원본 데이터가 코드 안에서 갱신될 때는 클래스로 묶는 편이 훨씬 낫다. 변환 함수로 묶으면 가공한 데이터를 새로운 레코드에 저장하므로, 원본 데이터가 수정되면 일관성이 깨질 수도 있기 때문이다.

## 절차

1. 변환할 레코드를 입력받아 값을 그대로 반환하는 변환 함수로 만든다.
→ 이 작업은 대체로 깊은 복사로 처리해야 한다. 변환 함수가 원본 레코드를 바꾸지 않는지 검사하는 테스트를 마련해두면 도움될 때가 많다.
2. 묶을 함수 중 함수 하나를 골라서 본문 코드를 변환 함수로 옮기고, 처리 결과를 레코드에 새 필드로 기록한다. 그런 다음 클라이언트 코드가 이 필드를 사용하도록 수정한다.
→ 로직이 복잡하면 함수 추출하기(6.1절)부터 한다.
3. 테스트한다.
4. 나머지 관련 함수도 위 과정에 따라 처리한다.

---

## 11. 단계 쪼개기

```jsx
const orderData = orderString.split(/s+/);
const productPrice = priceList[orderData[0].split('-')[1]];
const orderPrice = parseInt(orderData[1], 10) * productPrice;
```

🔽

```jsx
const orderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString) {
  const values = aString.split(/\s+/);
  return {
    productID: values[0].split('-')[1],
    quantity: parseInt(values[1], 10),
  };
}

function price(order, priceList) {
  return order.quantity * priceList[order.productID];
}
```

## 배경

서로 다른 두 대상을 한번에 다루는 코드를 발견하면 각각을 별개 모듈로 나누는 방법을 찾는다. 코드를 수정해야 할 때 하나에만 집중하기 위해서다. 모듈이 잘 분리돼 있다면 다른 모듈의 상세 내용은 전혀 몰라도 원하는 대로 수정을 끝마칠 수도 있다.

이렇게 단계를 쪼개는 기법은 주로 덩치 큰 소프트웨어에 적용된다. 하지만 규모에 관계없이 여러 단계로 분리하면 좋을만한 코드를 발견할 때마다 기본적인 단계 쪼개기 리팩터링을 한다면 코드가 훨씬 깔끔해진다. 다른 단계로 볼 수 있는 코드 영역들이 마침 서로 다른 데이터와 함수를 사용한다면 단계 쪼개기에 적합하다는 뜻이다. 이 코드 영역들을 별도 모듈로 분리하면 그 차이를 코드에서 훨씬 분명하게 드러낼 수 있다.

## 절차

1. 두 번째 단계에 해당하는 코드를 독립 함수로 추출한다.
2. 테스트한다.
3. 중간 데이터 구조를 만들어서 앞에서 추출한 함수의 인수로 추가한다.
4. 테스트한다.
5. 추출한 두 번째 단계 함수의 매개변수를 하나씩 검토한다. 그중 첫 번째 단계에서 사용되는 것은 중간 데이터 구조로 옮긴다. 하나씩 옮길 때마다 테스트한다.
→ 간혹 두 번째 단계에서 사용하면 안 되는 매개변수가 있다. 이럴 때는 각 매개변수를 사용한 결과를 중간 데이터 구조의 필드로 추출하고, 이 필드의 값을 설정하는 문장을 호출한 곳으로 옮긴다(8.4절).
6. 첫 번째 단계 코드를 함수로 추출(6.1절)하면서 중간 데이터 구조를 반환하도록 만든다.
→ 이때 첫 번째 단계를 변환기(transformer) 객체로 추출해도 좋다.