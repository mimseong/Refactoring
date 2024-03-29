# 8장

## 8.1 함수 옮기기

메서드 이동

좋은 소프트웨어 설계 핵심 : 모듈성

### 절차

1. 선택한 함수가 현재 컨텍스트 내에서 사용중인 모든 프로그램 요소 살펴보기. 함께 옮길 것들이 있는지 보기

   - 함께 옮겨야할 함수가 있다면 그 함수부터 옮기는게 나음. 여러게 옮겨야하면 영향이 적은 함수부터 옮기기
   - 하위 함수들의 호출자가 고수준 함수 하나면 하위 함수를 고수준함수에 인라인 하고, 고수준함수 옮기고 다시 개별함수로 추출

1. 선택한 함수가 다형 메서드인지 확인 (js에서는 쓸 일 없을 것 같은?)

1. 선택한 함수(소스 함수)를 타깃 컨텍스트로 복사한 후 다듬기
   - 함수 본문에서 소스 컨텍스트의 요소 사용 시, 파라미터로 옮기거나 소스컨텍스트 자체를 참조로 넘기기
1. 정적 분석

1. 소스 컨텍스트에서 타깃 함수를 참조할 방법을 찾아 반영

1. 소스 함수를 타깃함수의 위임함수가 되도록 수정

   - 바로 호출하지 말고 원래 함수 부분에서 옮긴 함수를 또 호출하도록

1. 테스트

1. 소스 함수를 인라인할지 고민해보기

- 소스 함수는 위임으로 남겨둘 수 있지만, 타깃함수를 직접 호출하는데 무리가 없다면 제거하는게 나음

---

## 8.2 필드 옮기기

캡슐화 하지 않은 레코드를 옮기는 법 : 접근자 함수들을 만들기 (get XX?), 모든 함수가 접근자를 거치도록 고치기 <- 이게 캡슐화랑 다른게 뭐지?

### 절차

1. 소스 필드 캡슐화하기

1. 테스트

1. 타깃 객체에 필드(와 접근자 메서드들) 생성

1. 정적 검사

1. 소스객체에서 타깃 객체를 참조 할 수 있는지 확인하기

1. 접근자들이 타깃필드를 사용하도록 수정하기

1. 테스트

1. (소스 필드 제거)

1. (테스트)

---

## 8.3 문장을 함수로 옮기기

<-> 문장을 호출한 곳으로 옮기기

중복코드 제거하기

### 절차

1. 반복 코드가 함수 호출 부분과 멀리 떨어져 있다면 문장 슬라이드하기를 이용해 근처로 옮기기

1. 타깃 함수를 호출하는 곳이 한곳이라면 단순하게 소스 위치에서 해당 코드를 복붙해서 테스트 하고 끝

1. 아니라면 호출자 중 하나에서 '타깃 함수 호출 부분이랑 그 함수로 옮기려는 문장들을 함께' 다른 함수로 추출하기 (이 함수 이름을 a라고 하면)

   ```js
   const Content = () => {
     const title = "title"; // 얘를 getContents 안으로 옮기고 싶음
     const getContents = () => {
       const subTitle = "~~";
       // ~~~~
       return ~~ ;
     };
     return (
       <div>
         {title} {getContents()}
         <div>
            ~~~~~
            <div>
                {title} {getContents()}
            </div>
         </div>
       </div>
     );
   };
   ```

1. 다른 호출자 모두가 방금 추출한 함수 a를 사용하도록 수정하기. 하나 수정하고 테스트 반복

   ```js
   // {title} {getContents()} => newGetContents() 이런식으로 변경
   const newGetContents = () => {
     const title = "title";
     return title + getContents();
   };
   ```

1. 모든 호출 자가 새로운 함수 a를 사용하게 되면, 원래함수를 새로운 함수 안으로 인라인 하고 원래 함수 제거

   ```js
   // {title} {getContents()} => newGetContents() 이런식으로 변경
   const newGetContents = () => {
       const title = "title";
       const subTitle = "~~";
       // ~~~~
       return ~~ ;
   };
   ```

1. 새함수 이름을 원래 함수 이름으로 바꾸기

---

## 8.4 문장을 호출한 곳으로 옮기기

<-> 문장을 호출한 곳으로 옮기기

### 절차

1. 한 두군데만 고쳐야한다면 복붙하고 테스트

1. 아니라면 이동하지 않길 원하는 모든 문장을 함수로 추출하고 쉬운이름 붙여주기

   ```js
   // {title} {getContents()} => newGetContents() 이런식으로 변경
   const getContents = () => {
     notMoved();
     const title = "title";
     // ~~~~
     return "~~";
   };

   const notMoved = () => {
     // 이동하지 않을 코드들
     const subTitle = "~~";
     const date = "~~";
     // ~~
     return "~~";
   };
   ```

1. 원래 함수를 인라인하기

   ```
   원래 호출 하던 곳이 getContents() 였다면
   그 부분을 {notMoved()} 랑 {title} 이런 식으로 변경?
   ```

1. 추출된 함수의 이름을 원래 함수의 이름으로 변경
   ```
    기존 getContents 지우고 notMoved이름을 getContents로 변경
   ```

---

## 8.5 인라인 코드를 함수 호출로 바꾸기

```js
let hasApple = false;
for (const item of fruits) {
  if (item === "apple") hasApple = true;
}
// 이 함수를 이렇게 변경하기
const hasApple = fruits.includes("apple");
```

### 절차

1. 인라인 코드를 함수 호출로 대체 하기

1. 테스트

---

## 8.6 문장 슬라이드 하기

조건문의 공통 실행코드 빼내기

비슷한 기능 하는 애들 끼리 모아두기?

### 절차

1. 코드 조각(문장들)을 이동할 목표 위치 찾기. 위치 변경 시 동작이 달라지는 코드가 있는지 확인하기.

   - 조건

     - 코드 조각에서 참조하는 요소를 선언하는 문장 앞으로는 이동 불가
     - 코드 조각을 참조하는 요소의 뒤로는 이동 불가
     - 코드 조각에서 참조하는 요소를 수정하는 문장을 건너 뛰어 이동 불가
     - 코드 조각이 수정하는 요소를 참조하는 요소를 건너 뛰어 이동 불가

       ```js
       if (isMinus) number *= -1;
       total = number + count;
       // 위와 같은 경우는 위치를 변경 할 수 없음
       ```

1. 코드 조각을 목표 위치에 잘라내기 붙여넣기

1. 테스트

---

## 8.7 반복문 쪼개기

반복문 내 두가지 일 하기 x

반복문 내 한가지 일만 하기

최적화 ... 반복문 쪼개기가 다른 더 강력한 최적화를 적용할 수 있는 길을 열어주기도 <- 그래도 포기가 안되는데..

### 절차

1. 반복문을 복제해 두개로 만들기

1. 반복문이 중복되어 생기는 부수효과를 파악해 제거하기

1. 테스트

1. 완료 시 각 반복문을 함수로 추출할지 고민

## 8.8 반복문을 파이프라인으로 바꾸기

```js
for (const item of items) {
  if (item.value === RUNNING) {
    running.push(item);
  }
}

items.filter((item) => {
  return item.value === RUNNING;
});

const restData = data?.filter(
  (item) => !firstData?.map((item) => item).includes(item)
);
```

### 절차

1. 반복문에서 사용하는 컬렉션을 가리키는 변수를 하나 만들기

1. 반복문의 첫 줄부터 시작해서 각각의 단위 행위를 적절한 컬렉션 파이프라인 연산으로 대체 하기. 연산의 결과를 기초로 연쇄적으로 수행

1. 반복문의 모든 동작을 대체 했다면 반복문 지우기

---

## 8.9 죽은 코드 제거하기

안 쓰는 코드 주석 처리 -> 삭제해도 버전 관리 시스템에서 이전 코드 찾을 수 있음

### 절차

1. 죽은 코드를 외부에서 참조할 수 있는 경우라면 혹시라도 호출 하는 곳이 있는지 확인하기

1. 없다면 죽은 코드 제거

1. 테스트
