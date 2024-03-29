## 6.8 매개변수 객체 만들기

데이터 항목 여러개를 데이터 구조로 모아 매개변수로 만들기

### 절차

1. 적절한 데이터 구조 만들기

1. 테스트

1. 함수선언바꾸기 (매개변수 수정) 로 새 데이터 구조를 매개변수로 추가
    - const exampleFunction = (originalData1, originalData2, newDataObject) => { ... }

1. 테스트

1. 함수 호출 시 데이터 구조 인스턴스 넘기도록 수정
    - exampleFunction (originalData1, originalData2, newDataObject);
    - 하나 고치고 하나 테스트

1. 기존 매개변수를 사용하던 코드를 새 데이터 구조의 원소를 사용하도록 변경
    - exampleFunction (newDataObject);

1. 다 바꿨다면 기존 매개변수를 제거 하고 테스트


## 6.9 여러 함수를 클래스로 묶기

공통된 데이터를 사용하는 함수들을 새 클래스에 모으기
    - 클라이언트가 객체의 핵심 데이터를 변경 가능
    - 파생객체 일관되게 관리 가능
함수를 묶는 다른 방법 : 여러함수를 변환함수로 묶기

### 절차 

1. 함수들이 공유하는 공통 데이터 레코드를 캡슐화
    - 공통 데이터가 레코드 구조로 묶여있지 않다면 매개변수 객체 만들기를 통해 데이터를 하나로 만들기

1. 공통 레코드를 사용하는 함수 각각을 새 클래스로 옮기기 (함수 옮기기)

1. 데이터 조작 로직들은 함수로 추출해 새 클래스로 옮기기


## 6.10 여러 함수를 변환 함수로 묶기

원본데이터가 코드 안에서 갱신 시 클래스로 묶기

변환 함수로 묶을 시 가공 데이터를 새 레코드에 저장하므로 원본 데이터가 깨짐

```
function wonToDollar (money) {...}
function wonToEuro (money) {...}

--> function wonToMoney (money) {
const result = _.cloneDeep(money);
result .euro = wonToEuro(result);
result .dollar = wonToDollar(result);
... // 여기에 중첩 함수로 wonTo~ 함수들 깔려있고
return result ;
}
// 이런 느낌인가?
```

### 절차

1. 변환할 레코드를 입력 받아 값을 그대로 반환하는 변환함수 만들기
    - 깊은 복사로 처리

1. 묶을 함수 중 함수 하나를 골라 본문 코드를 변환 함수로 옮기고, 처리 결과를 레코드에 새 필드로 기록. 그 후 클라이언트 코드가 이 필드를 사용하도록 수정

1. 테스트

1. 반복


## 6.11 단계 쪼개기

서로 다른 두 대상을 다루면 쪼개기
다른 단계에 있는 코드들 쪼개기

### 절차

1. 두번 째 단계에 해당하는 코드를 독립 함수로 추출
    ```
    1. 양치하는 함수
    2. 세수하는 함수 <-- 이게 두번 째 단계 코드?
    ```

1. 테스트

1. 중간 데이터 구조를 만들어 앞에서 추출한 함수의 인수로 추가

1. 테스트

1. 추출한 두번 째 단계 함수의 매개변수를 하나씩 검토 (ex. 세수), 그 중 첫번째 단계에서 사용되는 것은 중간 데이터 구조로 옮김. 하나씩 옮기면서 테스트

1. 첫번째 단계코드를 함수로 추출하면서 중간 데이터 구조를 반환하도록 만들기
