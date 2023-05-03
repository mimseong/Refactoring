

# Chapter 12 상속 다루기

## 12.1 메서드 올리기
> 상속 받은 클래스들에 중복되는 메서드가 있을 경우 상위 클래스로 메서드를 올리는 리팩터링

```js
//변경 전
class Emplyee {...}

class SalesPerson extends Employee {
  get name() {...}
}

class Engineer extends Employee {
  get name() {...}
}

//변경 후
class Employee {
  get name() {...}
}

class SalesPerson extends Employee {...}
class Engineer extends Employee {...}
```

**메서드 올리기를 적용하기 쉬운 상황**
- 메서드들의 본문 코드가 똑같을 떄

**메서드 올리기가 어려운 상황**
- 해당 메서드의 본문에서 참조하는 필드들이 서브클래스에만 있는 경우
- 이럴 경우 필드를 먼저 슈퍼클래스로 올린 후에 메서드를 올려야 한다.

절차
1. 똑같이 동작하는 메서드인지 살핀다.
  - 하는 일은 같지만 코드가 다르다면 본문 코드가 같아질 때까지 리팩터링한다.
2. 메서드 안에서 호출하는 다른 메서드와 참조하는 필드를 슈퍼클래스에서도 호출하고 참조할 수 있는지 확인한다.
3. 메서드 시그니처가 다르다면 함수 선언 바꾸기로 슈퍼클래스에서 사용하고 싶은 형태로 통일하낟.
4. 슈퍼클래스에 새로운 메서드를 생성하고, 대상 메서드의 코드를 복사해넣는다.
5. 정적 검사 수행
6. 서브클래스 중 하나의 메서드를 제거한다.
7. 테스트
8. 모든 서브클래스의 메서드가 없어질 때까지 다른 서브 클래스의 메서드를 하나씩 제거한다.


## 12.2 필드 올리기

- 서브클래스들이 독립적으로 개발되었거나 뒤늦게 하나의 계층구조로 리팩터링된 경우, 일부 기능, 특히 필드가 중복되었을 가능성이 높다.
- 비슷한 필드들이 비슷한 방식으로 쓰인다면 슈퍼클래스로 끌어올리자.

필드 올리기의 장점
- 데이터 중복 선언을 없앨 수 있다.
- 해당 필드를 사용하는 동작을 서브클래스에서 슈퍼클래스로 옮길 수 있다.

```java
//변경 전
class Employee {...}

class SalesPerson extends Employee {
  private String name;
}

class Engineer extends Employee {
  private String name;
}

//변경 후
class Employee {
  protected String name;
}
  
class SalesPerson extends Employee {...}
class Engineer extends Employee {...}
```

절차
1. 후보 필드들을 사용하는 곳 모두가 그 필드를 똑같은 방식으로 사용하는지 살핀다.
2. 필드들의 이름이 각기 다르다면 똑같은 이름으로 바꾼다.
3. 슈퍼클래스에 새로운 필드를 생성한다.
  - 서브클래스에서 이 필드에 접근할 수 있어야한다. (대부분 protected 사용)
4. 서브클래스의 필드들을 제거한다.
5. 테스트


## 12.3 생성자 본문 올리기
- 생성자는 일반 메서드와는 달리 할 수 있는 일과 호출 순서에 제약이 있기 때문에 다른 방식으로 접근해야한다.

**절차**
1. 슈퍼클래스에 생성자가 없다면 하나 정의한다. 서브 클래스의 생성자들에서 이 생성자가 호출되는지 확인한다.
2. 문장 슬라이드하기로 공통 문장 모두를 super() 호출 직후로 옮긴다.
3. 공통 코드를 슈퍼클래스에 추가하고 서브클래스들에서는 제거한다. 생성자 매개변수 중 공통 코드에서 참조하는 값들을 모두 super()로 건넨다.
4. 테스트한다.
5. 생성자 시작 부분으로 옮길 수 없는 공통 코드에는 함수 추출하기와 메서드 올리기를 차례로 적용한다.

*super(): super 키워드는 부모 오브젝트의 함수를 호출할 때 사용합니다.
생성자에서는 super 키워드 하나만 사용되거나 this 키워드가 사용되기 전에 호출되어야 합니다. 
또한 super 키워드는 부모 객체의 함수를 호출하는데 사용될 수 있습니다.



**예시**

```js
class Party {}

class Employee extends Party {
  contructor(name, id, monthlyCost) {
    super();
    this._id = id;
    this._name = name;
    this._monthlyCost = monthlyCost;
  }
  // 생략
}

class Department extends Party {
  contructor(name, staff) {
    super();
    this._name = name;
    this._staff = staff;
  }
  // 생략
}

```

`this._name = name;` 이라는 이름 대입 부분이 공통 코드이다.
따라서 Employee에서 이 대입문을 슬라이드하여 super() 호출 바로 아래로 옮긴다.


```js
class Employee extends Party {
  contructor(name, id, monthlyCost) {
    super();
    this._name = name;
    this._id = id;
    this._monthlyCost = monthlyCost;
  }
  // 생략
}
```

테스트가 성공하면, 이 공통 코드를 슈퍼클래스로 옮긴다.
```js
class Party {
  constructor(name) {
    this._name = name;
  }
}

class Employee extends Party {
  contructor(name, id, monthlyCost) {
    super(name);
    this._id = id;
    this._monthlyCost = monthlyCost;
  }
  // 생략
}

class Department extends Party {
  contructor(name, staff) {
    super(name);
    this._staff = staff;
  }
  // 생략
}

**예시: 공통 코드가 나중에 올 때**

아래 Manager 클래스에서 isPrivilged()는 grade 필드에 값이 대입된 후에야 호출될 수 있고, 서브클래스 만이 이 필드에 값을 대입할 수 있다.
따라서 super() 호출 후에 공통 코드가 나오게 된다.

```js
class Employee {
  constructor(name) {...}
  get isPrivilieged() {...}
  assignCar() {...}
}

class Manager extends Employee {
  constructor(name, grade) {
    super(name);
    this._grade = grade;
    if(this.isPriviliged_ this.assignCar(); // 모든 서브클래스가 수행한다.
  }
  get isPrivilieged() {
    return this._grade > 4;
  }
}
```

이럴 경우 공통코드를 함수로 추출한다.


```js
class Manager extends Employee {
  constructor(name, grade) {
    super(name);
    this._grade = grade;
    this.finishConstruction();
  }
  
  finishConstruction() {
    if(this.isPriviliged) this.assignCar();
  }
}
```

그런 다음 추출한 메서드를 슈퍼클래스로 옮긴다.(메서드 올리기)

```js
class Employee {
  constructor(name) {
    this._name = name;
    finishConstruction();
  }
  get isPrivilieged() {...}
  assignCar() {...}
  
  finishConstruction() {
    if(this.isPriviliged) this.assignCar();
  }
}

class Manager extends Employee {
  constructor(name, grade) {
    super(name);
    this._grade = grade;
  }
  get isPrivilieged() {
    return this._grade > 4;
  }
}
```

## 12.4 메서드 내리기

- 특정 서브클래스 하나와만 관련된 메서드를 슈퍼클래스에서 제거하고 해당 서브클래스에 추가하는 것이 깔끔하다.

절차
1. 대상 메서드를 모든 서브클래스에 복사한다.
2. 슈퍼클래스에서 그 메서드를 제거한다.
3. 테스트한다.
4. 이 메서드를 사용하지 않는 모든 서브클래스에서 제거한다.
5. 테스트한다.


## 12.5 필드 내리기

- 서브클래스 하나(혹은 소수)에서만 사용하는 필드는 해당 서브클래스로 옮긴다.

절차
1. 대상 필드를 모든 서브클래스에 정의한다.
2. 슈퍼클래스에서 그 필드를 제거한다.
3. 테스트한다.
4. 이 필드를 사용하지 않는 모든 서브클래스에서 제거한다.
5. 테스트한다.


## 12.6 타입 코드를 서브클래스로 바꾸기

- 타입 코드: 비슷한 대상을 특정 특성에 따라 구분하기 위한 코드

타입 코드 사용 시 서브클래스가 필요한 이유
- 조건에 따라 다르게 동작하도록 해주는 다형성 제공
  - 타입 코드에 따라 동작이 달라져야하는 함수가 여러 개일 때 유용
- 특정 타입에서만 의미가 있는 값을 사용하는 필드나 메서드가 있을 때 서브클래스를 만들어 사용 가능






