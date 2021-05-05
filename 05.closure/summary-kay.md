> **학습목표:**
클로저의 일반적인 정의로부터 그 의미를 파악하고 다양한 사례를 통해 성질을 살펴본 후, 마지막에 다시 재조합해서 이해하기 쉬운 문장으로 바꿔볼 수 있다.

## 01 클로저의 의미 및 원리 이해
클로저(Closure)는 여러 함수형 프로그래밍 언어에서 등장하는 보편적인 특성
>
"A closure is the combination of a function and the lexical environment within which that function was declared. - **MDN(Mozilla Developer Network)** "
👉 클로저는 함수와 그 함수가 선언될 당시의 Lexical environment의 상호관계에 따른 현상
👉 어떤 함수에서 선언한 변수를 참조하는 내부 함수에서만 발생하는 현상

```js
var outer = function() {
	var a = 1;
  	var inner = function () {
    	return ++a;
    };
  	return inner; // inner 함수 자체를 반환
};
var outer2 = outer();
console.log(outer2());  // 2
console.log(outer2());  // 3
```
> 클로저란 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상

## 02 클로저와 메모리 관리
클로저는 어떤 필요에 의해 의도적으로 함수의 지역변수를 메모리를 소모하도록 함으로써 발생.
필요성이 사라진 시점에서 참조 카운트를 0으로 만들면 언젠가 GC가 수거해갈 것이고 소모됐던 메모리가 회수.
참조 카운트를 0으로 만드려면, 식별자에 참조형이 아닌 기본형 데이터 (보통 null, undefined)를 할당하면 된다.

```js
var outer = (function () {
	var a = 1;
  	var inner = function () {
    	return ++a;
    };
  	return inner;
})();
console.log(outer());
console.log(outer());
outer = null;  // outer 식별자의 inner 함수 참조를 끊음
```

## 03 클로저 활용 사례
### 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때
#### 콜백 함수를 내부함수로 선언해서 외부 변수를 직접 참조
```js
...
var alertFruit = function (fruit) {
   	alert('your choice is' + fruit);
};
fruits.forEach(function (fruit) {
	var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click', alertFruit(fruit));
    $li.appendChild($li);
});
document.body.appendChild($ul);
alertFruit(fruits[1]);
```

#### Bind
```js
...
fruits.forEach(function (fruit) {
	var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click', alertFruit.bind(null, fruit));
    $li.appendChild($li);
});
...
```
#### 콜백함수를 고차함수로 바꿔서 클로저를 적극적으로 활용
```js
...
var alertFruitBuilder = function (fruit) {
	return function() {
    	alert('your choice is' + fruit);
    };
};
fruits.forEach(function (fruit) {
	var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click', alertFruitBuilder(fruit));
    $li.appendChild($li);
});
...
```
### 접근 권한 제어(정보 은닉)
> 정보 은닉 (information hiding)은 어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서 모듈 간의 결합도를 낮추고 유연성을 높이고자 하는 현대 프로그래밍 언어의 중요한 개념 중 하나.

#### 접근 권한
- public
- private
- protected

자바스크립트는 기본적으로 변수 자체ㅐ에 이러한 접근 권한을 직접 부여하도록 설계돼 있지 않음. 
👉 클로저를 이용하면 함수 차원에서 public한 값과 privite한 값을 구분하는 것이 가능.

```jsx
var outer = function() {
	var a = 1;
  	var inner = function() {
    	return ++a;
    };
  	return inner;
};
var outer2 = outer();
console.log(outer2());
console.log(outer2());

// return을 활용한 클로저로 외부 스코프에서 함수 내부의 변수들 중 선택적으로 일부의 변수에 대한 접근 권한을 부여할 수 있다.
// 즉 외부에 제공하고자 하는 정보들을 모아서 return하고 내부에서만 사용할 정보들은 return 하지 않는 것으로 접근 권한 제어가 가능.
```

### 부분 적용 함수
> **부분 적용 함수(partially applied function)란?**
n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게끔 하는 함수.

#### bind 메서드를 활용한 부분 적용 함수
```js
var add = function () {
	var result = 0;
    for (var i = 0l i < arguments.length; i++) {
    	result += arguments[i];
    }
    return result;
};
var addPartial = add.bind(null, 1, 2, 3, 4, 5);
console.log(addPartial(5, 6, 7, 8, 9, 10);  // 55
```

### 커링 함수
>**커링 함수(currying function)란** 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성한 것.

- 커링은 한 번에 하나의 인자만 전달하는 것을 원칙으로 한다.
- 중간 과정상의 함수를 실행한 결과는 그다음 인자를 받기 위해 대기만 할 뿐으로 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않는다. 
(부분 적용 함수는 여러 개의 인자를 전달할 수 있고, 실행 결과를 재실행할 때 원본 함수가 무조건 실행)

```jsx
var curry3 = function (func) {
  return function (a) {
   	return function (b) {
      return func(a, b);
    };
  };
};

var getMaxWith10 = curry3(Math.max)(10);
console.log(getMaxWith10(8));  // 10
console.log(getMaxWith10(25));  // 25

var getMinWith10 = curry3(Math.min)(10);
console.log(getMinWith10(8));  // 8
console.log(getMinWith10(25));  // 10
```

## 04 정리
- 클로저란 어떤 함수에서 선언한 변수를 참조하는 내부함수를 외부로 전달할 경우, 함수의 실행 컨텍스트가 종료된 후에도 해당 변수가 사라지지 않는 현상.

- 내부함수를 외부로 전달하는 방법에는 함수를 return 하는 경우 뿐 아니라 콜백으로 전달하는 경우도 포함.

- 클로저는 그 본질이 메모리를 계속 차지하는 개념이므로 더는 사용하지 않게 된 클로저에 대해서는 메모리를 차지하지 않도록 관리해줄 필요가 있음.

- 클로저는 이 책에서 소개한 활용 방안 외에도 다양한 곳에서 활용할 수 있는 중요한 개념.
