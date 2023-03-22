# 07 캡슐화

## 7.1 레코드 캡슐화하기

### 절차

1. 레코드를 담은 변수를 캡슐화 하기
   ```js
   const fruit = {name:’apple’, amount : 2};
   const getTmpFruit = () => { return fruit };
   ```
1. 레코드를 감싼 단순한 클래스로 해당 변수의 내용을 교체하기. 이 클래스에 원본 레코드를 반환하는 접근자도 정의하고 변수를 캡슐화하는 함수들이 이 접근자를 사용하도록 수정
   ```js
   class Fruit {
     constructor(data) {
       this.data = data;
     }
   }
   ```
1. 테스트
1. 원본 레코드 대신 새로 정의한 클래스 타입의 객체를 반환하는 함수 만들기
   ```js
   const fruit = new Fruit({ name: "apple", amount: 2 });
   const getFruit = () => {
     return fruit;
   };
   ```
1. 레코드를 반환하는 예전 함수를 사용하는 코드를 4에서 만든 새 함수를 사용하도록 변경. 필드에 접근할 때는 객체의 접근자 사용. 적절한 접근자 없을 시 추가. 하나 변경 시마다 테스트
   ```js
   class Fruit {
     constructor(data) {
       this.data = data;
     }
     // 여기부터 새로 추가됨
     set name(aString) {
       this.data.name = aString;
     }
     set amount(aNumber) {
       this.data.amount = aNumber;
     }
     get name() {
       return this.data.name;
     }
     get amount() {
       return this.data.amount;
     }
   }
   // fruit의 name get/set 하고 싶으면 ..
   // 불변으로 사용하고 싶으면 setter 선언 안하면 되나?
   getFruit().name;
   getFruit().amount = 3;
   ```
1. 클래스에서 원본 데이터를 반환하는 접근자와 원본 레코드를 반환하는 함수 제거
1. 테스트
1. 레코드의 필드도 데이터 구조인 중첩 구조라면 레코드 캡슐화하기와 컬렉션 캡슐화하기를 재귀적으로 적용

---

## 7.2 컬렉션 캡슐화 하기

### 절차

1. 변수 캡슐화 하기

   ```js
   class Fruit {
     constructor(name) {
       this.name = name;
       this.countries = [];
     }
     get name() {
       return this.name;
     }
     get countries() {
       return this.market;
     }
     set countries(aList) {
       this.countries = aList;
     }
   }

   class Countries {
     constructor(name, isDomestic) {
       this.name = name;
       this.isDomestic = isDomestic;
     }
     get name() {
       return this.name;
     }
     get isDomestic() {
       return this.isDomestic;
     }
   }
   ```

1. 컬렉션에 원소 추가/제거하는 함수 만들기

   - 컬렉션 자체를 수정하는 setter 제거 혹은 인수로 받은 컬렉션 복제해 저장해놓기

   ```js
   class Fruit {
     constructor(name: string) {
       this.name = name;
       this.countries = [];
     }
     // 중략
     addCountry(aCountry: Countries) {
       this.countries.push(aCountry);
     }
     removeCountry(aCountry: Countries) {
       const idx = this.countries.find((item) => item.name === aCountry.name);
       this.countries.splice(idx, 1); // 첫번 째 인자로 들어온 인덱스부터 몇 개 삭제 하는지
     }
   }
   ```

1. 정적 검사 수행
1. 컬렉션을 참조하는 부분 모두 찾기. 컬렉션의 변경자를 호출하는 부분 모두 추가/제거 함수로 교체. 하나 수정하고 하나 테스트
1. 컬렉션 게터를 수정해 원본 내용을 수정할 수 없는 읽기 전용 프락시/복제본 반환

   ```js
   get countries(){return this.countries.slice();} // slice()를 통해 원본은 안 보냄
   ```

1. 테스트

---

## 7.3 기본형을 객체로 바꾸기

데이터 값을 객체로 전환
출력 이상의 기능이 필요할 시 데이터 표현 전용 클래스 정의

### 절차

1. 캡슐화 하기
1. 단순한 값 클래스 만들기. 생성자는 기존 값을 저장하고 게터 추가
1. 정적 검사
1. 값 클래스의 인스턴스를 새 만들어서 필드에 저장하도록 세터를 수정
1. 새 만든 클래스의 게터를 호출한 결과를 반환도록 게터 수정
1. 테스트
1. 함수이름을 바꾸면 원본 접근자의 동작을 더 잘드러낼 수 있는지 확인

---

## 7.4 임시 변수를 질의 함수로 바꾸기

긴 함수의 한 부분을 별도의 함수로 추출 할 때 변수들을 함수로 바꿔놓으면 편함 (-> 추출한
함수에 변수를 전달 따로 안 하고 호출하면 되니까?)

임시 변수 -> 질의 함수 변경이 무조건 좋은 건 아님

### 절차

1. 변수가 사용되기 전에 값이 확실히 결정 되는지, 변수를 사용할 때마다 계산 로직이 매번 다른 결과를 내지 않는지 확인
1. 읽기 전용으로 만들 수 있는 변수는 읽기 전용으로 만들기
   ```js
   // ex. 원가는 고정이니까 읽기전용 const, 정가는 변하니까 let
   const cost = a * b + c;
   let price = cost * margin;
   ```
1. 테스트
1. 변수 대입문을 함수로 추출
   ```js
   get getCost (){
     return a * b +c;
   }
   const cost = getCost();
   ```
1. 테스트
1. 변수 인라인하기로 임시 변수 제거
   ```js
   get getCost (){
     return a * b +c;
   }
   let price  = getCost() * margin;
   ```

## 7.5 클래스 추출하기

<-> 클래스 인라인 하기

약간,, 디비 테이블 복잡해지면 쪼개고 쪼개는 그런 느낌?

```js
/**
 ex.
 학생 - 학번, 이름, 수업 (단과대, 강의명, 교수, 시간)
 --> 수업 - 단과대, 강의명, 교수, 시간 쪼개기
 */
```

### 절차

1. 클래스의 역할을 분리할 방법을 정하기
1. 분리될 역할을 담당할 클래스 새로 만들기
1. 원래 클래스의 생성자에서 새로운 클래스의 인스턴스를 생성하여 필드에 저장
1. 분리될 역할에 필요한 필드들을 새 클래스로 옮긴다. 하나 옮기고 테스트 반복
1. 메서드들도 새 클래스로 옮기기. 호출 많이 당하는 애들부터 옮기기. 하나 옮기고 테스트 반복
1. 양쪽 클래스의 인터페이스를 살펴보며 불필요한 메서드 제거, 이름도 알맞게 변경
1. 새 클래스를 외부로 노출할지 결정. 노출 시 새 클래스에 참조를 값으로 바꾸기 를 적용할지 고민

---

## 7.6 클래스 인라인하기

<-> 클래스 추출하기

### 절차

1. 소스 클래스의 각 퍼블릭 메서드에 대응하는 메서드들을 타킷 클래스에 새성
1. 소스 클래스의 메서드를 사용하는 코드를 모드 타킷 클래스의 위임 메서드를 사용하도록 변경
1. 소스 클래스의 메서드와 필드를 모드 타깃 클래스로 옮김
1. 소스 클래스를 삭제

---

## 7.7 위임 숨기기

<-> 중개가 제거하기

클라이언트에서 위임 객체 호출하는데
인터페이스가 변경 되면 클라이언트에서 모든 호출 코드 수정 해야함

이러한 의존성을 제거하기위해 위임을 숨기면 위임 객체가 수정되더라도 클라이언트에는 영향이 없음

```js
const manager = aEmp.department.manager; // client 원래 이렇게 호출

class Emp {
  get manager() {
    return this.department.manager; // 위임 숨김
  }
}
const manager = aEmp.manager; // client
// department.manager에서 department.part.manager 로 바뀌어도 클라이언트는 변화 없음
```

### 절차

1. 위임객체의 각 메서드에 해당하는 위임 메서드를 서버에 생성
1. 클라이언트가 위임 객체 대신 서버를 호출하도록 수정. 하나 바꾸고 하나 테스트
1. 모두 수정 했다면 서버로부터 위임 객체를 얻는 접근자 제거

   ```js
   class Emp {
     // ...
     get department() {
       // 이걸 제거해서 manager로만 접근 가능하도록 수정
       return this.department;
     }

     get manager() {
       return this.department.manager;
     }
   }
   ```

1. 테스트

---

## 7.8 중개자 제거하기

<-> 위임 숨기기

위임 숨긴게 너무 많으면?..

### 절차

1. 위임 객체를 얻는 게터를 만든다
1. 위임 메소드를 호출하는 클라이언트가 모드 이 게터를 거치도록 수정하기. 하나 고치고 테스트
1. 모두 수정했다면 위임 메서드 삭제

---

## 7.9 알고리즘 교체하기

기존 복잡한 알고리즘을 간단한 방법으로 변경

ex. 길게 for, if 등등 써서 필터링 하는 걸 .filter(item => ~) 과 같이 변경?

### 절차

1. 교체할 코드를 함수 하나에 모으기
1. 이 함수만을 이용해 동작을 검증하는 테스트 마련
1. 대체할 알고리즘 준비
1. 정적 검사 준비
1. 기존 알고리즘과 새 알고리즘의 결과를 비교한느 테스트 수행. 두 결과가 같을 시 끝. 아니라면 다시 수정, 테스트, 디버깅

---
