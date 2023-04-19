# 10. 조건부 로직 간소화

## 10.1 조건문 분해하기

복잡한 조건부 로직 - 읽기 어려움

코드 분해 후 함수 호출로 변경하여 의도 드러내기

### 절차

1. 조건식과 그 조건식에 딸린 조건절 각각을 함수로 추출하기

   ```js
   if (!여름전 && !여름후) {
     // => 해당 조건을 isSummer()와 같이 변경
     // 이 안에 있는 로직도 함수로 추출
     charge = summerCharge();
   } else {
     // 이 안에 로직도 함수로 추출
     charge = regularCharge();
   }
   // 혹은 3항연산자
   charge = isSummer() ? summerCharge() : regularCharge();
   ```

## 10.2 조건식 통합하기

비교 조건은 상이하나 수행 동작은 같을 때 통합하기

조건부 코드 통합 이유

1. 나뉜 조건들을 통합함으로써 하려는 일이 명확해짐
1. 이 작업이 함수 추출하기 까지 이어질 수 있음 -> 함수 추출하기를 통해 의도가 분명한 코드가 될 수 있음

독립된 검사일 시 해당 리팩터링 하면 안됨

### 절차

1. 해당 조건식들 모드에 부수 효과가 없는지 확인하기
   - 부수효과가 있는 조건식들은 질의함수와 변경함수 분리하기 적용 (11.1절)
1. 조건문 두개를 선택해 두 조건문의 조건식들을 논리 연산자로 결합

   - 순차적으로 이뤄지는 (같은 레벨) 조건문은 or 결합
   - 중첩된 조건문은 and 결합

1. 테스트

1. 조건이 하나만 남을 때까지 반복

1. 하나로 합쳐진 조건식을 함수로 추출할지 고려하기

```js
if (이거면) return 0;
if (저거면) return 0;
if (그러면) return 0;

if (이거면 || 저거면) return 0;
if (그러면) return 0;
// 이런 식으로 순차적으로 조건 옮기기
```

```js
if (이거면) {
  if (저거면) return 0;
  return 1;
}

if (이거면 && 저거면) return 0;
return 1;
```

## 10.3 중첩 조건문을 보호 구문으로 바꾸기

참-거짓으로 분기가 아닌 정상만 처리하는 코드에서 if문에서 비정상 조건 검사하고 빠져나오는 걸 **보호 구문** 이라함

의도를 명확히 하기 위함

### 절차

1. 교체해야 할 조건 중 가장 바깥 것을 선택해 보호 구문으로 변경

1. 테스트

1. 반복

1. 모든 보호구문이 같은 결과를 반환 하면 보호 구문들의 조건식을 통합하기

```js
let result;
if (직원) {
  if (퇴사자) result = 0;
  else if (휴직자) result = 0;
  else result = getSalary();
}
// 위의 조건을 아래와 같이 변경
if (퇴사자) return 0;
if (휴직자) return 0;
return getSalary();
```

- 조건 반대로 만들기

```js
if (data?.resource) setResourceData(data.resource);
// 해당 조건을 아래와 같이 바꾸는?
if (!data?.resource) return;
setResourceData(data.resources);
```

## 10.4 조건부 로직을 다형성으로 바꾸기

### 절차

1. 다형적 동작을 표현하는 클래스들이 아직 없다면 팩터리 함수를 포함해 만들기
1. 호출하는 코드에서 팩토리 함수를 사용하도록 변경
1. 조건부 로직 함수를 슈퍼클래스로 옮기기
1. 서브 클래스 중 하나 선택하기. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드 오버라이드 하기. 조건부 문장 중 선택된 서브 클래스에 해당하는 조건절을 서브클래스 메소드로 복사한 후 수정
1. 같은 방식으로 각 조건절을 해당 서브클래스에서 메서드로 구현하기
1. 슈퍼 클래스 메서드에는 기본 동작 부분만 남기기.

## 10.5 특이 케이스 추가하기

널(null) 객체 패턴

### 절차

1. 컨테이너(데이터구조/클래스)에 특이 케이스 검사하는 속성 추가 후, false 반환하기

   ```js
   class Customer {
     get isUnknown() {
       return false;
     }
   }
   ```

1. 특이 케이스 객체 만들기. 이 객체는 특이 케이스인지 검사하는 속성만 포함. 이 속성은 true 를 반환하게 하기

   ```js
   class UnknownCustomer {
     get isUnknown() {
       return true;
     }
   }
   ```

1. 클라이언트에서 특이 케이스인지 검사하는 코드를 함수로 추출하기. 모든 클라이언트가 값을 직접 비교하는 대신 해당 함수를 사용하도록 수정

   ```js
   function isUnknown(arg) {
     if (!arg instanceof Customer || arg === "미확인고객")
       throw new Error("err");
     return arg === "미확인고객";
   }
   ```

1. 코드에 새로운 특이 케이스 대상을 추가하기. 함수의 반환값으로 받거나 변환함수를 적용하면 됨.
   ```js
   get customer(){
       return this.customer === '미확인고객' ? new UnknownCustomer():this.customer;
   }
   ```
1. 특이케이스를 검사하는 함수 본문을 수정하여 특이 케이스 객체의 속성을 사용하도록 하기

   ```js
   // 위에 선언헸던 함수 고치기
   function isUnknown(arg) {
     if (!arg instanceof Customer || !arg instanceof UnknownCustomer)
       throw new Error("err");
     return arg.isUnknown; // 얘는.. Customer면 false, UnknownCustomer면 true 리턴
   }
   ```

1. 테스트
1. 여러 함수를 클래스로 묶기나 여러함수를 변환함수로 묶기를 적용해 특이 케이스를 처리하는 공통 동작을 새로운 요소 옮기기.

   ```js
   // 클래스 UnknownCustomer 에도 Customer 처럼 get name 같은 거 선언해서

   const customerName = aCutomer.name;
   // 했을 때 알아서 Customer면 실제 이름, UnknownCustomer면 거주자불명(?) 같은 거 리턴하게 하기
   ```

1. 아직도 특이 케이스 검사 함수를 이용하는 곳이 있다면 검사함수를 인라인하기.

## 10.6 어서션 추가하기

특정 조건이 참일 때 작동하는 코드가 있을 때 어서션 추가

어서션은 항상 참이라고 가정하는 조건부 문장

오류 검출에도 쓰이지만, 프로그램이 어떤 상태로 가정돼있는지 알려주는 도구

### 절차

1. 참이라고 가정하는 조건이 보이면 그 조건을 명시하는 어서션을 추가
   - 어서션은 시스템 운영에 영향을 주지 않음 -> 추가한다고 동작이 달라지지 않음


## 10.7 제어 플래그를 탈출문으로 바꾸기

제어 플래그 : 코드의 동작을 변경하는데 사용되는 변수, 어딘가에서 값을 계산해온 후 다른 데서 조건으로 쓰임, 안쓰는게 좋음

ex. const isVisitor = false 선언한 후 visitor인지 판단하고 밑에서 if(isVisitor) 이런식으로 쓰는 거?

### 절차

1. 제어 플래그를 사용하는 코드를 함수로 추출할지 고려하기

1. 제어 플래그를 갱신하는 코드를 각각 적절한 제어문으로 바꾸기. 하나 바꾸고 테스트
   - 제어문은 주로 return, break, continue

1. 모두 수정했다면 제어 플래그 제거
