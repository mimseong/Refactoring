

# Chapter 10 조건부 로직 간소화

> 조건부 로직은 프로그램의 힘을 강화하는 데 크게 기여하지만, 프로그램을 복잡하게 만드는 원흉이기도 하다.

### 상황에 따라 사용하면 좋은 리팩터링 기법
- 복잡한 조건문일 때: 조건문 분해하기
- 논리적 조합을 명확하게 다듬어야할 때: 중복 조건식 통합하기
- 함수의 핵심 로직에 본격적으로 들어가기 앞서 무언가를 검사해야할 때: 중첩 조건문을 보호 구문으로 바꾸기
- 똑같은 분기 로직(switch문)이 여러곳에 등장: 조건부 로직을 다형성으로 바꾸기
- Null 같은 특이 케이스를 처리하는 로직이 거의 똑같을 때: 특이 케이스 추가하기
- 프로그램의 상태를 확인하고 그 결과에 따라 다르게 동작해야 할 때: 어서션 추가하기
- 제어 플래그를 이용해 코드 동작 흐름을 변경하는 코드: 제어 플래그를 탈출문으로 바꾸기


## 10.1 조건문 분해하기

```js
//변경 전
if(!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
  
  
//변경 후
if(summer())
  charge = summerCharge();
else
  charge = regularCharge();
```

- 복잡한 로건부 로직은 프로그램을 복잡하게 만든다.
- 조건문은 코드가 '왜' 일어나는지는 제대로 말해주지 않을 때가 많아서 문제다.
- 거대한 코드 블록이 있다면, 코드를 부위별로 분해한 다음 해체된 코드 덩어리들을 각 덩어리의 의도를 살린 이름의 함수호출로 바꿔주자. 그러면 전체적인 의도가 더 확실히 드러난다.

**절차**
1. 조건식과 그 조건식에 딸린 조건절 각각을 함수로 추출한다.


**예시**
여름철이면 할인율이 달라지는 어떤 서비스의 요금을 계산하는 로직

```js
if(!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
```

1. 조건식이 너무 길어 어떤 조건에서 조건문이 실행되는지 알기 어렵다. 조건식을 별도 함수로 추출하자.
  ```js
    if(summer())
      charge = quantity * plan.summerRage;
    else
      charge = quantity * plan.regularRate + plan.regularServiceCahrge;
     
    function summer() {
      return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
    }
  ```

2. 조건이 만족했을 때의 로직도 또 다른 함수로 추출한다.
  ```js
    if(summer())
      charge = summerCharge();
    else
      charge = quantity * plan.regularRate + plan.regularServiceCahrge;
     
    function summer() {
      return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
    }
    
    function summerCharge() {
      return quantity * plan.summerRage;
    }
  ```

3. else 절도 별도 함수로 추출한다.
  ```js
  if(summer())
      charge = summerCharge();
    else
      charge = regularCharge();
     
    function summer() {
      return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
    }
    
    function summerCharge() {
      return quantity * plan.summerRage;
    }
    
    function regularCharge() {
      return quantity * plan.regularRate + plan.regularServiceCahrge;
    }
  ```
  
  4. 원한다면 삼항 연산자로 바꿀 수도 있다.
   ```js
    charge = summer() ? summerCharge() : regularCharge();
     
    function summer() {
      return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
    }
    
    function summerCharge() {
      return quantity * plan.summerRage;
    }
    
    function regularCharge() {
      return quantity * plan.regularRate + plan.regularServiceCahrge;
    }
  ```

## 10.2 조건식 통합하기


```js
//변경 전
  if(anEmployee.seniority < 2) return 0;
  if(anEmployee.monthDisabled > 12) return 0;
  if(anEmployee.isPartTime) return 0;
  
//변경 후
  if(isNotEligibleForDisability()) return 0;
  
  function isNotEligibleForDisability() {
    return ((anEmployee.seniority < 2)
            || (anEmployee.monthDisabled > 12)
            || (anEmployee.isPartTime);
  }
```

- 비교하는 조건은 다르지만 그 결과를 수행하는 동작을 똑같은 코드가 있다면, 조건 검사를 하나로 통합하는게 낫다.
- 'and' 연산자롸 'or' 연산자를 사용하면 여러 개의 비교 로직을 하나로 합칠 수 있다.

### 조건부 코드를 통합하는 게 중요한 이유
1. 여러 조각으로 나뉜 조건들을 하나로 통합함으로써 동작의 목적이 명확해진다.
2. 이 작업이 함수 추출하기까지 이어질 가능성이 높기 때문이다. 복잡한 조건식을 함수로 추출하면 코드의 의도가 훨씬 분명하게 드러나는 경우가 많다.


**절차**
1. 해당 조건식들 모두에 부수효과는 없는지 확인한다.
  - 부수효과가 있는 조건식들에는 질의함수와 변경함수 분리하기를 먼저 적용한다.
2. 조건문 두 개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다.
3. 테스트
4. 조건이 하나만 남을 때까지 2~3 과정을 반복.
5. 하나로 합쳐진 조건식을 함수로 추출할 지 고려해본다.


## 10.3 중첩 조건문을 보호 구문으로 바꾸기
### 조건문의 두 가지 형태
1. 참인 경로와 거짓인 경로가 모두 정상 동작인 경우: if-else 구문을 사용
2. 한쪽만 정상인 경우: 비정상 조건을 if에서 검사한 후, 조건이 참이면 함수에서 빠져나오기 **(보호 구문)**

**절차**
1. 교체해야할 조건 중 가장 바깥 것을 보호 구문으로 바꾼다.
2. 테스트
3. 1~2 과정을 필요한 만큼 반복
4. 모든 보호구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합


**예시**
직원 급여를 계산하는 코드
현직 직원만이 급여를 받아야하므로 퇴사한 직원, 은퇴한 직원인지를 검사한다.

```js
  function payAmount(employee) {
    let result;
    if(employee.isSeperated) {
      result = {amount: 0, reasonCode: "SEP"};
    } else {
      if(employee.isRetired) {
        result = {amount: 0, reasonCode: "RET"};
      } else {
        lorem.ipsum(dolor.sitAmet); //급여 계산 로직
        consectetur(adispicing).elit();
        sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
        ut.enim.ad(minim.veniam);
        result = somFinalComputation();
      }
    }
    return result;
  }
```

실제로 중요한 일들이 중첩된 조건에 가려서 잘 보이지 않는다.
이 코드는 모든 조건이 거짓일 때만 일어나므로 보호 구문을 사용하면 코드의 의도가 더 잘 드러난다.


최상의 조건부터 보호 구문으로 바꿔보자.

```js
  function payAmount(employee) {
    let result;
    if(employee.isSeperated) return {amount: 0, reasonCode: "SEP"};
    if(employee.isRetired) {
      result = {amount: 0, reasonCode: "RET"};
    } else {
      lorem.ipsum(dolor.sitAmet); //급여 계산 로직
      consectetur(adispicing).elit();
      sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
      ut.enim.ad(minim.veniam);
      result = somFinalComputation();
    }
    return result;
  }
```

그 다음 조건도 보호 구문으로 바꿔준다.
```js
  function payAmount(employee) {
    let result;
    if(employee.isSeperated) return {amount: 0, reasonCode: "SEP"};
    if(employee.isRetired) return {amount: 0, reasonCode: "RET"};

    lorem.ipsum(dolor.sitAmet); //급여 계산 로직
    consectetur(adispicing).elit();
    sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
    ut.enim.ad(minim.veniam);
    result = somFinalComputation();
    return result;
  }
```

그 후 아무역할을 하지 않는 result 변수를 제거해준다.
```js
  function payAmount(employee) {
    if(employee.isSeperated) return {amount: 0, reasonCode: "SEP"};
    if(employee.isRetired) return {amount: 0, reasonCode: "RET"};

    lorem.ipsum(dolor.sitAmet); //급여 계산 로직
    consectetur(adispicing).elit();
    sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
    ut.enim.ad(minim.veniam);
    return somFinalComputation();
  }
```

## 10.4 조건부 로직을 다형성으로 바꾸기
> 클래스와 다형성을 이용하면 조건부 로직을 직관적으로 구조화할 수 있다.

1. 타입이 여러개이고, 각 타입이 조건부 로직을 자신만의 방식으로 처리하도록 구성된 로직의 경우
  - case 별로 클래스를 하나씩 만들어 공통 switch 로직의 중복을 없애는 방법 활용
2. 기본 동작을 위한 case문과 그 변형 동작으로 구성된 로직의 경우
  - 기본 동작을 슈퍼클래스로 넣고, 변형 동작 case 들을 각각의 서브클래스로 만드는 방법 활용

**예시**

새의 종에 따른 비행 속도와 깃털 상태를 알려주는 프로그램

```js
  function plumages(birds) {
    return new Map(birds.map(b => [b. name, plumage(b)]));
  }
  
  function speeds(birds) {
    return new Map(birds.map(b => [b. name, airSpeedVelocity(b)]));
  }
  
  function plumage(bird) {
    switch (bird.type) {
      case '유럽 제비':
        return '보통이다';
     case '아프리카 제비':
        return (bird.numberOfCoconuts > 2) ? '지쳤다' : '보통이다';
     case '노르웨이 파랑 앵무':
        return (bird.voltage > 100) ? '그을렸다' : '예쁘다';
     default:
        return '알 수 없다';
    }
  }
  
  function airSpeedVelocity(bird) {
    switch (bird.type) {
      case '유럽 제비':
        return 35;
     case '아프리카 제비':
        return 40 - 2 * bird.numberOfCoconuts;
     case '노르웨이 파랑 앵무':
        return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
     default:
        return null;
    }
  }
```

새의 종류에 따라 다르게 동작하는 함수가 있으므로, 종류별 클래스를 만들어서 각각에 맞는 동작을 표현하면 될 것 같다.

먼저 airSpeedVelocity()와 plumage()를 Bird라는 클래스로 묶어보자.


```js
  

  function plumages(birds) {
    return new Bird(bird).plumage; new Map(birds.map(b => [b. name, plumage(b)]));
  }
  
  function speeds(birds) {
    return new Bird(bird).airSpeedVelocity;  new Map(birds.map(b => [b. name, airSpeedVelocity(b)]));
  }
  
  class Bird {
    constructor (birdObject) {
      Object.assign(this, birdObject);
    }
    
    get plumage() {
      switch (this.type) {
        case '유럽 제비':
          return '보통이다';
       case '아프리카 제비':
          return (this.numberOfCoconuts > 2) ? '지쳤다' : '보통이다';
       case '노르웨이 파랑 앵무':
          return (this.voltage > 100) ? '그을렸다' : '예쁘다';
       default:
          return '알 수 없다';
      }
    }
    
    get airSpeedVelocity() {
      switch (this.type) {
        case '유럽 제비':
          return 35;
       case '아프리카 제비':
          return 40 - 2 * this.numberOfCoconuts;
       case '노르웨이 파랑 앵무':
          return (this.isNailed) ? 0 : 10 + bird.voltage / 10;
       default:
          return null;
      }
    }
  }
  
  
  
  function airSpeedVelocity(bird) {
    switch (bird.type) {
      case '유럽 제비':
        return 35;
     case '아프리카 제비':
        return 40 - 2 * bird.numberOfCoconuts;
     case '노르웨이 파랑 앵무':
        return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
     default:
        return null;
    }
  }
```

