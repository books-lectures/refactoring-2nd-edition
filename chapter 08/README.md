# 8. 기능 이동

요소를 다른 컨텍스트(클래스나 모듈)로 옮기는 일 역시 리팩터링의 중요한 축이다.

옮기기는 문장단위, 함수단위등이 있다. 마지막으로 필요없는 코드를 제거하기가 있다

## 1. 함수 옮기기

### 배경

좋은 소프트웨어 설계의 핵심은 모듈성이다. 모듈성이 좋으면 어딘가를 수정해야 할때 해당 기능과 깊이 관련된 작은 일부만 이해해도 가능하게 된다.

모듈성을 높이려면 서로 연관된 요소들을 함께 묶고, 요소사이의 연결관계를 쉽게 찾고 이해할 수 있도록 해야한다. 보통은 이해도가 높아질 수록 소프트웨어 요소들을 더 잘 묶는 새로운 방법을 깨우치게 된다. 그렇게 높아진 이해를 반영하려면 요소들을 이리저리 옮겨야 한다

어떤 함수가 모듈 A보다 모듈 B를 더 많이 참조한다면 이 함수를 모듈B로 옮겨주는것이 합당하다

### 절차

1. 선택한 함수가 현재 컨텍스트에서 사용중인 요소를 모두 살펴본다. 이 요소들중 같이 옮겨야 할것이 있는지 살펴본다
2. 선택한 함수가 다형메서드인지 확인한다
3. 선택한 함수를 타깃 컨텍스트(클래스 or 모듈)로 복사한다. 타깃함수가 새로운 터전에 잘 자리잡도록 다듬는다(함수명 변경, 매개변수 를 넘기는 등)
4. 정적분석 수행
5. 소스 컨텍스트에서 타깃함수를 참조할 방법을 찾아 반영한다
6. 소스 함수를 타깃함수의 위임 함수가 되도록 수정한다
7. 테스트한다
8. 소스함수를 인라인할지 고민해본다

### [ 예시 ]

중첩 함수를 독립적으로 사용할 목적으로 최상위로 옮기는 예시

### BEFORE

```jsx
function trackSummary(points) {
    const totalTime = calculateTime();
    const totalDistance = calculatedDistance();
    const pace = totalTime / 60 / totalDistance;
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    }

    // 총 거리 계산 헬퍼 함수
    function calculatedDistance() {
        let result = 0;
        for(let i=0; i < points.length; i++) {
            result += distance(points[i-1], points[i])
        }
        return result;
    }

    function distance(d1, d2) {
        // 두지점 간의 거리 계산
    };
    function radians(degrees) {
        // 라디안 값으로 변환
    };
    function calculateTime() {
        // 총 시간 계산
    }
}
```

### STEP 1

- 추출할 함수를 최상위 함수로 복제한다
- 의존하고 있는 함수들이 다른곳에서 쓰이지 않는경우 함수 안으로 옮긴다
- 원래함수에서 추출된 함수를 호출하여, 추출된 함수가 잘 작동하는지 확인한다

```jsx
function trackSummary(points) {
    const totalTime = calculateTime();
    const totalDistance = calculatedDistance();
    const pace = totalTime / 60 / totalDistance;
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    }

    // 총 거리 계산 헬퍼 함수
    function calculatedDistance() {
        return top_calculatedDistance(points);
    }

    function calculateTime() {
        // 총 시간 계산
    }
}

// 총 거리 계산 헬퍼 함수 복사본
function top_calculatedDistance(points) {
    let result = 0;
        for(let i=0; i < points.length; i++) {
            result += distance(points[i-1], points[i])
        }
        return result;
				
				// 의존하는 함수들 내부로 가져옴
        function distance(d1, d2) {
            // 두지점 간의 거리 계산
        };
        function radians(degrees) {
            // 라디안 값으로 변환
        };
}
```

### AFTER

- 추출한 함수를 호출하는 기능만 하고있는 원본함수를 제거한다
- 추출한 함수 이름을 적절하게 변경한다

```jsx
function trackSummary(points) {
    const totalTime = calculateTime();
    const totalDistance = totalDistance(points);
		// 추출한 함수를 사용
    const pace = totalTime / 60 / totalDistance(points);
    return {
        time: totalTime,
        distance: totalDistance,
        pace: pace
    }

    function calculateTime() {
        // 총 시간 계산
    }
}

// 총 거리 계산 헬퍼 함수
function totalDistance(points) {
    let result = 0;
        for(let i=0; i < points.length; i++) {
            result += distance(points[i-1], points[i])
        }
        return result;

        function distance(d1, d2) {
            // 두지점 간의 거리 계산
        };
        function radians(degrees) {
            // 라디안 값으로 변환
        };
}
```

## 2. 필드 옮기기

주어진 문제에 적합한 데이터 구조를 활용하면, 코드는 자연스럽고 단순하고 직관적으로 짜여진다. 

그렇기 때문에 데이터구조는 중요하다. 데이터 구조를 잘 짜려면 경험과 도메인 주도 설계같은 기술이 필요하다

데이터구조가 적절치 않은경우 곧바로 수정해야한다. 고치지 않고 남겨진 잘못된 데이터 구조는 이후 작성되는 코드들을 복잡하게 만들어 버린다

### 절차

1. 소스필드가 캡슐화 되어있지 않다면 캡슐화한다
2. 테스트한다
3. 타깃객체에 필드와 접근자 메서드를 생성한다
4. 정적 검사를 수행한다
5. 소스 객체에서 타깃 객체를 참조할 수 있는지 확인한다
6. 접근자들이 타깃필드를 사용하도록 수정한다
7. 테스트한다
8. 소스필드를 제거한다
9. 테스트한다

### 예시

### BEFORE

고객클래스(Customer)와 계약(CustomerContract) 클래스

```jsx
class Customer {
    constructor(name, discountRate) {
        this._name = name;
        this._discountRate = discountRate;
        this._contract = new CustomerContract(dateToday());
    }

    get discountRate() {return this._discountRate}
    becomePrefferd() {
        this.discountRate += 0.03;
        // other stuffs
    }
    applyDiscount(amount) {
        return amount.subtract(amount.multiply(this._discountRate));
    }
}

class CustomerContract {
    constructor(startDate) {
        this._startDate = startDate;
    }
}
```

### AFTER

discountRate필드를 고객클래스(Customer) 에서 계약클래스(CustomerContract)로 옮기자

```jsx
class Customer {
    constructor(name, discountRate) {
        this._name = name;
        this._contract = new CustomerContract(dateToday());
        this._setDiscountRate(discountRate);
    }

    // 3. _discountRate 변수를 _contract에서 참조하고있다
    get discountRate() {return this._contract._discountRate}
    _setDiscountRate(aNumber) {this._contract.discountRate = aNumber}
    
    // 1. 메소드를 통해 필드값을 참조하여, 필드를 캡슐화함
    becomePrefferd() {
        this._setDiscountRate(this.discountRate += 0.03);
        // other stuffs
    }
    applyDiscount(amount) {
        return amount.subtract(amount.multiply(this._discountRate));
    }
}

class CustomerContract {
    constructor(startDate, discountRate) {
        this._startDate = startDate;
        // 2. 새로운 필드 discountRate 생성
        this._discountRate = discountRate
    }

    get discountRate() {return this._discountRate}
    set discountRate(arg) {this._discountRate = arg}
}
```

## 3. 문장을 함수로 옮기기

### 배경

중복제거는 코드를 건강하게 관리하는 가장 효과적인 방법중 하나이다

중복되는 코드들을 하나로 모아두면 나중에 수정할 일이 생겼을때도 한곳만 수정하면 된다

### 절차

1. 반복 코드가 함수 호출 부분과 멀리 떨어져 있다면 문장 슬라이드하기를 적용해 근처로 옮긴다
2. 타깃 함수를 호출하는 곳이 한 곳 뿐이라면 단순히 소스 위치에서 해당 코드를 잘라내어 피호출 함수로 복사하고 테스트한다
3. 호출자가 둘 이상이면 호출자 중 하나에서 타깃함수 호출 부분과 그 함수로 옮기려는 문장들을 함게 다른함수로 추출한다. 추출한 함수에 임시이름을 짓는다
4. 다른 호출자 모두가 방금 추출한 함수를 사용하도록 수정한다. 하나씩 수정할 때마다 테스트한다
5. 모든 호출자가 새로운 함수를 사용하게 되면 원래 함수를 새로운 함수 안으로 인라인 한 후 원래 함수를 제거한다
6. 새로운 함수의 이름을 원래 함수의 이름으로 바꿔준다

### 예시

### BEFORE

```jsx
function renderPerson(outStream, person) {
    const result = [];
    result.push(`<p>${person.name}</p>`);
    result.push(renderPhoto(person.photo));
    result.push(`<p>제목: ${person.photo.title}</p>`);   // <-- 중복
    result.push(emitPhotoData(person.photo))            // <-- 중복
    return result.join("\n");
}

function photoDiv(p) {
    return [
        "<div>",
        `<p>제목: ${p.title}</p>`,  // <-- 중복 
        emitPhotoData(p),          // <-- 중복
        "</div>",
    ].join("\n");
}

function emitPhotoData(aPhoto) {
    const result = [];
    result.push(`<p>위치: ${aPhoto.location}</p>`,);
    result.push(`<p>날짜: ${aPhoto.date.toDateString()}</p>`,);
    return result.join("\n");
}
```

### AFTER

```jsx
function renderPerson(outStream, person) {
    const result = [];
    result.push(`<p>${person.name}</p>`);
    result.push(renderPhoto(person.photo));
		// 1. 중복되는 부분 합칠 함수 생성
    emitPhotoData(person.photo);
    return result.join("\n");
}

function photoDiv(p) {
    return [
        "<div>",
				// 1. 중복되는 부분 합칠 함수 생성
        emitPhotoData(p),
        "</div>",
    ].join("\n");
}

// 3. 함수명 수정
function emitPhotoData(p) {
    return [
				// 2. photoDiv 함수의 내용과 합침
        `<p>제목: ${p.title}</p>`,
        `<p>위치: ${aPhoto.location}</p>`,
        `<p>날짜: ${aPhoto.date.toDateString()}</p>`,
    ].join("\n");
}
```

---

# 4. 문장을 호출한 곳으로 옮기기

```jsx
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${photo.title}</p>\n`);
  outStream.write(`<p>위치: ${photo.location}</p>\n`);
}
```

🔽

```jsx
emitPhotoData(outStream, person.photo);
outStream.write(`<p>위치: ${photo.location}</p>\n`);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${photo.title}</p>\n`);
}
```

## 배경

초기에는 응집도 높고 한 가지 일만 수행하던 함수가 어느새 둘 이상의 다른 일을 수행하게 바뀔 수 있다.

예컨대 여러 곳에서 사용하던 기능이 일부 호출자에게는 다르게 동작하도록 바뀌어야 한다면 이런 일이 벌어진다. 그렇다면 개발자는 동작을 함수에서 꺼내 해당 호출자로 옮겨야 한다. 이런 상황이 오면 우선 문장 슬라이드하기(8.6절)을 적용해 달라지는 동작을 함수의 시작 혹은 끝으로 옮긴 후, 바로 이어 문장을 호출한 곳으로 옮기기 리팩터링을 적용하면 된다.

호출자와 호출 대상의 경계를 완전히 다시 그어야 할 때도 있다. 후자의 경우라면 함수 인라인하기(6.2절)부터 적용한 다음, 문장 슬라이드하기(8.6절)와 함수 추출하기(6.1절)로 더 적합한 경계를 설정하면 된다.

## 절차

1. 호출자가 한두 개뿐이고 피호출 함수도 간단한 단순한 상황이면, 피호출 함수의 처음(혹은 마지막)줄(들)을 잘라내어 호출자(들)로 복사해 넣는다(필요하면 적당히 수정한다). 테스트만 통과하면 이번 리팩터링은 여기서 끝이다.
2. 더 복잡한 상황에서는, 이동하지 ‘않길’ 원하는 모든 문장을 함수로 추출(6.1절)한 다음 검색하기 쉬운 임시 이름을 지어준다.
→ 대상 함수가 서브클래스에서 오버라이드 됐다면 오버라이드한 서브클래스들의 메서드 모두에서 동일하게, 남길 부분을 메서드로 추출한다. 이때 남겨질 메서드의 본문은 모든 클래스에서 똑같아야한다. 그런 다음 (슈퍼클래스의 메서드만 남기고) 서브클래스들의 메서드를 제거한다.
3. 원래 함수를 인라인(6.2절)한다.
4. 추출된 함수의 이름을 원래 함수의 이름으로 변경한다(함수 이름 바꾸기(6.5절)).
→ 더 나은 이름이 떠오르면 그 이름을 사용하자.

---

# 5. 인라인 코드를 함수 호출로 바꾸기

```jsx
let appliesToMass = false;
for (const s of states) {
  if (s === 'MA') {
    appliesToMass = true;
  }
}
```

🔽

```jsx
let appliesToMass = states.includes('MA');
```

## 배경

이미 존재하는 함수와 똑같은 일을 하는 인라인 코드를 발견하면 보통은 해당 코드를 함수 호출로 대체하길 원할 것이다. 예외가 있다면, 기존 함수의 코드를 수정하더라도 인라인 코드의 동작은 바뀌지 않아야 할 때뿐이다. 이 경우인가를 판단하는 데는 함수 이름이 힌트가 된다. 이름을 잘 지었다면 인라인 코드 대신 함수 이름을 넣어도 말이 된다. 말이 되지 않는다면 함수 이름이 적절하지 않거나(함수 이름을 바꿔주자(6.5절)), 그 함수의 목적이 인라인 코드의 목적과 다르기 때문일 것이다(따라서 함수 호출로 대체하면 안 된다).

특히 라이브러리가 제공하는 함수로 대체할 수 있다면 훨씬 좋다. 함수 본문을 작성할 필요조차 없어지기 때문이다.

## 절차

1. 인라인 코드를 함수 호출로 대체한다.
2. 테스트한다.

---

# 6. 문장 슬라이드하기

```jsx
const pricingPlan = retrievePricingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;
```

🔽

```jsx
const pricingPlan = retrievePricingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retreiveOrder();
let charge;
```

## 배경

관련된 코드들이 가까이 모여 있다면 이해하기가 더 쉽다. 예컨대 하나의 데이터 구조를 이용하는 문장들은 (다른 데이터를 이용하는 코드 사이에 흩어져 있기보다는) 한데 모여 있어야 좋다.

관련 코드끼리 모으는 작업은 다른 리팩터링(주로 함수 추출하기(6.1.절))의 준비 단계로 자주 행해진다. 관련 있는 코드들을 명확히 구분되는 함수로 추출되는 게 그저 문장들을 한데로 모으는 것보다 나은 분리법이다. 하지만 코드들이 모여 있지 않다면 함수 추출은 애초에 수행할 수조차 없다.

## 절차

1. 코드 조각(문장들)을 이동할 목표 위치를 찾는다. 코드 조각의 원래 위치와 목표 위치 사이의 코드들을 훑어보면서, 조각을 모으고 나면 동작이 달라지는 코드가 있는지 살핀다. 다음과 같은 간섭이 있다면 이 리팩터링을 포기한다.
→ 코드 조각에서 참조하는 요소를 선언하는 문장 앞으로는 이동할 수 없다.
→ 코드 조각을 참조하는 요소의 뒤로는 이동할 수 없다.
→ 코드 조각에서 참조하는 요소를 수정하는 문장을 건너뛰어 이동할 수 없다.
→ 코드 조각이 수정하는 요소를 참조하는 요소를 건너뛰어 이동할 수 없다.
2. 코드 조각을 원래 위치에서 잘라내어 목표 위치에 붙여 넣는다.
3. 테스트한다.

---

# 7. 반복문 쪼개기

```jsx
let averageAge = 0;
let totalSalary = 0;
for (const p of people) {
  averageAge += p.age;
  totalSalary += p.salary;
}
averageAge /= people.length;
```

🔽

```jsx
let totalSalary = 0;
for (const p of people) {
  totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people) {
  averageAge += p.age;
}
averageAge /= people.length;
```

## 배경

두 일을 한꺼번에 처리할 수 있다는 이유에서 종종 반복문 하나에서 두 가지 일을 수행하는 모습을 보게 된다. 하지만 이렇게 하면 반복문을 수정해야 할 때마다 두 가지 일 모두를 잘 이해하고 진행해야 한다. 반대로 각각의 반복문으로 분리해두면 수정할 동작 하나만 이해하면 된다.

반복문을 두 번 실행해야 하므로 이 리팩터링을 불편해하는 프로그래머도 많다. 제발 **리팩터링과 최적화를 구분하자.** 최적화는 코드를 깔끔히 정리한 이후에 수행하자. 반복문을 두 번 실행하는 게 병목이라 밝혀지면 그때 다시 하나로 합치기는 식은 죽 먹기다. 하지만 긴 리스트를 반복하더라도 병목으로 이어지는 경우는 매우 드물다. 오히려 반복문 쪼개기가 다른 더 강력한 최적화를 적용할 수 있는 길을 열어주기도 한다.

## 절차

1. 반복문을 복제해 두 개로 만든다.
2. 반복문이 중복되어 생기는 부수효과를 파악해서 제거한다.
3. 테스트한다.
4. 완료됐으면, 각 반복문을 함수로 추출(6.1절)할지 고민해본다.

---

# 8. 반복문을 파이프라인으로 바꾸기

```jsx
const names = [];
for (const i of input) {
  if (i.job === 'programmer') {
    names.push(i.name);
  }
}
```

🔽

```jsx
const names = input
  .filter(i => i.job === 'programmer')
  .map(i => i.name);
```

## 배경

옛날에는 객체 컬렉션을 순환할 때 단순 반복문을 사용했다. 하지만 요즘 언어가 제공하는 컬렉션 파이프라인을 이용하면 처리 과정을 일련의 연산으로 표현할 수 있다. 이때 각 연산은 컬렉션을 입력받아 다른 컬렉션을 내뱉는다. 논리를 파이프라인으로 표현하면 이해하기 훨씬 쉬워진다. 객체가 파이프라인을 따라 흐르며 어떻게 처리되는지를 읽을 수 있기 때문이다.

## 절차

1. 반복문에서 사용하는 컬렉션을 가리키는 변수를 하나 만든다.
→ 기존 변수를 단수히 복사한 것일 수도 있다.
2. 반복문의 첫 줄부터 시작해서, 각각의 단위 행위를 적절한 컬렉션 파이프라인 연산으로 대체한다. 이 때 컬렉션 파이프라인 연산은 (1)에서 만든 반복문 컬렉션 변수에서 시작하여, 이전 연산의 결과를 기초로 연쇄적으로 수행된다. 하나를 대체할 때마다 테스트한다.
3. 반복문의 모든 동작을 대체했다면 반복문 자체를 지운다.
→ 반복문이 결과를 누적 변수(accumulator)에 대입했다면 파이프라인의 결과를 그 누적 변수에 대입한다.

---

# 9. 죽은 코드 제거하기

```jsx
if (false) {
  doSomethingThatUsedToMatter();
}
```

🔽

```jsx

```

## 배경

죽은 코드(Dead code)가 몇 줄 있다고 해서 시스템이 느려지는 것도 아니고 메모리를 많이 잡아 먹지도 않는다. 심지어 최신 컴파일러들은 이런 코드를 알아서 제거해준다. 그렇더라도 죽은 코드가 있다면 소프트웨어의 동작을 이해하는 데는 커다란 걸림돌이 될 수 있다. 이 코드들 스스로는 ‘절대 호출되지 않으니 무시해도 되는 함수다’라는 신호를 주지 않기 때문이다.

죽은 코드를 지웠다가 다시 필요해질 날이 오지 않을까 걱정할 필요도 없다. 우리에겐 VCS가 있기 때문이다. 그런 날이 반드시 올 거라 생각된다면 어느 리비전에서 삭제했는지를 커밋 메시지로 남겨놓자.

VCS가 보편화되지 않고 쓰기 불편했던 시절에는 죽은 코드를 주석 처리하는 방법이 널리 쓰였다. 지금은 코드가 몇 줄 안 되는 초기 단계부터 VCS를 이용하므로, 더 이상은 필요치 않다.

## 절차

1. 죽은 코드를 외부에서 참조할 수 있는 경우라면(예컨대 함수 하나가 통째로 죽었을 때) 혹시라도 호출하는 곳이 있는지 확인한다.
2. 없다면 죽은 코드를 제거한다.
3. 테스트한다.