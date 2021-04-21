다른언어와 자바스크립트 this 의 차이점

대부분 객체지향언어의 this : 클래스로 생성한 인스턴스 객체로 클래스에서만 사용할 수 있다
자바스크립트의 this : 어디서든 사용 가능하며 상황에 따라 this가 바라보는 대상이 달라진다.

따라서 정확한 작동 방식을 이해해야 의도한 대상을 가리키게 할 수 있다.
함수와 객체(메서드)의 구분이 느슨한 자스에서 this는 이 둘을 구분하는 실질적으로 거의 유일한 기능

 

 

# 01 상황에 따라 달라지는 this

this는 실행컨텍스트가 생성될 때(함수를 호출할 때) 함께 결정 => this는 함수를 호출할 때 결정된다.
함수를 어떤 방식으로 호출하냐에 따라 값이 달라짐

## 1. 전역 공간에서의 this

전역 컨텍스트를 생성하는 주체가 전역 객체이기 때문에 전역공간에서 this는 전역 객체를 가리킨다.
전역 객체는 자바스크립트 런타임 환경에 따라 다른 이름과 정보를 가진다.
(브라우저 환경에서 전역객체는 window이고 Node.js 환경에서는 global이다.)

`console.log(this === window);` 

전역변수를 선언하면 자바스크립트 엔진은 이를 전역객체의 프로퍼티로 할당한다.  => 변수이면서 객체의 프로퍼티
자바스크립트의 모든 변수는 특정 객체의 프로퍼티로서 동작하여 1을 출력하게 된다.

```js
var a = 1;

// 모두 1
console.log(a);
console.log(window.a);
console.log(this.a);
``` 

사용자가 var 연산자를 이용, 변수를 선언할 때 실제 자바스크립트 엔진은 어떤 특정 객체의 프로퍼티로 인식하는 것
특정 객체는 실행 컨텍스트의 LexicalEnvironment로 실행컨텍스트는 변수를 수집해서 L.E의 프로퍼티로 저장한다.
이후 어떤 변수를 호출하면 L.E를 조회해서 일치하는 프로퍼티가 있을 경우 그 값을 반환하며 전역 컨텍스트의 경우 L.E는 전역객체를 그대로 참조하는 것. 더 정확하게는 GlobalEnv가 전역 객체를 참조하는데 전역 컨텍스트의 L.E가 GlobalEnv를 참조함

 

_※ a를 직접 호출할 때도 1이 나오는 이유?_

변수 a에 접근하고자 하면 스코프 체인에서 a를 검색하다가 가장 마지막에 도달하는 전역 스코프의 L.E, 즉 전역객체에서 해당 프로퍼티 a를 발견해서 그 값을 반환한다. 편하게 window.가 생략된다고 받아들여도 된다. 
따라서 전역 공간에서는 var로 변수를 선언하는 대신 window의 프로퍼티에 직접 할당하더라도 똑같이 동작한다.

그러나 삭제의 경우 다른 결과를 불러올 수 있다. c, d처럼 처음부터 전역객체의 프로퍼티로 할당한 경우는 삭제가 되나 
전역변수로 선언한 경우 삭제가 되지 않는다. 혹시나 사용중 실수로 삭제할 것에서 방어하는 것으로 전역변수를 선언하면 자바스크립트엔진이 자동으로 전역객체의 프로퍼티로 할당하면서 해당 프로퍼티의 configurable 속성 (변경 삭제 가능성)을 false로 정의한다. 

```js
var a = 1;
delete window.a; //false
console.log(a, window.a, this.a); //1 1 1

var b = 2;
delete b; //false  (window.b)
console.log(b, window.b, this.b); //2 2 2

window.c = 3;
delete window.c; //true
console.log(c, window.c, this.c); //Uncaught ErferenceError : c is not defined

window.d = 4;
delete d; //true
console.log(d, window.d, this.d); //Uncaught ErferenceError : D is not defined
```

delete 연산자는 객체의 속성을 제거합니다. 제거한 객체의 참조를 어디에서도 사용하지 않는다면 나중에 자원을 회수합니다.

-> var로 선언한 전역변수와 전역객체의 프로퍼티는 호이스팅 여부 및 configurable 여부에서 차이를 보인다.

## 2. 메서드로 호출할때 메서드 내부에서의 this

함수를 실행하는 일반적인 방법은 함수로서 호출하는 경우와 메서드로 호출 하는 경우가 있다.
함수와 메서드는 미리 정의한 동작을 수행하는 코드 뭉치로 이 둘의 차이는 독립성에 있다.
함수는 그 자체로 독립적인 기능을 수행하며, 메서드는 자신을 호출한 대상 객체에 관한 동작을 수행한다.


초보의 경우 메서드를 객체의 프로퍼티에 할당된 함수로 이해하나 
어떤 함수를 객체의 프로퍼티에 할당한다 해서 무조건 메서드가 되지는 않고
객체의 메서드로서 호출할 경우에만 메서드로 동작하고, 아니면 함수로 동작한다.


## 3. 함수로서


함수 호출이 발생할 때 상황에 따라 this가 자동으로 바인딩 되는 것을 알 수 있다.
자바스크립트는 내부적 this바인딩 외에도 명시적 this바인딩시키는 방법을 제공한다.

 

# 02 명시적으로 this를 바인딩하는 방법
앞의 규칙 외에 this에 별도의 대상을 바인딩 하는 방법도 있습니다.
이 메서드들은 모든 함수의 부모 객체인 Function.prototype 객체의 메서드이므로 모든함수에서 호출가능

 

2-1 call 메서드 
call 메서드는 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령

Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
 

(1) 함수를 실행할 때


이 때 call 메서드의 첫 번째 인자를 this로 바인딩하고, 이후의 인자들을 호출할 함수의 매개변수로 함
함수를 그냥 실행하면 this는 전역객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있다.

 

var func = function(a,b,c) {
	console.log(this, a, b, c);
};

func(1,2,3); // Window  1 2 3  ----this가 window
func.call({x:1}, 4,5,6); // Object 4 5 6   ---- this가 {x:1}
 

실습하기 : https://jsfiddle.net/

 

(2) 메서드를 실행할 때 

메서드일 경우도 객체의 메서드를 그냥 호출하면 this는 객체를 참조하지만 call메서드를 사용하면 임의의 객체를 this로 지정할 수 있다.

 

var obj = {
	a: 1,
	method: function (x,y) {
		console.log(this.a, x, y);
	}
};

obj.method(2, 3);  // 1 2 3
obj.method.call({a:4}, 5, 6); // 4 5 6
 

 

 

 

2-2 apply 메서드
Function.prototype.apply(thisArg[, argsArray])
 

call 메서드와 기능적으로 완전히 동일하나 call 메서드는 첫 번째 인자를 제외한 나머지 모든 인자들을 호출할 함수의 매개변수로 지정하는 반면, apply 메서드는 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정한다.

 

var func = function(a,b,c) {
	console.log(this, a, b, c);
};


func.apply({x:1}, [4,5,6]); //{x:1} 4 5 6

var obj = {
	a:1,
	method: function(x,y) {
		console.log(this.a, x, y);
	}
};
obj.method.apply({a:4}, [5,6]); //4 5 6
 

실습하기 : https://jsfiddle.net/

 

 

 

 

2-3 call / apply 메서드의 활용
1)유사배열객체에 배열메서드 적용, 2)생성자 내부에서 다른 생성자를 호출,
3)여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 사용한다.
 

1. 유사배열객체에 배열 메서드를 적용

 배열의 구조와 유사한 유사배열객체에 call / apply 메서드를 이용해 배열 메서드를 사용한다.

 

유사배열 객체란? array-like object

key가 0이나 양의 정수인 프로퍼티가 존재하고
length 프로퍼티의 값이0 또는 양의 정수인 객체
객체에는 배열 메서드를 직접 적용할 수 없다.



유사배열과 배열 차이

Array.isArray(이름); 을 돌려서 true/false 값으로 확인
유사배열 즉 객체의 경우 배열의 메서드를 쓸 수 없다.
 

배열 메서드인 push를 객체 obj에 적용해 프로퍼티 3에 'd'를 추가
slice메서드를 적용해 객체를 배열로 전환. call 메서드를 이용해 원본인 유사배열객체의 얕은 복사를 수행, slice가 배열 매서드이기 때문에 복사본을 배열로 반환

 

1) 유사배열객체에 배열 메서드를 적용

 

var obj = {
	0:'a',
	1:'b',
	2:'c',
	length : 3
};
Array.prototype.push.call(obj, 'd');  // push : 마지막에 추가
console.log(obj); // { 0:'a', 1:'b', 2:'c', 3:'d', length : 4}

var arr = Array.prototype.slice.call(obj);  // slice : 배열의 일부를 선택하여 새로운 배열을 만듦
console.log(arr); // ['a', 'b', 'c', 'd']
 

실습하기 : https://jsfiddle.net/

 

 

# 예시에 나온 새로운 함수
push() 메서드

배열의 끝에 하나 이상의 요소를 추가하고, 배열의 새로운 길이를 반환 참고
slice() 메서드

시작 인덱스값과 마지막 인덱스값을 받아 시작값부터 마지막값의 앞부분까지의 배열 요소를 추출하는 메서드
매개변수를 아무것도 넘기지 않으면 어떤 배열의 begin부터 end까지(end 미포함)에 대한 원본 배열의 얕은 복사본을 반환합니다. (원본 배열은 수정되지 않습니다.) 참고
jbAry.slice( 2, 5 ); // start, end

// jbAry 배열의 3번째 요소부터 5번째 요소까지 선택합니다.
// end에 값이 없으면 해당 배열의 마지막 요소까지 선택합니다. 
// 값이 음수면 마지막 요소를 기준으로 선택합니다.
 

 


2) arguments, NodeList에 배열 매서드를 적용

함수 내부에서 접근할 수 있는 arguments 객체도 유사배열객체이므로 위 방법으로 배열로 전환해서 활용할 수 있다. querySelectorAll, getElementsByClassName 등의 Node 선택자의 선택 결과인 NodeList도 마찬가지입니다.


function a() {
    var argv = Array.prototype.slice.call(arguments);
    console.log(arguments);  // length: 3 __proto__: Object 0: 1 1: 2 2: 3
    console.log(argv);  // length: 3 __proto__: Array(0)

    argv.forEach(function (arg) {
       console.log(arg);  //1 2 3
    });
}
a(1,2,3);

document.body.innerHTML = '<div>a</div><div>b</div><div>c</div>';  //뿌리기

var nodeList = document.querySelectorAll('div'); 
console.log(nodeList);  // length: 3 __proto__: NodeList

var nodeArr = Array.prototype.slice.call(nodeList);
console.log(nodeArr);  // length: 3 __proto__: Array(0)

nodeArr.forEach(function (node) {
	console.log(node);   // 세 개의 div 태그 안에 각각 감싸진 a b c 
});
 

실습하기 : https://jsfiddle.net/

 


 

 

 


3) 문자열에 배열 메서드 적용

 

배열처럼 인덱스와 length 프로퍼티를 지니는 문자열도 call/apply 메서드를 이용해서 배열 메서드를 적용 가능하나 문자열의 경우 length 프로퍼티는 읽기전용이라서 원본 문자열에 변경을 가하는 메서드인 push, pop, shift, unshift, splice 등은 에러가 나며 concat 처럼 대상이 반드시 배열이어야만 하는 경우에도 에러는 안나도 제대로된 결과를 뱉지 않음

 

var str = 'abc def';

Array.prototype.push.call(str, ', pushed string'); 
// Error : Cannot assign to read only pro

Array.prototype.concat.call(str, 'string'); //[String {"abc def"}, "string"]

Array.prototype.every.call(str, function(c) {return char !== ''; }); //false

Array.prototype.some.call(str, function(c) {return char !== ''; }); //true


var newArr = Array.prototype.map.call(str, function(c) {return char + '!'; });
console.log(newArr);

var newStr = Array.prototype.reduce.apply(str, [function(string, char, i) 
	{return string + char + i;}, '' ]);
console.log(newArr);
 

실습하기 : https://jsfiddle.net/

 

사실 call/apply 방법으로 형변환하는 것은 this를 원하는 값으로 지정해서 호출한다는 본래의 메서드 의도와는 다소 동떨어진 활용법이며 slice 메서드는 오직 배열 형태로 복사하기 위해서 사용됐다.

 

 

4) ES6의 Array.from 메서드

경험을 통해 뜻을 아는 사람이 아닌 이상 딱 봐선 모를 수 있으니ES6에서는 유사배열객체 또는 순회 가능한 모든 종류의 데이터타입을 배열로 전환하는Array.from 메서드를 새로 도입했다.

Array.from() 메서드는 유사 배열 객체(array-like object)나 반복 가능한 객체(iterable object)를 얕게 복사해 새로운 Array 객체를 만듭니다.  참고

 

var obj = {
	0: 'a',
	1: 'b',
	2: 'c',
	length: 3
};
var arr = Array.from(obj);
console.log(arr);  // ['a', 'b', 'c']
 

다음과 같은 경우에 Array.from()으로새 Array를 만들 수 있습니다.

유사 배열 객체 (length 속성과 인덱싱된 요소를 가진 객체)

순회 가능한 객체 (Map, Set 등객체의 요소를 얻을 수 있는 객체)

 

console.log(Array.from('foo'));
// expected output: Array ["f", "o", "o"]

console.log(Array.from([1, 2, 3], x => x + x));
// expected output: Array [2, 4, 6]
 

실습하기 : https://jsfiddle.net/

 

 

 

2) 생성자 내부에서 다른 생성자를 호출

생성자 내부에 다른 생성자와 공통된 내용이 있을 시 call/apply를 이용해 다른 생성자를 호출하면 반복을 줄일 수 있다. 예문에서는 생성자 함수 내부에서 다른 생성자 함수를 호출해서 인스턴스의 속성을 정의했다

 

function Person(name, gender) {
	this.name = name;
	this.gender = gender;
}
function Student(name, gender, school) {
	Person.call(this, name, gender);
	this.school = school;
}
function Employee(name, gender, company) {
	Person.apply(this, [name, gender]);
	this.company = company;
}
var by = new Student('하나', 'female', '교대');
var jn = new Employee('보영', 'female', '구글');

console.log(by);  // Student {name: "하나", gender: "female", school: "교대"}
console.log(jn);  // Employee {name: "보영", gender: "female", company: "구글"}
 

실습하기 : https://jsfiddle.net/

 

 


3) 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때

여러 개의 인수를 받는 메서드에게 하나의 배열로 인수들을 전달하고 싶을 때 apply를 사용한다.

 

예 > 배열에서 최대/최소값을 구해야 할 경우
var numbers = [10, 20, 3, 16, 45];
var max = min = numbers[0];

numbers.forEach(function(number){
	if(number > max) {
		console.log(max);    
		max = number;

  }
	if(number < min) {
		console.log(min);    
		min = number;
  }
});

console.log(max, min);  //45 3
 

실습하기 : https://jsfiddle.net/

 

 

ES5에서의 Math.max, Math.min 활용

 

Math.max / Math.min 메서드에 apply를 적용하면 코드가 많이 단축된다 

 

var numbers = [10, 20, 3, 16, 45];
var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);

console.log(max, min);  //45 3
 

Math.max() : 함수는 0이상의 숫자 중 가장 큰 숫자를 반환 
만약 아무 요소도 주어지지 않았다면 -Infinity로 반환
만약 인수 중 하나라도 숫자로 변환하지 못한다면 NaN로 반환
ex)Math.max(10, 20); // 20

Math.min() : 함수는 주어진 숫자들 중 가장 작은 값을 반환 
만약 주어진 인자값이 없을 경우, Infinity 가 반환
적어도 1개 이상의 인자값이 숫자형으로 변환이 불가능 한 경우, NaN 가 반환
 

 

ES6에서의 Math.max, Math.min 펼치기연산자 활용

Es6에서는 spread operator (펼치기연산자, 전개구문)를 이용하면 apply보다 더욱 간결함

 

var numbers = [10, 20, 3, 16, 45];
var max = Math.max(...numbers);
var min = Math.min(...numbers);

console.log(max, min);  //45 3
 

전개 구문을 사용하면 배열이나 문자열과 같이 반복 가능한 문자를 0개 이상의 인수 (함수로 호출할 경우) 또는 요소 (배열 리터럴의 경우)로 확장하여, 0개 이상의 키(key)-값(value)의 쌍으로 객체로 확장시킬 수 있다.
=> apply() 대체

인수 목록의 모든 인수는 전개 구문을 사용할 수 있으며, 여러번 사용될 수도 있습니다.
 

 

call/apply 메서드는 명시적으로 별도의 this를 바인딩하면서 함수나 메서드를 실행하는 좋은 방법이지만
이로 인해 this를 예측하기 어렵게도 만들어 코드 해석을 방해하는 단점이 있음
그러나 ES5 이하 환경에서는 대안이 없어 실무에서 광범위하게 사용된다

 

 

 

 
2-4 bind 메서드
 

ES5에서 추가된 기능
call과 비슷하나 즉시 호출하지 않고 넘겨받은 this와 인수들을 바탕으로 새로운 함수를 반환만 하는 메서드
새로운 함수를 호출할 때 인수를 넘기면 기존 bind메서드를 호출할 때 전달했던 인수들의 뒤에 이어서 등록

 

목적

함수에 this를 미리 적용한다
부분 적용 함수를 구현한다
 

var func = function (a, b, c, d){
	console.log(this, a, b, c, d);
};
func(1, 2, 3, 4);  
//Window {parent: global, opener: null, top: global, length: 0, frames: Window, …} 1 2 3 4

var bindFunc1 = func.bind({ x: 1 });
bindFunc1(5,6,7,8); //{x: 1} 5 6 7 8

var bindFunc2 = func.bind({ x: 1 }, 4, 5);
bindFunc2(6,7); //{x: 1} 4 5 6 7
bindFunc2(8,9);  //{x: 1} 4 5 8 9
 

실습하기 : https://jsfiddle.net/

 

 

 

name 프로퍼티

bind 메서드를 적용해서 새로 만든 함수는 name 프로퍼티에 bind의 수동태인 'bound'라는 접두어가 붙는다. 장점 : 코드 추적에 call이나 apply보다 수월하다.


예 ) bound xxx : 함수명이 xxx인 워본 함수에 bind 메서드를 적용한 새로운 함수

 

var func = function(a, b, c, d){
	console.log(this, a, b, c, d);
};

var bindFunc = func.bind({X:1}, 4, 5);
console.log(func.name);  //func
console.log(bindFunc.name);  //bound func
 

 

상위 컨텍스트의 this를 내부함수나 콜백 함수에 전달하기

이전의 self 변수를 활용한 우회법보다 call, apply, bind 메서드로는 더 깔끔한 처리 가능
아래는 call, bind로 내부함수에 this를 전달하는 방법

 

var obj = {
	outer: function(){
		console.log(this);  //{outer: ƒ}
		var innerFunc = function(){
			console.log(this);  //{outer: ƒ}
		};
    innerFunc.call(this);
    console.log(this); //{outer: ƒ}
    console.log(innerFunc(this)); //Window {parent: global, opener: null, top: global, length: 0, frames: Window, …}
  }
};
obj.outer();
var obj = {
	outer: function(){
		console.log(this);  //{outer: ƒ}
		var innerFunc = function(){
			console.log(this);  //{outer: ƒ}
		}.bind(this);
    innerFunc(this);
    console.log(this); //{outer: ƒ}    
  }
};
obj.outer();
 

 

콜백 함수를 인자로 받는 함수나 메서드 중에서 기본적으로 콜백 함수 내에서의 this에 관여하는 함수나 메서드에 대해서도 bind 메서드를 사용하면 this값을 사용자 편의에 따라 바꿀 수 있다

 

var obj = {
  logThis: function() {
    console.log(this); 
  },
  logThisLater1: function() {
    setTimeout(this.logThis, 500); 
  },
  logThisLater2: function() {
    setTimeout(this.logThis.bind(this), 1000); 
  },
};
obj.logThisLater1();  //Window {parent: global, opener: null, top: global, length: 0, frames: Window, …}
obj.logThisLater2();  //{logThis: ƒ, logThisLater1: ƒ, logThisLater2: ƒ}
 

실습하기 : https://jsfiddle.net/

 

 

 

2-5 화살표 함수의 예외사항
ES6에 새롭게 도입된 함수로 실행 컨텍스트 생성 시 this를 바인딩하는 과정이 제외
화살표 함수 내부에는 this가 아예 없으며 접근하고자 하면 스코프체인상 가장 가까운 this에 접근

 

var obj = {
	outer : function(){
		console.log(this);  //{outer: ƒ}
		var innerFunc = () => {
			console.log(this);  //{outer: ƒ}
		};
		innerFunc();  
	}
};
obj.outer();
 

상위 컨텍스트의 this를 내부함수나 콜백함수에 전달하기 예제를 화살표 함수로 바꾼 것
이렇게 하면 별도의 변수로 this를 우회하거나 call/bind/apply를 적용할 필요 없이 더욱 간결하고 편리하다

 

 

 

 
2-6 별도의 인자로 this를 받는 경우 (콜백 함수 내에서의 this)
 
콜백 함수는 다음 장에서 이어하기에 this와  관련된 부분만 간단히 짚는다
콜백 함수를 인자로 받는 메서드 중 일부는 추가로 this로 지정할 객체 (thisArg)를 인자로 지정할 수 있는 경우가 있다
이러한 메서드의 thisArt 값을 지정하면 콜백 함수 내부에서 this 값을 원하는 대로 변경할 수 있다

이런 형태는 여러 내부요소에 대해 같은 동작을 반복해야하는 배열 메서드에 많이 포진
ES6의 Set, Map 등 메서드에서도 일부 존재한다.


 

대표적인 배열 메서드인 forEach의 예

 

// report 객체의 프로퍼티 : sum, count  
// report 객체의 메서드 : add, average

var report = {
	sum: 0,
	count: 0,
    
	// add 메서드는 arguments를 배열로 변환해서 args 변수에 담고
	// 배열을 순회하면서 콜백 함수를 실행

	add: function(){
		var args = Array.prototype.slice.call(arguments);
        
        // 콜백 함수 내부에서의 this : forEach 함수의 두 번째 인자로 전달해준 this가 바인딩됨

		args.forEach(function (entry){
			this.sum += entry;
			++this.count;
		}, this);
	},
    
    // 메서드
    
	average : function(){
		return this.sum / this.count;
	}
};

// 괄호 안 숫자들을 인자로 삼아 add메서드를 호출하면 
// 세 인자를 배열로 만들어 forEach 메서드가 실행
// 콜백 함수 내부의 this는 이제 add 메서드에서의 this가 전달된 상태므로
// this(report)를 가리키고 있으므로 순회하면서 report.sum, report.count 값이 변경

report.add(60, 85, 95);
console.log(report.sum, report.count, report.average()); //240 3 80
 
실습하기 : https://jsfiddle.net/

 

 

콜백 함수와 함께 thisArg를 인자로 받는 메서드
forEach 외에도 thisArg를 인자로 받는 메서드는 여러개 있다

Array.prototype.forEach();
Set.prototype.forEach();
Map.prototype.forEach();

Array.prototype.map();
Array.prototype.filter();
Array.prototype.some();
Array.prototype.every();
Array.prototype.find();
Array.prototype.findIndex();
Array.prototype.flatMap();
Array.prototype.from();
 

 

 

### 요    약
 
- 명시적이지 않은 this
전역 공간에서의 this는 전역객체를 참조
어떤 함수의 메서드로서 호출한 경우 this는 메서드 호출 주체(메서드명 앞의 객체)를 참조
어떤 함수를 함수로서 호출한 경우 this는 전역 객체를 참조 (메서드 내부함수에서도 같음)
콜백 함수 내부에서 this는 해당 콜백 함수의 제어권을 넘겨받은 함수가 정의한 바에 따르며 정의하지 않은 경우 전역객체를 참조
생성자 함수에서 this는 생성될 인스턴스를 참조
 
- 명시적 this 바인딩의 규칙
call, apply 메서드는 this를 명시적으로 지정하면서 함수 또는 메서드를 호출
bind 메서드는 this와 함수에 넘길 인수를 일부 지정해서 새로운 함수를 생성
요소를 순회하면서 콜백 함수를 반복 호출하는 내용의 일부 메서드는 별도의 인자로 this를 받기도 함
