# 01. Execution context
- Execution context : **실행할 코드에 제공할 환경 정보들을 모아놓은 객체**

    JS의 동적 언어로서의 성격을 가장 잘 파악할 수 있는 개념

    클로저를 지원하는 대부분의 언어에서 이와 유사하거나 동일한 개념이 적용되어 있음


## 1. Stack

- 입구가 하나뿐인 우물
- 데이터 a, b, c, d 순서대로 저장했다면, 꺼낼 때는 d, c, b, a의 순서로 꺼냄
- 많은 프로그래밍 언어들은 stack이 넘칠 때 에러를 던짐

## 2. Queue

- 양쪽 모두 열려있는 파이프
- 종류에 따라 양쪽 모두 입출력이 가능한 큐도 있으나, 보통 한쪽은 출력만 다른쪽은 입력만 담당하는 구조
- 데이터 a, b, c, d 순서대로 저장했다면, 꺼낼 때도 a, b, c, d의 순서로 꺼냄

## 3. Execution context

- 동일한 환경에 있는 코드들을 실행할 때 필요한 환경 정보들을 모아 context를 구성하고, 이를 콜 스택(call stack)에 쌓아올림

    그리고 가장 위에 쌓여있는 context와 관련있는 코드들을 실행하는 식으로 전체 코드의 환경과 순서를 보장함

- 동일한 환경, 즉 하나의 execution context를 구성할 수 있는 방법: 전역공간, eval() 함수, **함수 실행** 등

- **Execution context / Call stack**
    - [1] JS 코드를 실행하는 순간, 전역 컨텍스트가 콜 스택에 담김

        전역 컨텍스트와 관련된 코드를 순차로 진행

    - [3] outer 함수 호출하면, js 엔진은 outer에 대한 환경 정보를 수집해서 outer 실행 컨텍스트를 생성 후 콜 스택에 담음

        콜 스택의 맨 위에 outer 실행 컨텍스트가 놓였으므로, 전역 컨텍스트의 실행을 일시중단 ⇒ outer 실행 컨텍스트와 관련된 코드(outer 함수 내부 코드)를 순차로 실행

    - [2] inner 함수가 호출되고, inner 함수의 실행 컨텍스트가 콜 스택의 가장 위에 담김

        outer와 관련된 코드 실행 일시중단 ⇒ inner 함수 내부의 코드를 순서대로 진행

    - inner 함수가 실행 종료 ⇒ inner 실행 컨텍스트가 콜 스택에서 제거 ⇒ outer 컨텍스트가 맨 위에 존재 ⇒ 중단했던 [2]의 다음 줄부터 실행

        outer 함수가 실행 종료 ⇒ outer 실행 컨텍스트가 콜 스택에서 제거 ⇒ 전역 컨텍스트가 맨 위에 존재 ⇒ 중단했던 [3]의 다음 줄부터 실행

        마지막 줄까지 실행 ⇒ 전역 공간에 더는 실행할 코드가 남아 있지 않아, 전역 컨텍스트도 제거 ⇒ 콜 스택에 아무것도 남지 않은 상태로 종료

    ```jsx
    //[1]

    var a = 1;

    function outer(){
      function inner(){
        console.log(a);  //undefined
        var a = 3;
      }
      inner();  //[2]
      console.log(a);  //1
    }

    outer();  //[3]
    console.log(a);  //1
    ```

- 이렇게 어떤 execution context가 활성화될 때, JS 엔진은 해당 context에 관련된 코드들을 실행하는 데 필요한 환경 정보들을 수집해서 execution context 객체에 저장

    이 객체는 JS 엔진이 활용할 목적으로 생성할 뿐 개발자가 코드를 통해 확인할 수는 없음

- 활성화된 execution context의 수집 정보
    1. `VariableEnvironment` : 현재 context 내의 식별자들에 대한 정보(environmentRecord) + 외부 환경 정보(outerEnvironmentReference)

        선언 시점의 LexicalEnvironment의 스냅샷으로, 변경 사항은 반영되지 않음

    2. `LexicalEnvironment` : 처음에는 VariableEnvironment와 같지만, 변경 사항이 실시간으로 반영
    3. `ThisBinding` : this 식별자가 바라봐야 할 대상 객체

# 02. VariableEnvironment

- Execution context를 생성할 때, VariableEnvironment에 정보를 먼저 담고, 이를 복사해서 LexicalEnvironment를 만들고, 이후에는 LexicalEnvironment를 주로 활용
- LexicalEnvironment와 다르게 최초 실행 시의 스냅샷을 유지
- environmentRecord와 outerEnvironmentReference로 구성 (자세한 내용은 아래에)

# 03. LexicalEnvironment

- 어휘적 환경, 정적 환경 등으로 불림
- "현재 context의 내부에는 a, b, c와 같은 식별자들이 있고, 그 외부 정보는 D를 참조하도록 구성되어 있다." (사전같은 느낌)

## 1. environmentRecord / hoisting

- environmentRecord : 현재 context와 관련된 코드의 식별자 정보들(매개변수의 이름, 함수 선언, 변수명 등)이 저장

    context 내부 전체를 처음부터 끝까지 쭉 훑어나가며 순서대로 수집

- 전역 실행 컨텍스트 : 변수 객체를 생성하는 대신 JS 구동 환경이 별도로 제공하는 global object(브라우저의 window, Node.js의 global 객체 등)를 활용. 이들은 JS 내장객체(native object)가 아닌 호스트객체(host object)로 분류됨
- 코드가 실행되지 전임에도 JS 엔진은 이미 해당 환경에 속한 코드의 변수명들을 모두 알고 있다는 것

    hoisting : 'JS 엔진은 식별자들을 최상단으로 끌어올려놓은 다음 실제 코드를 실행한다'고 간주하는 것

### Hoisting example 1

- `식별자들을 최상단으로 끌어올려놓은 다음 실제 코드를 실행한다` (실제 엔진은 이러한 변환 과정을 거치지는 않음)

1. 원본 코드

    ```jsx
    function a(x){
      console.log(x);  //1

      var x;
      console.log(x);  //1

      var x = 2;
      console.log(x);  //2
    }

    a(1)
    ```

2. arguments를 변수 선언/할당과 같다고 간주

    ```jsx
    function a(){
      var x = 1;
      console.log(x);  //1

      var x;
      console.log(x);  //1

      var x = 2;
      console.log(x);  //2
    }

    a()
    ```

3. hoisting

    `environmentRecord`는 현재 실행될 컨텍스트의 대상 코드 내 어떤 식별자들이 있는지에만 관심이 있고, 각 식별자에 어떤 값이 할당될 것인지는 관심이 없음

    따라서 호이스팅할 때 **변수명만 끌어올리고 할당 과정은 원래 자리에 남겨둠**

    ```jsx
    function a(){
      var x;
      var x;
      var x;

      x = 1;
      console.log(x);  //1
      console.log(x);  //1

      x = 2;
      console.log(x);  //2
    }

    a()
    ```

### Hoisting example 2

1. 원본 코드

    ```jsx
    function a(){
      console.log(b);  //b함수

      var b = 'bbb';
      console.log(b);  //'bbb'

      function b(){}
      console.log(b);  //'bbb'
    }

    a()
    ```

2. hoisting

    **변수는 선언부와 할당부를 나누어 선언부만 끌어올리는 반면, 함수 선언은 함수 전체를 끌어올림**

    (JS를 유연하고 배우기 쉬운 언어로 만들고자 하여, 함수를 선언한 위치와 무관하게 그 함수를 실행할 수 있게 되었지만 더 많은 혼란을 야기하기도 함)

    ```jsx
    function a(){
      var b;          //선언부만 끌어올림
      function b(){}  //함수 전체를 끌어올림

      console.log(b);  //b함수

      b = 'bbb';
      console.log(b);  //'bbb'
      console.log(b);  //'bbb'
    }

    a()
    ```

3. 함수 선언문을 함수 표현식으로 변경

    ```jsx
    function a(){
      var b;
      var b = function b(){}

      console.log(b);  //b함수

      b = 'bbb';
      console.log(b);  //'bbb'
      console.log(b);  //'bbb'
    }

    a()
    ```

### Function declaration vs Function expression

- **함수 선언문(function declaration):** function 정의부만 존재하고 별도의 할당 명령이 없음

    반드시 함수명이 정의되어야 함

- **함수 표현식(function expression):** 정의한 function을 별도의 변수에 할당

    함수명을 정의한 표현식은 '기명 함수 표현식', 정의하지 않은 것을 '익명 함수 표현식'이라고 부름

    일반적으로 함수 표현식은 '익명 함수 표현식'을 말함

```jsx
//function declaration
function a(){}
a();

//(익명) function expression
var b = function (){}
b();

//기명 function expression
var c = function d(){}
c();
d(); //에러! 함수명은 오직 함수 내부에서만 접근 가능
```

### 함수 선언문보다 함수 표현식을 사용하자

1. 원본코드

    ```jsx
    console.log(sum(1, 2));       //3
    console.log(multiply(3, 4));  //Uncaught Type Error: multiply is not a function

    //function declaration
    function sum(a, b){
      return a + b;
    }

    //function expression
    var multiply = function(a, b){
      return a * b;
    }
    ```

2. hoisting

    ```jsx
    var sum = function sum(a, b){    //함수 선언문 전체를 끌어올림
      return a + b;
    }

    var multiply;           //변수는 선언부만 끌어올림(함수도 하나의 값으로 취급)

    console.log(sum(1, 2));       //3
    console.log(multiply(3, 4));  //Uncaught Type Error: multiply is not a function

    multiply = function(a, b){       //변수의 할당부는 원래 자리에 남겨둠
      return a * b;
    }
    ```

3. hoisting-2

    ```jsx
    var sum;
    var multiply;

    sum = function sum(a, b){
      return a + b;
    }

    console.log(sum(1, 2));       //3
    console.log(multiply(3, 4));  //Uncaught Type Error: multiply is not a function

    multiply = function(a, b){
      return a * b;
    }
    ```

- 함수 선언문(function declaration)으로 함수들을 작성한다면 선언된 함수들이 전부 가장 위로 끌어올려진다. (함수들이 얼마나 멀리 떨어져있느냐에 상관없이)

    동일한 변수명에 서로 다른 값을 할당할 경우 나중에 할당한 값이 먼저 할당한 값을 override 한다.

    즉, 코드를 실행하는 중에 실제로 호출되는 함수는 맨 마지막에 선언된 함수뿐이다.

    ⇒ 함수 표현식(function expression)을 사용하면, 함수 선언 및 할당 코드 이전에 함수를 호출할 수 없으므로 상대적으로 안전하다.

- 전역공간에 함수를 선언하거나 동명의 함수를 중복 선언하는 경우는 없어야 한다.

    만에 하나 전역공간에 동명의 함수가 여럿 존재하는 상황이라 하더라도, 모든 함수가 함수 표현식으로 정의되어 있는 것이 상대적으로 안전하다.

## 2. outerEnvironmentReference / scope / scope chain

- Scope : 식별자에 대한 유효범위
- Scope chain : '식별자의 유효범위'를 안에서부터 바깥으로 차례로 검색해나가는 것 ⇒ 이것을 가능하게 하는 것이 LexicalEnvironment의 두 번째 수집 자료인 `outerEnvironmentReference`

- outerEnvironmentReference는 현재 호출된 함수가 `선언`될 당시의 LexicalEnvironoment를 참조

    오직 자신이 선언된 시점의 LexicalEnvironment만 참조하므로 가장 가까운 요소부터 차례대로만 접근 가능

    ⇒ 무조건 scope chain 상에서 가장 먼저 발견된 식별자에만 접근 가능

- 코드 상에서 어떤 변수에 접근할 때:

    현재 context의 LexicalEnvironment를 탐색

    ⇒ 발견하면 그 값을 반환

    ⇒ 발견하지 못할 경우 다시 `outerEnvironmentReference`에 담긴 LexicalEnvironment 탐색

    ⇒ 전역 컨텍스트의 LexicalEnvironment까지 탐색해도 해당 변수를 찾지 못하면 `undefined` 반환

- Scope chain example

    **변수 은닉화(variable shadowing)**: inner 함수 내부에서 a 변수를 선언했기 때문에, 전역 공간에서 선언한 동일한 이름의 a 변수에는 접근 불가

    ```jsx
    var a = 1;

    var outer = function(){
        var inner = function(){
            console.log(a);
            var a = 3;
        }
        inner();
        console.log(a);
    }

    outer();
    console.log(a);

    >>>
    --------------------------------------------------------------------------------------------
    |                                         | global context | outer context | inner context |
    --------------------------------------------------------------------------------------------
    | Lexical-    |     environmentRecord     |    a, outer    |     inner     |       a       |
    | Environment |---------------------------|----------------|--------------------------------
    |  (L.E)      | outerEnvironmentReference |                |   global L.E  |   outer L.E   |
    --------------------------------------------------------------------------------------------
    ```

- 스코프 체인 확인 (브라우저의 개발자 도구 콘솔)

    ```jsx
    var a = 1;

    var outer = function(){
        var b = 2;
        var inner = function(){
            console.log(b);
            debugger;             //console.dir(inner)
        };
        inner();
    };

    outer();
    ```

### Global variable vs Local variable

- 전역변수 : 전역 공간에서 선언한 변수
- 지역변수 : 함수 내부에서 선언한 변수는 무조건 지역변수
- 함수 선언문 대신 함수 표현식, 함수 표현식 대신 지역변수로 만드는 것이 더 안전 (함수를 지역변수로 선언하기 위해 외부에 또 다른 함수를 하나 더 만들기)

    ⇒ 코드의 안전성을 위해 가급적 전역변수 사용을 최소화하고자 노력하자

- 전역변수 사용의 최소화를 위한 방법들: 즉시실행함수(IFE), namespace, module pattern, sandbox pattern 등. 모듈 관리도구인 AMD, CommonJS, ES6의 모듈 등도 관련한 역할 수행

# 04. this

- 실행 컨텍스트의 `thisBinding`에는 this로 지정된 객체가 저장됨

    실행 컨텍스트 활성화 당시에 this가 지정되지 않은 경우 this에는 전역 객체가 저장됨

    그 밖에는 함수를 호출하는 방법에 따라 this에 저장되는 대상이 다름 (3장 참조)

# 05. Summary

- Execution context: 실행할 코드에 제공할 환경 정보들을 모아놓은 객체
- 실행 컨텍스트 객체는 활성화되는 시점에 `VariableEnvironment`, `LexicalEnvironment`, `ThisBinding`의 세 가지 정보를 수집
- `VariableEnvironment`와 `LexicalEnvironment`의 구성

    : 매개변수명, 변수의 식별자, 선언한 함수의 함수명 등을 수집하는 `environmentRecord` + 바로 직전 컨텍스트의 LexicalEnvironment 정보를 참조하는 `outEnvironmentReference`

- Hoisting : 코드 해석을 좀 더 수월하게 하기 위해 environmentRecord의 수집 과정을 추상화한 개념
