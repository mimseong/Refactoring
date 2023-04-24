
#  Chapter 11 API 리팩터링

- 모듈과 함수: 소프트웨어를 구성하는 빌딩 블록
- API: 블록들을 끼워맞추는 연결부

**미리보기**
- API의 기능이 명확하지 않다면: 질의 함수와 변경함수 분리하기
- 값 하나 때문에 함수들이 여러개로 나뉘었다면: 함수 매개변수화 하기
- 함수의 동작 모드를 전환하는 용도로만 쓰이는 매개변수가 있다면: 플래그 인수 제거하기
- 데이터 구조가 함수 사이를 건너다니며 필요이상으로 분해될 때는: 객체 통째로 넘기기
  - 혹은 매개변수를 질의함수로 바꾸기, 질의함수를 매개변수로 바꾸기

- 클래스(모듈)를 불변으로 만드려면: 세터 제거하기
- 호출자에 새로운 객체를 만들어 반환하려할 때: 생성자를 팩터리 함수로 바꾸기
- 많은 데이터를 받는 복잡한 함수를 객체로 변환하려할 때: 함수를 명령으로 바꾸기
- 그 명령 객체가 필요 없어졌을 때: 명령을 함수로 바꾸기


## 11.1 질의 함수와 변경함수 분리하기

- 외부에서 관팔할 수 있는 겉보기 부수효과가 없는 함수를 추구해야한다. 
- '질의함수(읽기 함수)는 모두 부수효과가 없어야 한다는 규칙을 따르자' -> 명령-질의 분리라고 한다.

- 값을 반환하면서 부수효과도 있는 함수는 상태를 변경하는 부분과 질의하는 부분을 분리하자.

### 절차
1. 대상 함수를 복제하고 질의 목적에 맞는 이름을 짓는다.
2. 새 질의 함수에서 부수효과를 모두 제거한다.
3. 정적 검사를 수행한다.
4. 원래 함수를 호출하는 곳을 모두 찾아낸다.
  - 호출하는 곳에서 반환값을 사용한다면 질의함수를 호출하도록 바꾸고, 원래 함수를 호출하는 코드를 바로 아래줄에 새로 추가한다.
5. 원래 함수에서 질의관련 코드를 제거한다.
6. 테스트

## 11.2 함수 매개변수화하기

- 두 함수의 로직이 아주 비슷하고 리터럴 값만 다른 경우, 그 다른 값만 매개변수로 받아 처리하는 함수로 합칠 수 있다.

```js
//변경 전
function tenPercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.05);
}

//변경 후
function raise(aPerson, factor) {
  aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

### 절차
1. 비슷한 함수 중 하나를 선택한다.
2. 함수 선언 바꾸기로 리터럴들을 매개변수로 추가한다.
3. 이 함수를 호출하는 곳 모두에 적절한 리터럴 값을 추가한다.
4. 테스트
5. 매개변수로 받은 값을 사용하도록 함수 본문을 수정한다.
6. 비슷한 다른 함수를 호출하는 코드를 찾아 매개변수화된 함수를 호출하도록 하나씩 수정한다.


### 예시

조금 복잡한 경우

```js
function baseCharge (usage) {
  if (usage > 0) return usd(0);
  const amount = 
        bottomBand(usage) * 0.03
         + middleBand(usage) * 0.05
         + topBand(usage) * 0.07;
  return usd(amount);
}

function bottomBand(usage) {
  return Math.min(usage, 100);
}

function middleBand(usage) {
  return usage > 100 ? Math.min(usage, 200) - 100 : 0;
}

function topBand(usage) {
  return usage > 200 ? usage - 200;
}
```

비슷한 함수를 매개변수화하여 통합할 때는 대상 함수 (bottomBand, middleBand, topBand) 중에 하나를 골라 매개변수를 추가한다.
위와 같이 범위를 다루는 로직의 경우에는 중간에 해당하는 함수에서 시작하는 것이 좋다.

대역의 상한, 하한을 이용해 계산하고 있으므로,  bottom, top 매개변수를 추가해주고
다른 대역을 계산하는 함수도 같이 바꿔준다.

```js
function withinBand(usage, bottom, top) {
  return usage > bottom ? Math.min(usage, top) - bottom : 0;
}

function baseCharge (usage) {
  if (usage > 0) return usd(0);
  const amount = 
        withinBand(usage, 0, 100) * 0.03
         + withinBand(usage, 100, 200) * 0.05
         + withinBand(usage, 200, Infinity) * 0.07;
  return usd(amount);
}
``
대역의 상한 호출을 대체할 때는 무한대를 뜻하는 Infinity를 이용하면 된다.
