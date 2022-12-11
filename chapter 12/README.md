# 12. 상속 다루기

상속은 막강한 도구지만, 잘못된 곳에서 사용되거나 나중에 환경이 변해 문제가 생기기도 한다. 이럴때는 앞으로 나오는 방법을 통해 상속을 위임으로 바꿔준다.

## 1. 메서드 올리기

### 배경

상속받은 클래스들에서 중복되는 메서드들이 있다면, 슈퍼클래스(부모) 로 옮겨 중복을 없앨 수 있다

### 절차

1. 똑같이 동작하는 메서드인지 살펴보고, 같은일을 하지만 코드가 다르다면 같아질 때까지 리팩터링 한다
2. 메서드 안에서 호출하는 다른 메서드와 참조하는 필드들을 슈퍼클래스에서도 호출 및 참조 할수 있는지 확인한다
3. 메서드 시그니처(메서드 명과 파라미터의 순서, 타입, 개수)가 다르다면 통일한다
4. 슈퍼클래스에 중복을 통합한 메서드를 추가한다
5. 정적 검사
6. 서브클래스중 하나의 메서드를 제거
7. 테스트
8. 서브클래스에 해당 메서드가 없어질 때까지 하나씩 제거

### 예시

```jsx
// BEFORE

class Employee {

}

class SalesPersom extends Employee {
    get name(){...}
}

class Engineer extends Employee {
    get name(){...}
}
```

```jsx
// AFTER

class Employee {
    get name(){...}
}

class SalesPersom extends Employee {
    
}

class Engineer extends Employee {   

}
```

## 2. 필드 올리기

### 배경

서브클래스들이 독립적으로 개발되었거나 뒤늦게 하나의 계층구조로 리팩터링 된 경우라면 일부 기능 및 필드가 중복되었을 가능성이 높다. 같은 역할을 하는 필드들이 있다면 슈퍼클래스로 끌어올려 중복을 피할 수 있다.

### 절차

1. 중복된다고 보이는 필드들이 똑같은 방식으로 사용되고 있는지 확인
2. 필드들의 이름이 다르다면 이름 통일
3. 슈퍼클래스에 해당 필드 생성
4. 서브 클래스들에서 해당 필드 제거
5. 테스트

### 예시

```java
// BEFORE

class Employee {

}

// private 접근 제한자: 단어 뜻 그대로 개인적인 것이라 외부에서 사용될 수 없도록 합니다.
class SalesPersom extends Employee {
    private String name;
}

class Engineer extends Employee {
    private String name;
}

```

```java
// AFTER

// protected 접근 제한자: 같은 패키지 또는 자식 클래스에서 사용할 수 있도록 합니다.
class Employee {
    protected String name;
}

class SalesPersom extends Employee {
    
}

class Engineer extends Employee {
    
}
```

## 3. 생성자 본문 올리기

### 배경

서브클래스들이 같은 메서드나 필드를 가질때 슈퍼클래스로 옮기기를 했지만 중복되는 부분이 생성자에 있다면 스텝이 꼬이게 된다. 생성자는 호출순서에 제약이 있기 때문에 조금 다른식으로 접근한다

### 절차

1. 슈퍼 클래스에 생성자가 없다면 정의한다. 서브클래스의 생성자 들에서 super()를 통해 이 생성자가 호출되는지 확인한다
2. 공통문장을 모두 super()호출 직후로 옮긴다
3. 공통코드를 슈퍼클래스에 추가하고, 서브클래스에서 제거한다. 생성자 매개변수중 공통 코드에서 참조하는 값들을 모두 super로 건넨다
4. 테스트한다
5. 생성자 시작부분으로 옮길 수 없는 공통코드는 함수추출하기와 메서드 올리기를 차례로 적용한다

### 예시

```java
// BEFORE

class Party {

}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super();
        this._id = id;
        this._name = name;
        this._monthlyCost = monthlyCost;
    }
}

class Department extends Party {
    constructor(name, staff) {
        super();
        this._name = name;
        this._staff = staff;
    }
}
```

```java
// AFTER

class Party {
    constructor(name) {
        this._name = name;
    }
}

class Employee extends Party {
    constructor(name, id, monthlyCost) {
        super(name);
        this._id = id;
        this._monthlyCost = monthlyCost;
    }
}

class Department extends Party {
    constructor(name, staff) {
        super(name);
        this._staff = staff;
    }
}
```

## 4. 메서드 내리기

### 배경

특정 서브클래스와만 관련된 메서드는 슈퍼클래스에서 제거하고, 사용하는 서브클래스에 추가하는 것이 깔끔하다

### 절차

1. 제거할 메서드를 모든 서브클래스에 복사한다
2. 슈퍼클래스에서 해당 메서드를 제거한다
3. 테스트한다
4. 해당 메서드를 사용하지 않는 모든 서브클래스에서 제거한다
5. 테스트한다

### 예시

```jsx
// BEFORE

class Employee {
    get quota {}
}

class SalesPersom extends Employee {

}

class Engineer extends Employee {
    
}
```

```jsx
// AFTER

class Employee {
    
}

class SalesPersom extends Employee {

}

class Engineer extends Employee {
    get quota {}
}
```

## 5. 필드 내리기

### 배경

서브클래스에서만 사용하는 필드는 해당 서브클래스로 옮긴다

### 절차

1. 대상필드를 모든 서브클래스에 정의한다
2. 슈퍼클래스에서 해당 필드를 제거한다
3. 테스트한다
4. 이 필드를 사용하지 않는 모든 서브클래스에서 제거한다
5. 테스트한다

### 예시

```java
// BEFORE

class Employee {
    private String quota;
}

class SalesPersom extends Employee {

}

class Engineer extends Employee {
    
}
```

```java
// AFTER

class Employee {
    
}

class SalesPersom extends Employee {
    
}

class Engineer extends Employee {
    private String quota;
}
```

## 6. 타입 코드를 서브클래스로 바꾸기

### 배경

타입(특정 조건) 에 따라 동작을 분기할 때 타입코드 대신 서브클래스를 이용하면 더 좋은점이 많다. 먼저 기대하는대로 조건에 따라 다르게 동작하도록 할 수 있고, 해당 조건에 필요한 필드나 메서드를 서브클래스에 추가함으로써 불필요한 코드를 없앨 수 있다

### 절차

1. 타입 코드 필드를 자가 캡슐화 한다
2. 타입코드값 중 하나를 선택하여 그 값에 해당하는 서브클래스를 만든다
3. 매개변수로 받은 타입코드와 매치되는 서브클래스를 매핑하는 선택 로직을 만든다
4. 테스트한다
5. 타입코드 값 각각에 대한 서브클래스와 매핑 로직을 추가한다
6. 타입코드 필드를 제거한다
7. 테스트한다

### 예시

```jsx
// BEFORE

function createEmployee(name, type) {
    return new Employee(name, type);
}
```

```jsx
// AFTER

function createEmployee(name, type) {
    switch (type) {
        case "engineer": return new Engineer(name);
        case "salesperson": return new SalesPerson(name);
    }
}
```

## 7. 서브클래스 제거하기

### 배경

소프트웨어가 성장함에 따라 서브클래스가 사용되지 않기도 한다. 더이상 쓰지 않는 서브클래스는 슈퍼클래스의 필드로 대체한다

### 절차

1. 서브클래스의 생성자를 팩토리 함수로 바꾼다
2. 서브클래스의 타입을 검사하는 코드가 있다면 슈퍼클래스로 옮긴다
3. 서브클래스의 타입을 나타내는 필드를 슈퍼클래스에 만든다
4. 서브클래스를 참조하는 메서드가 방금 만든 타입 필드를 이용하도록 수정한다
5. 서브클래스를 지운다
6. 테스트한다

### 예시

```jsx
// BEFORE

class Person {
    get genderCode() {return "X";}
}

class Male extends Person {
    get genderCode() {return "M";}
}

class Female extends Person {
    get genderCode() {return "F";}
}
```

```jsx
// AFTER

class Person {
    constructor(genderCode) {
        this._genderCode = genderCode
    }
    get genderCode() {return this._genderCode;}
}

function createPerson(type) {
    switch (type) {
        case "M": return new Person("M");
        case "F": return new Person("F");
				default: return new Person("X")
    }
}
```

## 8. 슈퍼클래스 추출하기

### 배경

비슷한 일을 하는 두 클래스가 보이면 상속 매커니즘을 이용해 둘의 공통 슈퍼클래스를 만들어 공통되는 부분을 옮길 수 있다

### 절차

1. 빈 슈퍼클래스를 만든다. 기존 클래스들이 새 슈퍼클래스를 상속하도록 한다
2. 테스트한다
3. 생성자, 메서드, 필드 올리기를 이용해 공통 원소를 슈퍼클래스로 옮긴다
4. 서브클래스에 남은 메서드들을 검토한다. 
5. 원래 클래스들을 사용하는 코드를 검토하여 슈퍼클래스의 인터페이스를 사용하게 할지 고민해본다

### 예시

```jsx
// BEFORE

class Department {
    get totalAnnualCost() {}
    get name() {}
    get headCount() {}
}

class Employee {
    get annualCost() {}
    get name() {}
    get id() {}
}
```

```jsx
// AFTER

class Party {
    get name() {}
    get annualCost() {}
}

class Department extends Party {
    get headCount() {}
}

class Employee extends Party {
    get id() {}
}
```

## 9.  계층 합치기

### 배경

기능을 위로 올리거나 아래로 내리는 일은 자주 일어난다. 어떤 클래스와 그 부모가 너무 비슷해져서 더는 독립적으로 존재해야할 이유가 사라지는 경우가 있어 하나로 합쳐야 하는 경우다

### 절차

1. 제거할 것을 고른다
2. 모든 요소를 하나의 클래스로 옮긴다
3. 제거할 클래스를 참조하던 모든 코드가 남겨질 클래스를 참조하도록 고친다
4. 빈 클래스를 제거한다
5. 테스트

## 10. 서브클래스를 위임으로 바꾸기

### 배경

경우에 따라 동작이 달라지는 객체들은 상속으로 표현하는게 자연스럽다.

하지만 상속은 단점이 있다. 무언가 달라져야 하는 이유가 여러개여도 상속에서는 단 하나의 이유만 선택해 기준으로 삼을 수 밖에 없다. 예를들어 객체의 동작을 나이대와 소득 수준에 따라 달리하고 싶으면 서브클래스는 젊은이와 어르신이 되거나 혹은 부자와 서민이 되어야 한다. 둘다는 안된다.

위임은 이를 해결해준다. 위임은 객체 사이의 일반적인 관계이므로 상호작용에 필요한 인터페이스를 명확히 정의할 수 있다. 상속보다 결합도가 훨씬 약하다.

### 절차

1. 생성자를 호출하는 곳이 많다면 생성자를 팩터리 함수로 바꾼다
2. 위임으로 활용할 빈 클래스를 만든다. 이 클래스의 생성자는 서브 클래스에 특화된 데이터를 전부 받아야 하며, 보통은 슈퍼클래스를 가리키는 역참조도 필요하다.
3. 위임을 저장할 필드를 슈퍼클래스에 추가한다
4. 서브클래스 생성코드를 수정하여 위임 인스턴스를 생성하고 위임 필드에 대입해 초기화한다
5. 서브클래스의 메서드 중 위임 클래스로 이동할 것을 고른다
6. 함수 옮기기를 적용해 위임 클래스로 옮긴다. 
7. 서브 클래스 외부에도 원래 메서드를 호출하는 코드가 있다면 서브 클래스의 위임 코드를 슈퍼 클래스로 옮긴다.
8. 테스트
9. 서브클래스의 모든 메서드가 옮겨질때까지 반복
10. 테스트
11. 서브클래스를 삭제한다.

### 예시1

공연 예약 클래스 예시, 추가비용을 다양하게 설정할 수 있는 프리미엄 예약용 서브클래스

서브크래스를 위임으로 바꾸려 하는 이유? 기본 예약에서 프리미엄 예약으로 동적으로 전환해야 하는 경우. → 이러한 요구가 커지면 서브클래스를 위임으로 바꾸는게 좋다.

```jsx
get hasTalkback(){
return this._premiumDelegate.hasTalkback;
}

get hasTalkback(){
	return(this._premiumDelegate) 
	? this._premiumDelegate.hasTalkback
	: this._show.hasOwnProperty('talkback') && !this.isPeakDay;
}
```

위임을 적용하면 분배 로직과 양방향 참조가 더해지는 등 복잡도가 높아진다. 그래도 이 리팩터링이 여전히 가치가 있다. 동적으로 프리미엄 예약으로 바꿀 수 있고 상속은 다른 목적으로 사용할 수 있게 되었다.

### 예시2

서브클래스가 여러개 일때도 이번 리팩터링을 적용할 수 있다.

```jsx
function createBird(data){
	switch(data.type){
		case '유럽 제비' :
		return new EuropeanSwallow(data);
		case '아프리카 제비' :
		return new AfricanSwallow(data);
		case '노르웨이 파랑 앵무' :
		return new NorwegianBlueParrot(data);
		default:
		return new Bird(data);
	}
}

...
```

이 코드는 야생 조류와 사육 조류를 구분짓기 위해 크게 수정할 예정이다.

wildBird, captiveBird 라는 두 서브 클래스로 모델링 하는 방법도 있으나 상속은 한 번만 쓸 수 있으나 야생과 사육을 기준으로 나누려면 종에 대한 분류를 포기해야 한다.

```jsx
//bird class
constructor(data){
	this._name: data.name;
	this._plumage = data.plumage;
	this._speciesDelegate = this.selectSpeciesDelegate(data);
}

selectSpeciesDeletgate(data){
	switch(data.type) {
	case '유럽 제비':
		return new EuropeanSwallowDelegate();
	default: return null;
	}
}

//슈퍼클래스의 airSpeedVelocity() 를 수정하여 위임이 존재하면 위임의 메서드를 호출하도록 한다.
get airSpeedVelocity(){
	return this._speciesDelegate ? this._speciesDelegate.airSpeedVelocity : null;
}
```

bird를 상속으로 부터 구제한 것 외에 이 리팩터링에서 얻은 건 무엇일까?

위임으로 옮겨진 종 계층 구조는 더 엄격하게 종과 관련한 내용만을 다루게 되었다.

위임 클래스들은 종에 따라 달라지는 데이터와 메서드 만을 담게 되고 종과 상관없는 공통 코드는 bird 자체와 미래의 서브 클래스들에 남는다.

## 11. 슈퍼클래스를 위임으로 바꾸기

### 배경

상속이 더는 최선의 방법이 아니게 되면 슈퍼클래스를 위임으로 바꿀 수 있다. 상속을 먼저 적용하고 나중에 문제가 생기면 슈퍼클래스를 위임으로 바꾸는것이 저자의 조언.

### 절차

1. 슈퍼클래스 객체를 참조하는 필드를 서브클래스에 만든다.
2. 슈퍼클래스의 동작 각각에 대응하는 전달 함수를 서브클래스에 만든다. 서로 관련된 함수끼리 그룹으로 묶어 진행하며 그룹을 하나씩 만들 때마다 테스트 한다.
3. 슈버클래스의 동작 모두가 전달 함수로 오버라이드 되었다면 상속 관계를 끊는다.

### 예시

고대 스크롤을 보관하고있는 도서관

1. scroll 에 카탈로그 아이템을 참조하는 속성을 만들고 슈퍼클래스의 인스턴스를 새로 하나 만들어 대입

```jsx
constructor(id, title, tags, dateLastCleaned){
	super(id,title,tags)
	this._catalogItem = new CatalogItem(id, title, tags)
	this._lastCleaned = dateLastCleaned;
}
```

1. ㅇl 서브클래스에서 사용되는 슈퍼클래스의 동작 각각에 대응하는 전달 메서드를 만든다.
2. 카탈로그 아이템과의 상속관계를 끊는다