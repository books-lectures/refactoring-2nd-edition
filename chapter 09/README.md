# 9. 데이터 조직화

## 1. 변수 쪼개기

역할 하나당 변수는 하나다. 여러 용도로 쓰인 변수는 코드를 읽는 이에게 혼란을 준다. 역할이 둘 이상인 변수가 있다면 쪼개야 한다.

1. 변수에 새로운 이름을 지어주고
2. 선언시 가능하면 불변으로 만든다.
3. 두번째 대입 전까지 모든 참조를 새로운 이름으로 바꾼다.
4. 두번째로 대입할 때 변수를 다시 선언한다.
5. 컴파일 후 테스트
6. 두번째 대입을 처리. 두번째 용도에 적합한 이름으로 수정

1-1. 예시 : 입력 매개변수의 값을 수정할때

```jsx
function discount (inputValue, quantity) {
	if(inputValue > 50) inputValue = inputValue -2;
	if(quantity > 100) inputValue = inputValue -1;
	return inputValue;
}
```

자바스크립트의 매개변수는 값에 의한 호출 방식으로 전달되므로 inputValue 를 수정해도 호출자에 영향을 주지 않는다.

```jsx
function discount(inputValue, quantity) {
	let result = inputValue;
	if(inputValue > 50) result = result -2;
	if(quantity > 100) result = result -1;
	return result;
} // 입력값에 기초하여 계산한다는 사실을 명확하기 하기
```

## 2. 필드이름 바꾸기
- 이름은 중요하다.
    1. 레코드를 클래스로 캡슐화한다.
    2. 캡슐화된 객체 안의 private 필드명을 변경하고 그에 맞게 내부 메서드들을 수정한다
    3. 테스트
    4. 생성자의 매개변수 중 필드와 이름이 겹치는게 있다면 함수 선언 바꾸기로 변경한다
    5. 접근자들의 이름도 바꿔준다.

## 3. 파생 변수를 질의 함수로 바꾸기
- 가변 데이터의 유효범위를 가능한 한 좁혀야 한다.

1. 변수값에 갱신되는 지점을 모두 찾는다. 필요하면 변수 쪼개기를 활용해 각 갱신 지점에서 변수를 분리한다.
2. 해당 변수의 값을 계산해주는 함수를 만든다.
3. 해당 변수가 사용되는 모든 곳에 어서션을 추가 하여 함수의 계산 결과가 변수의 값과 같은지 확인한다.
4. 테스트한다.
5. 변수를 읽는 코드를 모두 함수 호출로 대체한다.
6. 테스트
7. 변수를 선언하고 갱신하는 코드를 죽은 코드 제거하기로 없앤다.

```jsx
//ProductionPlan 클래스
get production() { return this._production;}
applyAdjustment(anAdjustment){
	this._adjustments.push(adAdjustment);
	this._production += anAdjustment.amount;
}
```

adjustment를 적용하는 과정에서 직접 관련이 없는 누적값 production 까지 갱신했다.

assert(this._production === this.calculatedProduction)

추가하여 테스트 후 어서션이 실패하지 않으면 필드를 반환하던 코드를 수정하여 계산 결과를 직접 반환하도록 한다.

```jsx
get production(){
	return this._adjustment.reduce((sum, a)=> sum+ a.amount, 0);
```

## 4. 참조를 값으로 바꾸기 
- 값 객체는 불변(const) 이기 떄문에 대체로 자유롭게 활용하기가 좋다.
1. 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인
2. 각각의 세터를 하나씩 제거
3. 이 값 객체의 필드들을 사용하는 동치성 비교 메서드를 만든다

## 5. 값을 참조로 바꾸기
- 모든 복제본을 갱신해야하며, 하나라도 놓치면 데이터 일관성이 깨지는 경우에는 복제된 데이터들을 모두 참조로 바꿔주는게 좋다.
1. 같은 부류에 속하는 객체들을 보관할 저장소를 만든다
2. 생성자에서 이 부류의 객체들 중 특정 객체를 정확히 찾아내는 방법이 있는지 확인한다
3. 호스트 객체의 생성자들을 수정하여 필요한 객체를 이 저장소에서 찾아내도록 한다. 하나 수정할 때마다 테스트 한다.

## 6. 매직 리터럴 바꾸기.
- 소스 코드에 등장하는 일반적인 리터럴 값

ex) M = 남성, “서울” = 본사, 9.80665 = 표준 중력