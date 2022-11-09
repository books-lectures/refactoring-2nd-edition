# 10. 조건부 로직 간소화

## 1. 조건문 분해하기

```jsx
if (!aDate.isBefore(plan.summerStart) && !aData.isAfter(pan,summerEnd))
	charge = quantity * plan.summerRate;
else
	charge = quantity * plan.summerRate + plan.rehularServiceCharge;
```

🔽

```jsx
if (summer())
	charge = summerCharge();
else
	charge = regularCharge();
```

## 배경

 복잡한 조건부 로직은 프로그램을 복잡하게 만드는 원흉에 속한다.
코드가 긴 함수는 그 자체로 읽기 힘들지만 조건문은 무슨 일이 일어나는지는 알려주만 왜 일어나는지 제대로 전하지 않을 때가 많아 이를 심화시킨다.

## 절차

1. 조건식과 그 조건식에 딸린 조건절을 각각 함수로 추출한다.

## 2. 조건문 통합하기

```jsx
if (anEployee.seniority < 2) return 0;
if (anEployee.monthsDisabled > 12) return 0;
if (anEployee.isPartTime) return 0;
```

🔽

```jsx
if (isNotEligibleForDisability()) return 0;

function isNotEligibleForDisability() {
	return if (anEployee.seniority < 2) 
	|| if (anEployee.monthsDisabled > 12) 
	||if (anEployee.isPartTime)
}
```

## 배경

 비교하는 조건은 다르지만 수행하는 동작은 같은 코드들에 경우 조건도 하나로 통합하는 것이 좋다.

통합함으로써 내가 하려는 일이 명확해지고 이 작업이 `함수 추출하기` 까지 이어질 가능성이 높다. 

## 절차

1. 해당 조건식들 모두에 부수효과가 없는지 확인한다.
2. 조건문 두 개를 선택하여 두 조건문의 조건식을 논리 연산자로 결합한다.
3. 테스트한다.
4. 조건이 하나만 남을 때까지 2~3 과정을 반복한다.
5. 하나로 합쳐진 조건식을 함수로 추출할지 고려해본다.

## 3. 중첩 조건문을 보호 구문으로 바꾸기

```jsx
function getPayAmount() 
{
	let result;

	if (isDead)
		result = deadAmount();
	else 
	{
		if (isSeaprated)
			result = separatedAmount();
		else 
		{
			if (isRetired)
				result = retiredAmount();
			else
				result = normalPayAmount();
		}
	}
}
```

🔽

```jsx
function getPayAmount() {
	if (isDead) return deadAmount();
	if (isSeaprated) return separatedAmount();
	if (isRetired) return retiredAmount();
	return normalPayAmount();
}
```

## 배경

 이 리펙터링의 핵심은 의도를 부여하는 데에 있다.
만일 함수의 반환점이 하나여야만 함수의 로직이 명확하다면 그렇게하되 그런 경우가 아니라면 고집하지 말자.

## 절차

1. 교체해야 할 조건 중 가장 바깥 것을 선택하여 보호 구문으로 바꾼다.
2. 테스트한다.
3. 1~2 과정을 필요한 만큼 반복한다.
4. 모든 보호 구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합한다.

## 4. 조건문 로직을 다형성으로 바꾸기

```jsx
switch (bird.type) 
{
	case '유령 제비' :
		return "보통이다";
	case '아프리카 제비' :
		return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
	case '노르웨이 파랑 앵무' :
		return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
	default:
		return "알 수 없다.";
}
```

🔽

```jsx
class EuropeanSwallow 
{
	get plumage() 
	{
		return "보통이다";
	}
	...
class AfricanSwallow
{
	get plumage() 
	{
		return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
	}
	...
class NorwegianBlueParrot
{
	get plumage() 
	{
		return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
	}
	...
```

## 배경

 복잡한 조건부 로직은 해석하기 어려운 대상 중 하나이다. 
클래스와 다형성을 이용한다면 조건을 분리하여 보다 직관적인 코드를 만들 수 있다.

## 절차

1. 다형적 동작을 표현하는 클래스들이 없다면 만들어준다. 이왕이면 적합한 인스턴스를 알아서 만들어 반환하는 팩터리 함수도 함께 만든다.
2. 호출하는 코드에서 팩터리 함수를 사용하게 한다.
3. 조건부 로직 함수를 슈퍼클래스로 옮긴다.
4. 서브클래스 중 하나를 선택한다. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드르 오버라이드한다. 조건부 문장 중 선택된 서브클래스에 해당하는 조건절을 서브클래스 메서드로 복사한 다음 적절히 수정한다.
5. 같은 방식으로 각 조건절을 해당 서브클래스에서 메서드로 구현한다.
6. 슈퍼클래스 메서드에서는 기본 동작 부분만 남긴다. 혹은 슈퍼클래스가 추상 클래스여야 한다면, 이 메서드를 추상으로 선언하거나 서브클래스에서 처리해야 함을 알리는 에러를 던진다.

## 5. 특이 케이스 추가하기

```jsx
if (aCustomer === '미확인 고객') {
  customerName = '거주자';
}
```

🔽

```jsx
class UnknownCustomer {
  get name() {
    return '거주자';
  }
}
```

## 배경

코드베이스에서 특정 값에 대해 똑같이 반응하는 코드가 여러 곳이라면 그 반응들을 한 데로 모으는 게 효율적이다.

특수한 경우의 공통 동작을 요소 하나에 모아서 사용하는 특이 케이스 패턴(Special Case Pattern)이라는 것이 있는데 이 패턴을 활용하면 특이 케이스를 확인하는 코드 대부분을 단순한 함수 호출로 바꿀 수 있다.

## 절차

이번 리팩터링의 대상이 될 속성을 담은 데이터 구조를 컨테이너라고 하겠다.

1. 컨테이너에 특이 케이스인지를 검사하는 속성을 추가하고, `false` 를 반환하게 한다.
2. 특이 케이스 객체를 만든다. 이 객체는 특이 케이스인지를 검사하는 속성만 포함하며, 이 속성은 `true` 를 반환하게 한다.
3. 클라이언트에서 특이 케이스인지를 검사하는 코드를 함수로 추출(6.1절)한다. 모든 클라이언트가 값을 직접 비교하는 대신 방금 추출한 함수를 사용하도록 고친다.
4. 코드에 새로운 특이 케이스 대상을 추가한다. 함수의 반환 값으로 받거나 변환 함수를 적용하면 된다.
5. 특이 케이스를 검사하는 함수 본문을 수정하여 특이 케이스 객체의 속성을 사용하도록 한다.
6. 테스트한다.
7. 여러 함수를 클래스로 묶기(6.9절)나 여러 함수를 변환 함수로 묶기(6.10절)를 적용하여 특이 케이스를 처리하는 공통 동작을 새로운 요소로 옮긴다.
→ 특이 케이스 클래스는 간단한 요청에는 항상 같은 값을 반환하는 게 보통이므로, 해당 특이 케이스의 리터럴 레코드를 만들어 활용할 수 있을 것이다.
8. 아직도 특이 케이스 검사 함수를 이용하는 곳이 남아 있다면 검사 함수를 인라인(6.2절)한다.

---

## 6. Assertion 추가하기

```jsx
if (this.discountRate) {
  base = base - (this.discountRate * base);
}
```

🔽

```jsx
assert(this.discountRate >= 0);
if (this.discountRate) {
  base = base - (this.discountRate * base);
}
```

## 배경


> Assertion은 시스템 운영에 영향을 주면 안 되므로 assertion을 추가한다고 해서 동작이 달라지지는 않는다.


특정 조건이 참일 때만 제대로 동작하는 코드 영역이 있을 수 있다. 이 때 알고리즘을 직접 연역하거나 주석에 적혀있는 것을 보고 알아차리는 것도 한 가지 방법이다. 하지만 assertion을 이용해 코드 자체에 삽입해놓는 것이 제일 좋다.

Assertion을 사용하는 프로그램이 어떤 상태임을 가정한 채 실행되는지를 다른 개발자에게 알려주는 훌륭한 소통 도구이다.

## 절차

1. 참이라고 가정하는 조건이 보이면 그 조건을 명시하는 assertion을 추가한다.