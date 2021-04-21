> **학습목표**
상황별로 this가 어떻게 달라지는지, 왜 그렇게 되는지, 예상과 다른 대상을 바라보고 있을 경우 그 원인을 효과적으로 추적하는 방법에 대해 할 수 있다.

대부분의 객체지향 언어에서 this는 클래스로 생성한 인스턴스 객체를 의미.
클래스에서만 사용할 수 있기 때문에 혼란의 여지가 없거나 많지 않다.

**자바스크립트에서의 this는 어디서든 사용할 수 있고, 상황에 따라 this가 바라보는 대상이 달라진다.**
함수와 객체(메서드)의 구분이 느슨한 자바스크립트에서 this는 실질적으로 이 둘을 구분하는 거의 유일한 기능.

# 상황에 따라 달라지는 this
>
**this는 기본적으로 실행 컨텍스트가 생성될 때 함께 결정된다.**
= this는 함수를 호출할 때 결정된다.
= this는 함수를 어떤 방식으로 호출하느냐에 따라 값이 달라진다.

## 전역 공간에서의 this
> **전역 공간에서 this는 전역 객체를 가리킨다.**
브라우저 환경에서 전역객체는 **window**
Node.js에서 전역객체는 **global**

### 전역변수와 전역객체
**전역변수를 선언하면 자바스크립트 엔진은 이를 전역객체의 프로퍼티로도 할당한다.**
👉 자바스크립트의 모든 변수는 실은 특정 객체(실행 컨텍스트의 L.E)의 프로퍼티로서 동작하기 때문.

실행 컨텍스트는 변수를 수집해서 LexicalEnvironment(L.E)의 프로퍼티로 저장.
이후 어떤 변수를 호출하면 L.E를 조회해서 일치하는 프로퍼티가 있을 경우 그 값을 반환.
**전역 컨텍스트의 L.E** -> (globalEnv) -> **전역객체**를 참조.
```jsx
var a = 1;
console.log(a);  // 1
console.log(window.a);  // 1
console.log(this.a);  // 1
```

#### Q. 전역 공간에서는 var로 변수를 선언하는 대신 window의 프로퍼티에 직접 할당해도 똑같이 동작할까?

A. 
대부분의 경우에서는 그렇다.
```jsx
var a = 1;
window.b = 2;
console.log(a, window.a, this.a);  // 1, 1, 1 
console.log(b, window.b, this.b);  // 2, 2, 2

window.a = 3;
b = 4;
console.log(a, window.a, this.a);  // 3, 3, 3 
console.log(b, window.b, this.b);  // 4, 4, 4

```
 
단, '삭제' 명령에 대해서는 다르게 동작한다.

```jsx
var a = 1;
delete window.a;  // false
console.log(a, window.a, this.a);  // 1, 1, 1

var b = 2;
delete b;  // false
console.log(b, window.b, this.b);  // 2, 2, 2

window.c = 3;
delete window.c;  // true
console.log(c, window.c, this.c);  // Uncaught ReferenceError: c is not defined

window.d = 4;
delete d;  // true
console.log(d, window.d, this.d);  // Uncaught ReferenceError: c is not defined
```
- 처음부터 전역객체의 프로퍼티로 할당한 경우: **삭제 가능**
- 전역변수로 선언한 경우: **삭제 불가능**
✔️ 전역변수로 선언하면 자바스크립트 엔진이 이를 전역객체의 프로퍼티로 할당하면서 추가적으로 해당 프로퍼티의 configurable 속성(변경 및 삭제 가능성)을 false로 정의하기 때문.


## 메서드로서 호출할 때 그 메서드 내부에서의 this
### 함수 vs. 메서드
>
프로그래밍 언어에서 함수와 메서드는 미리 정의한 동작을 수행하는 코드 뭉치로, 이 둘을 구분하는 유일한 차이는 **독립성**에 있다.
- 함수 : 그 자체로 독립적인 기능을 수행
- 메서드: 자신을 호출한 대상 객체에 관한 동작을 수행

```jsx
var func = function (x) {
	console.log(this, x);
};
func(1);  // Window { ... }, 1

var obj = {
	method: func
};
obj.method(2);  // { method: f } (obj), 2
```

#### 함수로서 호출과 메서드로서 호출을 구분하는 방법
- 점이 없으면 함수로서 호출
- 점이 있으면 메서드로서 호출
✔️ 점 표기법이든 대괄호 표기법이든, 어떤 함수를 호출할 때 그 함수 이름(프로퍼티명) 앞에 객체가 명시돼 있는 경우에는 메서드로 호출한 것이고, 그렇지 않은 모든 경우에는 함수로 호출한 것.

```jsx
var obj = {
 method: function (x) { console.log(this, x); }
};

obj.method(1);  // { method: f }, 1
obj['method'](2);  // { method: f }, 2
```

### 메서드 내부에서의 this
> **this에는 호출한 주체에 대한 정보가 담긴다.**
어떤 함수를 메서드로서 호출하는 경우 호출 주체는 바로 함수명(프로퍼티명) 앞의 객체 = 마지막 점 앞에 명시된 객체가 곧 this.

```jsx
var obj = {
	methodA: function () { console.log(this); },
  	inner: {
    	methodB: function () { console.log(this); }
    }
};

obj.methodA();  // { methodA: f, inner: { ... } } === obj
obj['methodA']();  // obj

obj.inner.methodB();  // {methodB: f} === obj.inner
obj.inner['methodB']();  // obj.inner
obj['inner'].methodB();  // obj.inner
obj['inner']['methodB']();  // obj.inner
```

## 함수로서 호출할 때 그 함수 내부에서의 this
### 함수 내부에서의 this
어떤 함수를 함수로서 호출할 경우에는 (호출 주체를 명시하지 않기 때문에, 알수 없으므로) this가 지정되지 않는다.
this가 지정되지 않는 경우 this는 전역객체를 바라본다.
👉 설계상의 오류

### 메서드의 내부함수에서의 this
```jsx
var obj1 = {
	outer: function () {
    	console.log(this);  // (1)
      	var innerFunc = function () {
        	console.log(this);  // (2)
        }
    	innerFunc();
      	
    	var obj2 = {
    		innerMethod: innerFunc
    	};
    	obj2.innerMethod();
	}
};
obj1.outer();
```
✔️ (1) obj1 (2) window, obj2
> this 바인딩에 관해서는 함수를 실행하는 당시의 주변 환경 (메서드 내부인지, 함수 내부인지 등)은 중요하지 않다.
**오직 해당 함수를 호출하는 구문 앞에 점 또는 대괄호 표기의 유무가 관건이다.**

### 메서드의 내부 함수에서 this를 우회하는 방법
>
**호출 주체가 없을 때는 자동으로 전역객체를 바인딩 하지 않고, 호출 당시 주변 환경의 this를 그대로 상속받아 사용할 수 있으면 좋겠다!**
스코프 체인처럼 this 역시 현재 컨텍스트에 바인딩된 대상이 없으면 직전 컨텍스트의 this를 바라보는 것이 더 자연스럽기 때문.

#### 내부함수에서 this를 우회하는 방법
```jsx
var obj = {
	outer: function () {
    	console.log(this);  // (1) obj
      	var innerFunc1 = funtion () {
        	console.log(this);  // (2) window
        }
      	innerFunc1();
      	
      	var self = this;  // 상위 스코프의 this를 저장해서 내부함수에서 활용하려는 수단
      	var innerFunc2 = function () {
        	console.log(self);  // (3) obj
        };
      	innerFunc2();
    }
};
obj.outer();

```
### this를 바인딩하지 않는 함수
ES6에서는 함수 내부에서 this가 전역객체를 바라보는 문제를 보완하고자, this를 바인딩하지 않는 화살표 함수(arrow function)를 도입.
화살표 함수는 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠지게 되어, 상위 스코프의 this를 그대로 활용할 수 있다.

```jsx
var obj = {
	outer: function () {
    	console.log(this);  // (1) obj
      	var innerFunc = () => {
        	console.log(this);  // (2) obj
        }
      	innerFunc();
    }
};
obj.outer();
```
그 밖에도 call, apply 등의 메서드를 활용해 함수를 호출할 때 명시적으로 this를 지정하는 방법이 있다.


## 콜백 함수 호출 시 그 함수 내부에서의 this
```jsx
setTimeout(function () { console.log('(1)',this); }, 300);  // (1)

[1, 2, 3, 4, 5].forEach(function (x) {  // (2)
	console.log('(2)',this, x);
});

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector('#a')
	.addEventListener('click', function (e) {  // (3)
		console.log('(3)',this, e);
	});
```
(1) setTimeout 함수와 (2) forEach 메서드는 그 내부에서 콜백 함수를 호출할 때 대상이 될 this를 지정하지 않는다. 👉 this는 전역 객체를 참조
(3) addEventListener 메서드는 콜백 함수를 호출할 때 자신의 this를 상속하도록 정의한다. 👉 메서드명의 점 앞부분이 this

## 생성자 함수 내부에서의 this
> **생성자 함수: 어떤 공통된 성질을 지니는 객체들을 생성하는데 사용하는 함수**
객체지향 언어에서는 생성자를 클래스(class), 클래스를 통해 만든 객체를 인스턴스(instance)라고 한다.

자바스크립트는 함수에 생성자로서의 역할을 함께 부여함.

new 명령어와 함께 함수를 호출하면 해당 함수가 생성자로서 동작하게 된다.
어떤 함수가 생성자 함수로서 호출된 경우 **내부에서의 this는 곧 새로 만들 구체적인 인스턴스 자신**이 된다.

1. 생성자 함수를 new 명령어와 함께 호출
2. 생성자의 prototype 프로퍼티를 참조하는 	```__proto__``` 라는 프로퍼티가 있는 객체(인스턴스) 생성 
3. 미리 준비된 공통 속성 및 개성을 해당 객체(this) 에 부여

```jsx
var Cat = function (name, age) {
	this.bark = '야옹';
  	this.name = name;
  	this.age = age;
};
var choco = new Cat('초코', 7); 
var nabi = new Cat('나비', 5);
console.log(choco, nabi);

/* 결과
Cat {bark: "야옹", name: "초코", age: 7} 
Cat {bark: "야옹", name: "나비", age: 5}
*/
```

# 명시적으로 this를 바인딩 하는 방법
앞 절의 규칙에 부합하지 않는다면 아래 방법들을 사용하여 this에 별도의 대상을 바인딩 하는 방법을 사용했을 것으로 추측.

## call 메서드
```jsx
Function.prototype.call(thisArg[,arg1,[,arg2[,...]]])
```
> call 메서드는 **메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령**
이때 call메서드의 첫 번째 인자를 this로 바인딩 하고, 이후의 인자들을 호출할 함수의 매개변수로 한다.

👉 함수를 실행할 때 임의의 객체를 this로 지정할 수 있다.
```jsx
var func = function (a,b,c) {
	console.log(this, a, b, c);
};

func(1,2,3);  // Window{...} 1 2 3
func.call({x: 1}, 4,5,6);  // {x: 1} 4 5 6
```

👉 메서드에 대해서도 임의의 객체를 this로 지정할 수 있다.
```jsx
var obj = {
	a: 1,
  	method: function (x, y) {
    	console.log(this.a, x, y);
    } 
};

obj.method(2, 3);  // 1 2 3
obj.method.call({a: 4}, 5, 6);  // 4 5 6
```

## apply 메서드

```jsx
Function.prototype.apply(thisArg[, argsArray])
```
> **apply 메서드는 call 메서드와 기능적으로 완전히 동일.**
call 메서드는 첫 번째 인자를 제외한 나머지 모든 인자들을 호출할 함수의 매개변수로 지정하는 반면,  apply 메서드는 **두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정한다**는 점에서만 차이가 있다.

```jsx
var func = fuction (a,b,c) {
	console.log(this, a, b, c);
};
func.apply({x: 1}, [4, 5, 6]);  // {x: 1} 4 5 6

var obj = {
	a: 1,
  	method: function (x, y) {
    	console.log(this.a, x, y);
    }
};
obj.method.apply({a: 4}, [5, 6]); // 4 5 6
```

## call/ apply 메서드의 활용
### 유사배열객체에 배열 메서드를 적용
객체에는 배열 메서드를 직접 적용할 수 없다.
key가 0이나 양의 정수인 프로퍼티가 존재하고 length 프로퍼티의 값이0 또는 양의 정수인 객체 (유사배열객체)의 경우 call, apply메서드를 이용해 배열 메서드를 차용할 수 있다.

```jsx
var obj = {
	0:'a',
	1:'b',
	2:'c',
	length : 3
};
Array.prototype.push.call(obj, 'd');
console.log(obj); // { 0:'a', 1:'b', 2:'c', 3:'d', length : 4}

var arr = Array.prototype.slice.call(obj);
console.log(arr); // ['a', 'b', 'c', 'd']
```

함수 내부에서 접근할 수 있는 arguments 객체도 유사배열객체이므로 위 방법으로 배열로 전환해서 활용할 수 있다. querySelectorAll, getElementsByClassName 등의 Node 선택자의 선택 결과인 NodeList도 마찬가지.

```jsx
function a() {
    var argv = Array.prototype.slice.call(arguments);
    console.log(arguments);  
    console.log(argv);

    argv.forEach(function (arg) {
       console.log(arg);
}
a(1,2,3);

document.body.innerHTML = '<div>a</div><div>b</div><div>c</div>';

var nodeList = document.querySelectorAll('div'); 
console.log(nodeList);
var nodeArr = Array.prototype.slice.call(nodeList);
console.log(nodeArr);

nodeArr.forEach(function (node) {
	console.log(node);
});
```


배열처럼 인덱스와 length 프로퍼티를 지니는 문자열도 call/apply 메서드를 이용해서 배열 메서드를 적용 가능.
단, 문자열의 경우 length 프로퍼티는 읽기전용이기 때문에 원본 문자열에 변경을 가하는 메서드인 push, pop, shift, unshift, splice 등은 에러를 던짐.
concat 처럼 대상이 반드시 배열이어야만 하는 경우에도 에러는 안나지 않지만 제대로된 결과를 얻을 수 없음.

 ```jsx

var str = 'abc def';

Array.prototype.push.call(str, ', pushed string'); 
// Error : Cannot assign to read only pro

Array.prototype.concat.call(str, 'string'); //[String {"abc def"}, "string"]

Array.prototype.every.call(str, function(c) {return char !== ''; }); //false

Array.prototype.some.call(str, function(c) {return char !== ''; }); //true


var newArr = Array.prototype.map.call(str, function(c) {return char + '!'; });
console.log(newArr); // ['a!', 'b!', 'c!', '!', 'd!', 'e!', 'f!' ]

var newStr = Array.prototype.reduce.apply(str, [function(string, char, i) 
	{return string + char + i;}, '' ]);
console.log(newArr);  // "a0b1c2 3d4e5f6"
 ```

사실 call/apply 방법으로 형변환하는 것은 this를 원하는 값으로 지정해서 호출한다는 본래의 메서드 의도와는 다소 동떨어진 활용법.
slice 메서드는 오직 배열 형태로 복사하기 위해서 사용되었지만, 코드만 봐서는 어떤 의도인지 파악하기 쉽지 않다.
 
ES6에서는 유사배열객체 또는 순회 가능한 모든 종류의 데이터타입을 배열로 전환하는Array.from 메서드를 새로 도입했다.

Array.from() 메서드는 유사 배열 객체(array-like object)나 반복 가능한 객체(iterable object)를 얕게 복사해 새로운 Array 객체를 만든다.

 ```jsx

var obj = {
	0: 'a',
	1: 'b',
	2: 'c',
	length: 3
};
var arr = Array.from(obj);
console.log(arr);  // ['a', 'b', 'c']
 ```


### 생성자 내부에서 다른 생성자를 호출

생성자 내부에 다른 생성자와 공통된 내용이 있을 시 call/apply를 이용해 다른 생성자를 호출하면 반복을 줄일 수 있다. 

 ```jsx

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
var by = new Student('보영', 'female', '단국대');
var jn = new Employee('재난', 'male', '구글');

console.log(by);  // Student {name: "보영", gender: "female", school: "단국대"}
console.log(jn);  // Employee {name: "재난", gender: "male", company: "구글"}
 ```

### 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용

 

#### 최대/최소값을 구하는 코드를 직접 구현
```jsx
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
 ```

👉 Math.max / Math.min 메서드에 apply를 적용하면 훨씬 간단

 
#### 여러 인수를 받는 메서드(Math.max / Math.min)에 apply 적용
```jsx
var numbers = [10, 20, 3, 16, 45];
var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);

console.log(max, min);  //45 3
 ```
 

#### ES6에서의 펼치기연산자 활용
```jsx

var numbers = [10, 20, 3, 16, 45];
var max = Math.max(...numbers);
var min = Math.min(...numbers);

console.log(max, min);  //45 3
 
```

call/apply 메서드는 명시적으로 별도의 this를 바인딩하면서 함수나 메서드를 실행하는 좋은 방법이지만
이로 인해 this를 예측하기 어렵게도 만들어 코드 해석을 방해하는 단점이 있다.
그러나 ES5 이하 환경에서는 대안이 없어 실무에서 광범위하게 사용.


## bind 메서드
```jsx
Function.prototype.bind(thisArg[,arg1,[, arg2[,...]]])
```
> ES5에서 추가된 기능으로 call 과 비슷하지만 즉시 호출하지는 않고 넘겨 받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드.

👉 this 지정과 부분 적용 함수 구현

```jsx
var func = function (a, b, c, d) {
	console.log(this, a, b, c, d);
};
func(1, 2, 3, 4);  // Window {...} 1 2 3 4

var bindFunc1 = func.bind({x: 1});
bindFunc1(5, 6, 7, 8);  // {x: 1} 1 2 3 4

var bindFunc2 = func.bind({x: 1}, 4, 5);
bindFunc2(6, 7);  // {x: 1} 4 5 6 7
bindFunc2(8, 9);  // {x: 1} 4 5 8 9
```
### name 프로퍼티
> 
bind 메서드를 적용해서 새로 만든 함수는 name 프로퍼티에 ```bound``` 라는 접두어가 붙는다.
👉 원본 함수에 bind 메서드를 적용한 새로운 함수라는 의미로, 기존의 call, apply 보다 코드를 추적하기에 더 수월함

```jsx
var func = function (a, b, c, d) {
	console.log(this, a, b, c, d);
};
var bindFunc = func.bind({x: 1}, 4, 5);
console.log(func.name);  // func
console.log(bindFunc.name);  // bound func
```

### 상위 컨텍스트의 this를 내부함수나 콜백함수에 전달하기

```jsx
// call
var obj = {
	outer: function() {
    	console.log(this);
      	var innerFunc = function () {
        	console.log(this);
        };
      	innerFunc.call(this);
    }
};
obj.outer();

// bind
var obj = {
	outer: function() {
    	console.log(this);
      	var innerFunc = function () {
        	console.log(this);
        }.bind(this);
      	innerFunc.();
    }
};
obj.outer();
```

## 화살표 함수의 예외사항
> ES6에 새롭게 도입된 화살표 함수는 실행 컨텍스트 생성 시 this를 바인딩하는 과정이 제외됨.
이 함수 내부에는 this가 아예 없으며, 접근하고자 하면 스코프체인상 가장 가까운 this에 접근하게 된다.

```jsx
var obj = {
	outer: function() {
    	console.log(this);
      	var innerFunc = () => {
        	console.log(this)
        };
  		innerFunc();
    };
};
obj.outer();
```

## 별도의 인자로 this를 받는 경우 (콜백 함수 내에서의 this)
### 콜백 함수와 함께 thisArg를 인자로 받는 메서드
```jsx
Array.prototype.forEach(callback[, thisArg])
Array.prototype.map(callback[, thisArg])
Array.prototype.filter(callback[, thisArg])
Array.prototype.some(callback[, thisArg])
Array.prototype.every(callback[, thisArg])
Array.prototype.find(callback[, thisArg])
Array.prototype.findIndex(callback[, thisArg])
Array.prototype.flatMap(callback[, thisArg])
Array.prototype.from(arrayLike[, callback[, thisArg]])
Set.prototype.forEach(callback[, thisArg])
Map.prototype.forEach(callback[, thisArg])
```

# 정리
 
### ✔️ **명시적 this 바인딩이 없는 한 늘 성립**
>
- 전역공간에서의 this는 전역객체(Window, global)를 참조한다.
- 어떤 함수를 메서드로서 호출한 경우 this는 메서드 호출 주체(메서드명 앞의 객체)를 참조한다.
- 어떤 함수를 함수로서 호출한 경우 this는 전역객체를 참조한다. 메서드 내부함수에서도 동일하다.
- 콜백 함수 내부에서의 this는 해당 콜백 함수의 제어권을 넘겨받은 함수가 정의한 바에 따르며, 정의하지 않은 경우에는 전역객체를 참조한다.
- 생성자 함수에서의 this는 생성될 인스턴스를 참조한다.

### ✔️ 명시적 this 바인딩
>
- call, apply 메서드는 this를 명시적으로 지정하면서 함수 또는 메서드를 호출한다.
- bind 메서드는 this 및 함수에 넘길 인수를 일부 지정해서 새로운 함수를 만든다.
- 요소를 순회하면서 콜백 함수를 반복 호출하는 내용의 일부 메서드는 별도의 인자로 this를 받기도 한다.

