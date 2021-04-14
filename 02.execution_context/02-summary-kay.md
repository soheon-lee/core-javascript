>**학습목표**
실행 컨텍스트를 정확히 이해하는 것.

## 실행 컨텍스트란?
>
**실행할 코드에 제공할 환경 정보들을 모아놓은 객체
**자바스크립트의 동적 언어로서의 성격을 가장 잘 파악할 수 있는 개념

#### 스택(stack)과 큐(queue)
![](https://images.velog.io/images/k904808/post/8b26be27-c4b4-44ba-9b38-287ce65786c3/image.png)
[이미지 출처](https://gohighbrow.com/stacks-and-queues/)

- **스택**은 출입구가 하나뿐이 깊은 우물 같은 데이터 구조.
저장한 순서의 반대로 꺼낼 수 밖에 없다.

- **큐**는 양쪽이 모두 열려있는 파이프 같은 구조.
보통은 한쪽은 입력만, 다른 한쪽은 출력만을 담당하는 구조. 
저장한 순서대로 꺼낼 수 밖에 없다.

#### 실행 컨텍스트
동일한 환경에 있는 코드들을 실행할 때 필요한 환경 정보를 모아 컨텍스트를 구성하고, 이를 콜 스택(call stack)에 쌓아올렸다가, 가장 위에 쌓여있는 컨텍스트와 관련 있는 코드들을 실행하는 식으로 전체 코드의 환경과 순서를 보장한다.

동일한 환경, 즉 하나의 실행 컨텍스트를 구성할 수 있는 방법으로 전역공간, eval() 함수, 함수 등이 있다.

#### 실행 컨텍스트와 콜 스택

```jsx
// -----(1)
var a = 1;
function outer() {
  function inner() {
      console.log(a);  // undefined
      var a = 3;
  }
  inner(); // -----(2)
  console.log(a); // 1
}
outer(); // -----(3)
console.log(a); //  1
```
![](https://images.velog.io/images/k904808/post/69fa2bfc-e889-46e4-b73e-5712559fabea/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-04-08%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%209.43.49.png)

처음 자바스크립트 코드를 실행하는 순간 (1) 전역 컨텍스트가 콜 스택에 담긴다.

전역 컨텍스트와 관련된 코드들을 순차로 진행하다가 (3)에서 outer 함수를 호출하면 자바스트립트 엔진은 outer에 대한 환경 정보를 수집해서 outer 실행 컨텍스트를 생성한 후 콜스텍에 담는다.
콜 스택의 맨 위에 outer 실행 컨텍스트가 놓인 상태가 됐으므로 전역 컨택스트와 관련된 코드의 실행을 일시 중단하고 대신 outer 실행 컨텍스트와 관련된 코드, 즉 outer 함수 내부의 코드들을 순차로 실행.

다시 (2)에서 inner 함수의 실행 컨텍스트가 콜 스택의 가장 위에 담기면 outer 컨텍스트와 관련된 코드의 실행을 중단하고 inner 함수 내부의 코드를 순서대로 진행.

![](https://images.velog.io/images/k904808/post/d4ad747f-8fbb-4776-ae10-8800b62f4106/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-04-08%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%209.42.02.png)

inner 함수 내부에서 a 변수의 값을 출력하고 나면 inner 실행 컨텍스트가 콜 스택에서 제거.

아래 있던 outer 컨택스트가 콜 스택의 맨 위에 존재하게 되므로 중단했던 (2)의 다음 줄 부터 이어서 실행.
a 변수의 값을 출력하고 나면 outer 함수의 실행이 종료되어 outer 실행 컨텍스트가 콜 스택에서 제거되고, 콜 스택에는 전역 컨텍스트만 남아있게 됨.

실행을 중단했던 (3)의 다음줄 부터 실행. a 변수의 값을 출력하고 나면 전역 공간에 더는 실행할 코드가 남아있지 않아 전역 컨텍스트도 제거되고, 콜 스택에는 아무것도 남지 않은 상태로 종료.

어떤 실행 컨텍스트가 활성화 될 때 자바스크립트 엔진은 해당 컨텍스트에 관련된 코드들을 실행하는데 필요한 환경 정보들을 수집해서 실행 컨텍스트 객체에 저장 (개발자가 코드로 확인할 수는 없음).

#### 활성화된 실행 컨텍스트의 수집 정보
- **VariableEnvironment**
현재 컨텍스트 내의 식별자들에 대한 정보 + 외부 환경정보. 
선언 시점의 LexicalEnviornment의 스냅샷으로 변경 사항 반영되지 않음.

- **LexicalEnvironment**
처음에는 VariableEnvioronment와 같지만, 변경 사항이 실시간으로 반영됨.

- **ThisBinding**
this 식별자가 바라봐야 할 대상 객체.

## VariableEnvironment
![](https://images.velog.io/images/k904808/post/a8c6cf8c-c37b-40ec-9bff-45ab743924da/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-04-08%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2010.18.18.png)

VariableEnvironment에 담기는 내용은 LexicalEnvironment와 같지만 최초 실행 시의 스냅샷을 유지한다는 점이 다름. 
실행 컨텍스트를 생성할 때 variableEnvironment에 정보를 먼저 담은 다음, 이를 그대로 복사해서 LexicalEnvironment를 만들고, 이후에는 LexicalEnvironment를 주로 활용하게 된다.

## LexicalEnvironment

### EnvironmentRecord와 호이스팅
>**EnvironmentRecord**
- **현재 컨텍스트와 관련된 코드의 식별자 정보들이 저장.**
- 전체 컨텍스트를 구성하는 함수에 지정된 매개변수 식별자, 선언한 함수가 있을 경우 그 함수 자체, var로 선언된 변수 식별자 등.

변수 정보를 수집하는 과정을 모두 마쳤더라도 아직 실행 컨텍스트가 관여할 코드는 실행되기 전의 상태.
자바스크립트 엔진은 식별자들을 최상단으로 끌어올려놓은 다음 실제 코드를 실행하는 것으로 간주할 수 있다.

#### 호이스팅 규칙
```jsx
//예제 1

function a(x) {
	console.log(x)  // (1)
  	var x;
  	console.log(x);  // (2)
  	var x = 2;
  	console.log(x);  // (3)
}
a(1);
```

```jsx
// 매개변수를 변수 선언/ 할당과 같다고 간주해서 변환한 상태

function a() {
  	var x = 1;
	console.log(x)  // (1)
  	var x;
  	console.log(x);  // (2)
  	var x = 2;
  	console.log(x);  // (3)
}
a();
```

```jsx
// 호이스팅을 마친 상태

function a() {
  	var x;
  	var x;
  	var x;
  
  	x = 1;
	console.log(x)  // (1)
  	console.log(x);  // (2)
  	x = 2;
  	console.log(x);  // (3)
}
a(1);
```
✔️ (1) 1, (2) 1, (3) 2가 출력됨

```jsx

// 예제 2
function a() {
	console.log(b);  // (1)
  	var b = 'bbb';
  	console.log(b);  // (2)
  	function b() {}
  	console.log(b);  // (3)
}
a();
}

// 호이스팅을 마친 상태

function a() {
	var b;  // 변수는 선언부만 끌어올린다.
  	function b() {}  // 함수는 전체를 끌어올린다.
  
  	console.log(b);  // (1)
  	b = bbb;  // 변수의 할당부는 원래 자리에 남겨둔다.
  	console.log(b);  // (2)
  	console.log(b);  // (3)
}
a();
```
✔️ (1) b함수, (2) 'bbb', (3) 'bbb'가 출력됨

#### 함수 선언문(function declaration)과 함수 표현식(function expression)

- 둘 모두 함수를 새롭게 정의할 때 쓰이는 방식
- **함수 선언문**은 function 정의부만 존재하고 별도의 할당 명령이 없음
- **함수 표현식**은 정의한 function을 별도의 변수에 할당하는 것을 말함. 일반적으로 함수 표현식은 익명 함수 표현식을 말함.

```jsx
function a () {} // 함수 선언문
a();
var b = function() {} // (익명)함수 표현식
b();
var c = function d () {} // 기명 함수 표현식 변수명: c 함수명: d
c(); // 실행 O
d(); // 에러!
```

>**기명 함수 표현식의 주의할 점**
외부에서는 함수명으로 함수를 호출할 수 없으며 오직 함수 내부에서만 접근할 수 있다.

```jsx
// 함수 선언문과 함수 표현식

console.log(sum(1,2));
console.log(multiply(3,4));

function sum(a, b) {  // 함수 선언문 sum
	return a + b;
}

var multiply = function (a, b) {  // 함수 표현식 multiply
	return a * b;
}

// 호이스팅을 마친 상태
var sum = function sum(a, b) {  // 함수 선언문은 전체를 호이스팅
	return a + b;
};
var multiply; // 변수는 선언부만 끌어올림
console.log(sum(1,2));
console.log(multiply(3,4));

multiply = function (a, b) {
	return a * b;
};

```
>상대적으로 함수 표현식이 안전하다.
원활한 협업을 위해서는 전역공간에 함수를 선언하거나 동명의 함수를 중복 선언 하는 경우는 없어야 한다.

### 스코프, 스코프 체인, outerEnvironmentReference
- 스코프(scope): 식별자에 대한 유효범위
- 스코프 체인(scope chain) 식별자의 유효범위를 안에서부터 바깥으로 차례로 검색해나가는 것

outerEnvironmentReference는 현재 호출된 함수가 **선언될 당시**의 LexicalEnvironment를 참조한다.

## this
this에는 실행 컨텍스트를 활성화하는 당시에 지정된 this가 저장됨. 
함수를 호출하는 방법에 따라 그 값이 달라지는데, 지정되지 않은 경우에는 전역 객체가 저장된다. 
