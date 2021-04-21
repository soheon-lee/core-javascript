- 다른 대부분의 객체지향 언어에서 this는 class로 생성한 instance 객체를 의미

    class에서만 사용할 수 있기 때문에 혼란의 여지가 없거나 많지 않다.

- 그러나, JS에서의 `this`는 어디서든 사용할 수 있다.

    함수와 객체(메서드)의 구분이 느슨한 JS에서 this는 실질적으로 이 둘을 구분하는 거의 유일한 기능

# 01. 상황에 따라 달라지는 this

- JS에서 `this`는 기본적으로 execution context(실행 컨텍스트)가 생성될 때 즉, **함수를 호출할 때 결정된다.**

## 1. 전역 공간에서의 this

- 전역 공간에서 this는 전역 객체를 가리킴 (전역 context를 생성하는 주체가 바로 전역 객체이기 때문)
- 전역 객체는 JS 런타임 환경에 따라 다른 이름과 정보를 가지고 있다. (ex. 브라우저 환경에서 전역객체는 `window`, Node.js 환경에서는 `global`)

    ```jsx
    // 브라우저 환경
    console.log(this === window);  //true

    // Node.js 환경
    console.log(this === global);  //true
    ```

- **전역변수를 선언하면 JS 엔진은 이를 전역객체의 property로 할당한다.**

    JS의 모든 변수는 실은 특정 객체의 property로서 동작한다.

    즉, 사용자가 var 연산자를 이용해 변수를 선언하더라도 실제 JS 엔진은 어떤 특정 객체(실행 컨텍스트의 LexicalEnvironment)의 property로 인식

    ```jsx
    // 브라우저 환경
    var a = 1;

    console.log(a);        //1
    console.log(window.a)  //1
    console.log(this.a)    //1
    ```

- 대부분의 경우, 전역 공간에서는 var 변수 대신 window의 property에 직접 할당하더라도 var 선언과 똑같이 동작

    ```jsx
    var a = 1;
    window.b = 2;

    console.log(a, window.a, this.a);  //1 1 1
    console.log(b, window.b, this.b);  //2 2 2
    ```

- `delete` 명령에 대해서는, 전역변수 선언과 전역객체의 property 할당이 전혀 다른 경우도 있다.

    처음부터 전역객체의 property로 할당한 경우는 삭제 가능, but 전역변수로 선언한 경우는 삭제 불가

    즉, 전역변수를 선언하면 JS 엔진이 이를 자동으로 전역객체의 property로 할당하면서, 추가적으로 해당 property의 configurable 속성(변경 및 삭제 가능성)을 false로 정의

    (사용자가 의도치 않게 삭제하는 것을 방지하는 나름의 방어 전략이라고 해석됨)

    ⇒ var로 선언한 전역변수와 전역객체의 property는 hoisting 여부 및 configurable 여부에서 차이를 보임

    ```jsx
    var a = 1;
    delete a;         //false
    console.log(a)    //1
    ```

    ```jsx
    var a = 1;
    delete window.a;  //false
    console.log(a)    //1
    ```

    ```jsx
    window.a = 1;
    delete a;         //true
    console.log(a)    //Uncaught ReferenceError: c is not defined
    ```

    ```jsx
    window.a = 1;
    delete window.a;  //true
    console.log(a);   //Uncaught ReferenceError: c is not defined
    ```

## 2. Method로서 호출할 때 그 method 내부에서의 this

### Function vs Method

- 둘을 구분하는 유일한 차이는 **독립성**
- Function: 그 자체로 독립적인 기능을 수행
- Method: 자신을 호출한 대상 객체에 관한 동작을 수행

### 함수로서 호출, 메서드로서 호출

- 어떤 함수를 객체의 property에 할당한다고 해서 그 자체로 무조건 method가 되는 것이 아니라, 객체의 method로서 호출할 경우에만 method로 동작하고 그렇지 않으면 함수로 동작

    ```jsx
    var func = function(){
        console.log(this);
    };
    func();          // Window{...}   -> function으로 동작

    var obj = {
        method: func
    };
    obj.method();    // {method: f}   -> method로 동작
    ```

- 점 표기법이든 대괄호 표기법이든, 어떤 함수를 호출할 때 그 함수 이름(property명) 앞에 객체가 명시되어 있는 경우에는 method로 호출한 것이고 그렇지 않으면 함수로 호출한 것

    ```jsx
    var obj = {
        method: function(){ console.log(this); }
    };

    obj.method();     // {method: f}
    obj['method']();  // {method: f}
    ```

- **this에는 호출한 주체에 대한 정보가 담김**

    어떤 함수를 method로서 호출하는 경우 호출 주체는 바로 함수명(property명) 앞의 객체이다.

    즉, 점 표기법의 경우 마지막 점 앞에 명시된 객체가 곧 this가 되는 것

    ```jsx
    var obj = {
        method1: function(){ console.log(this); },
        inner: {
            method2: function(){ console.log(this); }
        }
    };

    obj.method1();        // {method1: f, inner: {...}}  (===obj)
    obj.inner.method2();  // {method2: f}                (===obj.inner)
    ```

## 3. Function으로서 호출할 때 그 function 내부에서의 this

- 어떤 함수를 함수로서 호출할 경우에는 this가 지정되지 않고, this가 지정되지 않은 경우 this는 전역 객체를 바라봄

    this는 호출한 주체에 대한 정보가 담기는데, 함수로서 호출하는 것은 호출 주체(객체지향언어에서의 객체)를 명시하지 않고 개발자가 코드에 직접 관여해서 실행한 것이기 때문에 호출 주체의 정보를 알 수 없기 때문에 지정되지 않음

- this 바인딩에 관해서는 함수를 실행하는 당시의 주변 환경(메서드 내부인지 함수 내부인지 등)은 중요하지 않고, 오직 해당 함수를 호출하는 구문 앞에 점 또는 대괄호 표기가 있는지 없는지가 관건

    ⇒ Scope chain처럼 호출 당시 주변 환경의 this를 그대로 상속받아 사용할 수 있다면 훨씬 자연스러웠을 것

    (변수를 검색하면 가장 가까운 scope의 L.E를 찾고 없으면 상위 scope를 탐색하듯이, this 역시 현재 context에 바인딩된 대상이 없으면 직전 context의 this를 바라보게 하는 식으로..)

    ```jsx
    var obj1 = {
        outer: function(){
            console.log(this);           //this는 obj1

            var innerFunc = function(){
                console.log(this);
            }
            innerFunc();                 //this는 전역객체(Window)

            var obj2 = {
                innerMethod: innerFunc
            };
            obj2.innerMethod();          //this는 obj2
        }
    };

    obj1.outer();
    ```

### 내부함수에서의 this를 우회하는 방법

- **ES5:** 변수를 활용(ex. `var self = this;`)

    상위 스코프의 this를 저장해서 내부함수에서 활용하는 방법

    사람마다 쓰는 변수명은 _this, that, _ 등 다양한데, self가 가장 많이 쓰임

    ```jsx
    var obj = {
        outer: function(){
            console.log(this);            //this는 obj

            var innerFunc1 = function(){
                console.log(this);
            }
            innerFunc1();                 //this는 전역객체(Window)

            var self = this;
            var innerFunc2 = function(){
                console.log(self);
            };
            innerFunc2();                 //this는 obj
        }
    };

    obj.outer();
    ```

- **ES6:** 함수 내부에서 this가 전역객체를 바라보는 문제를 보완하고자, this를 바인딩하지 않는 **arrow function**을 새로 도입 (ES5 환경에서는 사용 불가)

    Arrow function은 execution context를 생성할 때 this 바인딩 과정 자체가 빠지게 되어, 상위 scope의 this를 그대로 활용 가능

    ```jsx
    var obj = {
        outer: function(){
            console.log(this);       //this는 obj

            var innerFunc = () => {
                console.log(this);
            };
            innerFunc();             //this는 obj
        }
    };

    obj.outer();
    ```

- 그 밖에도 call, apply 등의 method를 활용해 함수를 호출할 때 명시적으로 this를 지정하는 방법이 있음 ('02. 명시적으로 this를 바인딩하는 방법'에서 설명)

## 4. 콜백 함수 호출 시 그 함수 내부에서의 this

- 함수 A의 제어권을 다른 함수(또는 메서드) B에게 넘겨주는 경우, 함수 A는 콜백 함수

    함수 A는 함수 B의 내부 로직에 따라 실행되며, this 역시 함수 B 내부 로직에서 정한 규칙에 따라 값이 결정

- 콜백 함수도 함수이기 때문에 기본적으로 this가 전역객체를 참조하지만, 제어권을 받은 함수에서 콜백 함수에 별도로 this가 될 대상을 지정하는 경우에는 그 대상을 참조

- `setTimeout` function과 `forEach` method는 그 내부에서 콜백 함수를 호출할 때 대상이 될 this를 지정하지 않음 ⇒ this는 전역객체를 참조

    그러나 `addEventListener` method는 콜백 함수를 호출할 때 자신의 this를 상속하도록 정의되어 있음 ⇒ method명의 점(.) 앞부분이 곧 this

    ```jsx
    setTimeout(function(){ console.log(this); }, 300);     //this는 전역객체  ??

    [1, 2, 3].forEach(function(x){
        console.log(this, x);                              //this는 전역객체
    });

    document.body.innerHTML += '<button id="a">클릭</buttion>';
    document.body.querySelector('#a')
        .addEventListener('click', function(x){
            console.log(this, e);                          //this는 querySelector('#a')  ??
        });
    ```

## 5. 생성자 함수 내부에서의 this

- 생성자 함수: 어떤 공통된 성질을 지니는 객체들을 생성하는 데 사용하는 함수

    객체지향 언어에서는 생성자를 **class**, class를 통해 만든 객체를 **instance**라고 함 (7장 참조)

    즉, 생성자는 **구체적인 instance를 만들기 위한 일종의 틀**

- JS는 함수에 생성자로서의 역할을 함께 부여 ⇒ `new` 명령어와 함께 함수를 호출하면 해당 함수가 생성자로서 동작

    이렇게 어떤 함수가 생성자 함수로서 호출(new 명령어와 함께 함수 호출)된 경우 내부에서의 this는 곧 새로 만들 구체적인 instance 자신이 됨

    ```jsx
    var Cat = function(name, age){
        this.bark = 'Meow';
        this.name = name;
        this.age = age;
    };

    var choco = new Cat('초코', 7);    //this는 choco instance

    choco
    >>> Cat { bark: 'Meow', name: '초코', age: 7 }
    ```

# 02. 명시적으로 this를 바인딩하는 방법

## 1. call method

```jsx
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

- Method의 호출 주체인 함수를 즉시 실행하도록 하는 명령

    이 때, call method의 첫 번째 argument를 this로 바인딩하고, 이후의 인자들을 호출할 함수의 매개변수로 사용

    함수를 그냥 실행하면 this는 전역객체를 참조하지만, call method를 사용하면 임의의 객체를 this로 지정 가능

    ```jsx
    var func = function(a, b){
        console.log(this, a, b);
    };

    func(1, 2);
    >>> Window{...} 1 2

    func.call({x:1}, 1, 2);
    >>> {x:1} 1 2
    ```

    ```jsx
    var obj = {
        a: 1,
        method: function(x){
            console.log(this.a, x);
        }
    };

    obj.method(2);
    >>> 1 2

    obj.method.call({a:3}, 2);
    >>> 3 2
    ```

## 2. apply method

```jsx
Function.prototype.apply(thisArg[, argsArray])
```

- call method와 기능적으로 완전히 동일

    차이점은 apply method의 경우 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정

    ```jsx
    var func = function(a, b){
        console.log(this, a, b);
    };

    func.apply({x:1}, [2, 3])
    >>> {x:1} 2 3
    ```

## 3. call / apply method의 활용

- call이나 apply method를 잘 활용하면 JS를 더욱 다채롭게 사용 가능
- 명시적으로 별도의 this를 바인딩하면서 함수 또는 메서드를 실행하는 방법이지만, 오히려 이로 인해 this를 예측하기 어렵게 만들어 코드 해석을 방해한다는 단점도 존재

### 1) Array-like object에 array method 적용

- Key가 0 또는 양의 정수인 property가 존재하고, length property의 값이 0 또는 양의 정수인 객체, 즉 **유사배열객체**의 경우, call 또는 apply method를 이용해 배열 method를 차용할 수 있음
- `push`: array method인 push를 객체 obj에 적용해 property 2에 'c'를 추가

    `slice`: 원래 시작 index와 마지막 index를 받아 array 요소를 추출하는 method인데, 매개변수를 아무것도 넘기지 않을 경우에는 그냥 원본 배열의 얕은 복사본(배열)을 반환

    ```jsx
    var obj = {
        0: 'a',
        1: 'b',
        length: 2
    };

    Array.prototype.push.call(obj, 'c');      //왜 prototype으로만 가능??

    obj
    >>> {'0': 'a', '1': 'b', '2': 'c', length: 3}     //숫자??

    Array.prototype.slice.call(obj);
    >>> ['a', 'b', 'c']
    ```

- 함수 내부에서 접근할 수 있는 `arguments` 객체도 유사배열객체이므로 배열로 전환해서 활용 가능

    ```jsx
    function a(){
        var args = Array.prototype.slice.call(arguments);
        args.forEach(function(arg){
            console.log(arg);
        });
    }

    a(1, 2, 3);
    >>>
    1
    2
    3
    ```

- `querySelectorAll`, `getElementsByClassName` 등 Node 선택자로 선택한 결과인 `NodeList`도 가능

    ```jsx
    document.body.innerHTML = '<div>a</div><div>b</div>';

    var nodeList = document.querySelectorAll('div');
    var nodeArr = Array.prototype.slice.call(nodeList);

    nodeArr.forEach(function(node){
        console.log(node);
    });
    ```

- 배열처럼 index와 length property를 가지는 문자열에 대해서도 가능

    단, 문자열의 경우 length property가 읽기 전용이기 때문에, 원본 문자열에 변경을 가하는 method(push, pop, shift, unshift, splice 등)는 에러 발생

    concat처럼 대상이 반드시 배열이어야 하는 경우에는 에러는 나지 않지만 제대로 된 결과를 얻을 수 없음

    ```jsx
    var str = 'ab c';

    Array.prototype.push.call(str, 'pushed');
    >>> // TypeError: Cannot assign to read only property 'length' of object '[object String]'

    Array.prototype.concat.call(str, 'string');
    >>> [[String: 'ab c'], 'string']

    Array.prototype.every.call(str, function(char){return char !== ' ';});
    >>> false

    Array.prototype.some.call(str, function(char){return char !== ' ';});
    >>> true

    Array.prototype.map.call(str, function(char){return char + '!';});
    >>> ['a!', 'b!', ' !', 'c!']

    Array.prototype.reduce.apply(str, [function(string, char, i){return string + char + i;}, '']);    //이게뭐람...???
    >>> 'a0b1 2c3'
    ```

- 사실 call/apply를 이용해 형변환하는 것은 'this를 원하는 값으로 지정해서 호출한다'라는 본래의 method의 의도와는 다소 동떨어진 활용법이라 할 수 있음

    이에 **ES6**에서는 유사배열객체 또는 순회 가능한 모든 종류의 데이터 타입을 배열로 전환하는 `Array.from` method를 도입

    ```jsx
    var obj = {
        0: 'a',
        1: 'b',
        length: 2
    };

    Array.from(obj)
    >>> ['a', 'b']
    ```

### 2) 생성자 내부에서 다른 생성자를 호출

- 생성자 내부에 다른 생성자와 공통된 내용이 있을 경우 call 또는 apply를 이용해 다른 생성자를 호출하면 간단하게 반복을 줄일 수 있음

    ```jsx
    function Person(name, gender){
        this.name = name;
        this.gender = gender;
    }

    function Student(name, gender, school){
        Person.apply(this, [name, gender]);
        this.school = school;
    }

    new Student('Hong', 'female', 'univ');
    >>> Student { name: 'Hong', gender: 'female', school: 'univ' }
    ```

### 3) 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용

예를 들어 배열에서 최대/최솟값을 구해야 하는 경우

- **직접 구현**

    ```jsx
    var numbers = [3, 1, 10, 8];
    var max = min = numbers[0];

    numbers.forEach(function(num){
        if(num > max){
            max = num;
        }
        if(num < min){
            min = num;
        }
    });

    console.log(max, min);
    >>> 10 1
    ```

- **apply 활용**

    ```jsx
    var numbers = [3, 1, 10, 8];
    var max = Math.max.apply(null, numbers);   //null은 뭐지..??
    var min = Math.min.apply(null, numbers);

    console.log(max, min);
    >>> 10 1
    ```

- **spread operator 활용** (ES6의 펼치기 연산자)

    ```jsx
    var numbers = [3, 1, 10, 8];
    var max = Math.max(...numbers);
    var min = Math.min(...numbers);

    console.log(max, min);
    >>> 10 1
    ```

## 4. bind method

```jsx
Function.prototype.bind(thisArg[, arg1[, arg2[, ...]]])
```

- ES5에서 추가된 기능으로, call과 비슷하지만 즉시 호출하지는 않고 넘겨 받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 method

    새로운 함수를 호출할 때 인수를 넘기면, 그 인수들은 기본 bind method를 호출할 때 전달했던 인수들의 뒤에 이어서 등록됨

    즉, bind method는 함수에 **this를 미리 적용**하는 것과 **부분 적용 함수**를 구현하는 2가지 목적을 모두 지님

    ```jsx
    var func = function(a, b, c, d){
        console.log(this, a, b, c, d);
    };
    func(1, 2, 3, 4);
    >>> Window{...} 1 2 3 4

    var bindFunc1 = func.bind({x:1});         //this를 미리 적용
    bindFunc1(1, 2, 3, 4);
    >>> {x:1} 1 2 3 4

    var bindFunc2 = func.bind({x:1}, 1, 2);   //this 적용 + 부분적용함수 구현
    bindFunc2(3, 4);
    >>> {x:1} 1 2 3 4

    bindFunc2(5, 6);
    >>> {x:1} 1 2 5 6
    ```

- **name property**

    bind method를 적용해서 새로 만든 함수는, name property에 `bound`라는 접두어가 붙음

    어떤 함수의 name property가 'bound xxx'라면 함수명이 xxx인 원본 함수에 bind method를 적용한 함수라는 의미가 되므로, 기존의 call이나 apply보다 코드 추적이 수월

    ```jsx
    var func = function(a, b, c, d){
        console.log(this, a, b, c, d);
    };
    var bindFunc = func.bind({x:1}, 1, 2);

    func.name
    >>> func

    bindFunc.name
    >>> bound func
    ```

- **상위 컨텍스트의 this를 내부함수에 전달**

    ```jsx
    // call method

    var obj = {
        outer: function(){
            console.log(this);            //this는 obj

            var innerFunc = function(){
                console.log(this);
            };
            innerFunc.call(this);         //this는 obj
        }
    };

    obj.outer();
    ```

    ```jsx
    // bind method

    var obj = {
        outer: function(){
            console.log(this);            //this는 obj

            var innerFunc = function(){
                console.log(this);
            }.bind(this);
            innerFunc();                  //this는 obj
        }
    };

    obj.outer();
    ```

- **상위 컨텍스트의 this를 콜백 함수에 전달**

    `setTimeout`은 2번째 인자만큼의 시간 뒤에 1번째 인자(콜백 함수)를 실행하는 함수

    콜백 함수를 호출할 때 this를 따로 지정해주지 않으면 this는 전역 객체를 참조

    ```jsx
    var obj = {
        logThis: function(){
            console.log(this);
        },
        logThisLater1: function(){
            setTimeout(this.logThis, 500);
        },
        logThisLater2: function(){
            setTimeout(this.logThis.bind(this), 1000);
        }
    };

    obj.logThisLater1();
    >>> Window{...}           //전역 객체???Timeout??

    obj.logThisLater2();
    >>> obj{logThis: f, ...}  //obj
    ```

## 5. arrow function

- ES6에 새롭게 도입된 함수로, 실행 context 생성 시 this를 바인딩하는 과정이 제외

    즉 이 함수 내부에는 this가 아예 없으며, 접근하고자 하면 scope chain상 가장 가까운 this에 접근

    이렇게 하면 별도의 변수로 this를 우회하거나 call/apply/bind를 적용할 필요가 없어 더욱 간결하고 편리

    ```jsx
    var obj = {
        outer: function(){
            console.log(this);            //this는 obj

            var innerFunc = () => {
                console.log(this);
            };
            innerFunc();                  //this는 obj
        }
    };

    obj.outer();
    ```

## 6. 별도의 인자로 this를 받는 경우 (콜백 함수 내에서의 this)

- 콜백 함수를 인자로 받는 method 중 일부는 추가로 this로 지정할 객체(thisArg)를 인자로 넘기는 것이 가능

    이런 형태는 여러 내부 요소에 대해 같은 동작을 반복 수행해야 하는 **Array methods**에 많이 포진되어 있으며, 같은 이유로 ES6에 새로 등장한 Set, Map 등의 method에도 일부 존재

    ```jsx
    // forEach method

    var report = {
        sum: 0,
        count: 0,
        add: function(){
            var args = Array.prototype.slice.call(arguments);
            args.forEach(function(entry){
                this.sum += entry;        //콜백 함수 내부의 this는 forEach 함수의 2번째 인자로 전달해준 this가 바인딩
                ++this.count;
            }, this);                     //report.add로 호출했으니 this는 report
        },
    };

    report.add(1, 4, 5);

    console.log(report.sum, report.count);
    >>> 10 3
    ```

- 콜백 함수와 함께 `thisArg`를 인자로 받는 methods

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
