# 테스트 구축하기


> 💡 리팩터링을 제대로 하려면 불가피하게 저지르는 실수를 잡아주는 `테스트 수트`가 뒷받침 되어야 한다.   
리팩터링을 하지 않더라도, 좋은 테스트를 작성하는 일은 개발 효율을 높여준다.   
저자는 테스트코드가 갖춰지지 않은 코드를 리팩터링 할때, `테스트코드` 먼저 작성한다고 한다  

## 자가 테스트 코드의 가치

프로그래머는 `대부분의 시간`을 `디버깅` 하는데 쓴다. 버그 수정 자체는 대체로 금방 끝나지만 버그를 찾는과정이 끔찍하다. 버그를 잡으면서 새로운 버그를 만들어내는데 이는 많은 시간을 잡아먹게 된다.

```
 모든 테스트를 완전히 자동화하고 그 결과까지 스스로 검사하게 만들자
```

- 자동화 하면 컴파일 할 때마다 테스트를 할 수 있고, `디버깅 시간`이 줄어, 결과적으로 `생산성`이 급상승하게 된다.
- 테스트를 작성하려면 소프트웨어 본체 외의 부가적인 코드를 상당량 작성해야한다. 그렇기 때문에 테스트가 실제로 프로그래밍 속도를 높여주는 경험을 직접 해보지 않고서는, 자가 테스트의 진가를 납득하긴 어렵다.
- 테스트를 수동으로 하면 좀이 쑤실정도로 지겹다. 하지만 자동화한다면 테스트코드 작성이 재미있다
- 테스트를 작성하기 좋은 시점은 프로그래밍 시작 전이다.
- 기능을 추가하기 위해 무엇이 필요한지 고민하게 된다
- 구현보다 인터페이스에 집중하게 된다(무조건 좋은일이다)
- 코딩의 완료시점을 정확하게 판단할 수 있다(테스트 통과시점)

### TDD(Test-Driven Development)

- 테스트를 작성하고, 이 테스트를 통과하게끔 코드를 작성하고, 결과 코드를 최대한 깔끔하게 리팩터링하는 과정을 짧은 주기로 반복
- 테스트-코딩-리팩터링 과정을 여러차례 진행할 수 있기 때문에 생산적으로, 차분하게 작성가능

# 테스트 샘플코드

생산자가 생산 계획을 검토하고 수정하도록 해주는 애플리케이션의 코드를 테스트한다

```js
// 테스트용 데이터(Fixture)를 반환하는 함수
function sampleProvinceData() {
    return {
        name: "Asia",
        producers: [
            {name: "Byzantium", cost: 10, production: 9},
            {name: "Attalia", cost: 12, production: 10},
            {name: "Sinope", cost: 10, production: 6},
        ],
        demand: 30,
        price: 20
    }
}

// 지역 클래스
class Province {
    
    constructor(doc) {
				// 생성자에서 생성자 값을 조작하는 여러 작업을 할 수 있다
		    // 그런 작업(메소드)는 클래스 안에 작성해둔다
        this._name = doc.name;
        this._producers = [];
        this._totalProduction = 0;
        this._demand = doc.demand;
        this._price = doc.price;
        doc.producers.forEach(d => {
            this.addProducer(new Producer(this, d))
        });
    }

		// 꼭 필요한 property만 선언하고, computed value는 getter로 대체한다
    get name() {return this._name}
    get producers() {return this._producers.slice();}
    get totalProduction() {return this._totalProduction;}
    set totalProduction(arg) {return this._totalProduction = arg;}
    get demand() {return this._demand}
    set demand(arg) {this._demand = parseInt(arg);} // 숫자로 파싱후 저장
    get price() {return this._price}
    set price(arg) {this._price = parseInt(arg);} // 숫자로 파싱후 저장

    addProducer(arg) {
        this._producers.push(arg);
        this._totalProduction += arg.production;
    }

		// 생산 부족분
    get shortfall() {
        return this._demand - this.totalProduction;
    }
		
		// 수익
    get profit() {
        return this.demandValue - this.demandCost;
    }

    get demandValue() {
        return this.satisfiedDemand * this.price;
    }
    get satisfiedDemand() {
        return Math.min(this._demand, this.totalProduction);
    }

    get demandCost() {
        let remainingDemand = this.demand;
        let result = 0;
        this.producers.sort((a, b) => a.cost - b.cost).forEach(p => {
            const contribution = Math.min(remainingDemand, p.production);
            remainingDemand -= contribution;
            result += contribution * p.cost;
        });
        return result;
    }
}

// 생산자 클래스
// 단순 데이터 저장소
class Producer {

    constructor(aProvince, data) {
        this._province = aProvince
        this._cost = data.cost
        this._name = data.name
        this._production = data.production || 0

    }

    get name() {return this._cost}
    get cost() {return this._cost}
    set cost(arg) {this._cost = parseInt(arg);}
    get production() {return this._production;}
    set production(amountStr) {
        const amount = parseInt(amountStr);
        const newProduction = Number.isNaN(amount) ? 0 : amount;
        this._production += newProduction - this._production;
        this._production = newProduction;
    }
}
```

# 테스트 코드

- 테스트할때는 틀린 값 -> 맞는 값 -> 계산로직 틀리게 수정 -> 코드원복 의 과정으로 검증한다
- `it` 구문 하나당 검증도 하나씩만 하는 게 좋다. 앞쪽 검증을 통과하지 못하면 나머지 검증은 실행해보지 못하고 테스트가 실패하게 되는데, 실패 원인을 파악하는 데 유용한 정보를 놓치기 쉽기 때문이다. 다만 한 테스트로 묶어도 문제되지 않을 정도로 두 속성이 밀접 하다고 판단하면 동시에 검증 해도 좋다.
- 중복되는 픽스쳐를 상위 스코프로 올려, 테스트 끼리 상호작용하는 픽스처를 만들면 코드가 지저분해지고 의도하지 않는 결과가 나올 수 있다
- beforeEach로 각각의 테스트 바로 전에 Fixture를 초기화 한다
- assertion라이브러리는 픽스처 검증을 하는 용도이고 다양하다
- assert: assert.equal로 검증
- chai:  expect.equal로 검증
- 의식적으로 `코드를 망가뜨리기 위한 값`들을 넣어본다
- 에러와 실패를 구분해야한다. 이를 지원하는 테스트 프레임워크들이 있다
- 외부에서 들어온 잘못된 데이터가 프로그램 내부에 흘러서 디버깅 하기 어려워지는 경우는 `어셔선 추가하기`로 오류가 최대한 빨리 드러나게 해야한다
- 테스트에 너무 빠져들면 의욕이 떨어질 수 있기 때문에 `위험한 부분`에 집중하는게 좋다

## 생산 부족분, 이익 계산 테스트코드

```jsx
var assert = require('assert');
const chai = require('chai')
var expect = chai.expect;

describe("province", () => {
		let asia;
    beforeEach(function() {
        asia = new Province(sampleProvinceData());
    })

    it('shortfall', function() {
        const asia = new Province(sampleProvinceData()); // 픽스처(테스터에 필요한 데이터와 객체)
        assert.equal(asia.shortfall, 5) // 검증
        expect(asia.shortfall).equal(5) // chai 설치필요
    })

    it('profit', function() {
        const asia = new Province(sampleProvinceData());
        expect(asia.profit).equals(230)
    })
});
```

## Fixture 수정하기

실전에서 사용자가 값을 변경하며 사용하는 상황을 가정한다

```jsx
describe("province", () => {
    let asia;
    beforeEach(function() {
        asia = new Province(sampleProvinceData());
    })

    // 복잡한 setter를 위한 테스트
    it('change production', function() {
        // 픽스처를 수정해서 계산값 확인
        asia.producers[0].production = 10;
        expect(asia.shortfall).equals(5);
        expect(asia.profit).equal(230)
    });
});
```

## 경계조건 검사하기

우리의 의도대로 사용되는 범위 바깥의 상황 테스트하기

```jsx
// producer 컬렉션이 비어있는 경우
describe('no producers', () => {

		let noProducers;
    beforeEach(function() {
        const data = {
            name: "No producers",
            producers: [],
            demand: 30,
            price: 20
        }
        noProducers = new Province(data)
    })

    it('shortfall', function() {
        expect(noProducers.shortfall).equal(30)
    })

    it('profit', function() {
        expect(noProducers.profit).equal(0)
    })
		
})

// demand가 0, 음수 인경우
describe('province', () => {

    let asia;
    beforeEach(function() {
        asia = new Province(sampleProvinceData());
    })

    // demand가 0인경우
    it('zero demand', function() {
        asia = new Province(sampleProvinceData());
        asia.demand = 0;
        expect(asia.shortfall).equal(-25);
        expect(asia.profit).equal(0);
    })

    // demand가 음수
    it('minus demand', function() {
        asia = new Province(sampleProvinceData());
        asia.demand = -1;
        expect(asia.shortfall).equal(-26);
        expect(asia.profit).equal(-10);
    })
})
```

## 끝나지 않은 여정

- 위 예시는 단위테스트이다. 이외에도 컴포넌트 사이 상호작용 테스트나 소프트웨어의 다양한 계층 연동 테스트, 성능 테스트 등이 있다
- 한번에 완벽한 테스트를 갖추긴 힘들기 때문에 코드와 함께 `지속적인 보강`을 한다
- 기능을 추가할때마다 테스트를 추가하고, 기존 테스트를 리팩터링 한다
- 버그를 발견하는 즉시 버그를 명확하게 잡아내는 테스트 코드를 작성한다
