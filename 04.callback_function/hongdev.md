# 01. callback function

- 콜백 함수: 다른 코드의 인자로 넘겨주는 함수 + 그 **제어권**도 함께 위임한 함수
- callback: 되돌아 호출해달라 ⇒ 어떤 함수 X를 호출하면서 '특정 조건일 때 함수 Y를 실행해서 나에게 알려달라'는 요청을 함께 보냄 ⇒ 이 요청을 받은 함수 X는 해당 조건이 갖춰졌는지 여부를 스스로 판단하고 Y를 직접 호출

# 02. 제어권

## 1. 호출 시점

- setInterval을 실행하면 반복적으로 실행되는 내용 자체를 특정할 수 있는 고유한 ID 값이 반환됨

    이를 변수에 담는 이유는 반복 실행되는 중간에 종료(clearInterval)할 수 있게 하기 위해서임

    ```jsx
    var count = 0;
    var timer = setInterval(function(){
        console.log(count);
        if (++count > 4) clearInterval(timer);
    }, 300);
    ```

- timer 변수에는 setInterval의 ID 값이 담김

    setInterval에 전달한 첫 번째 인자인 cbFunc 함수(콜백 함수)는 0.3초마다 자동으로 실행될 것

    ```jsx
    var count = 0;
    var cbFunc = function(){
        console.log(count);
        if (++count > 4) clearInterval(timer);
    };
    var timer = setInterval(cbFunc, 300);

    >>>
    0
    1
    2
    3
    4
    ```

- 콜백 함수의 제어권을 넘겨받은 코드(setInterval)는 **콜백 함수 호출 시점에 대한 제어권**을 가짐

    ```jsx
    **Code                       | 호출 주체     | 제어권**
    -------------------------------------------------------
    cbFunc();                  | 사용자       | 사용자
    -------------------------------------------------------
    setInterval(cbFunc, 300);  | setInterval | setInterval
    ```

## 2. 인자

- 콜백 함수의 제어권을 넘겨받은 코드(map)은 콜백 함수를 호출할 때 **인자에 어떤 값들을 어떤 순서로 넘길 것인지에 대한 제어권**을 가짐

    ```jsx
    // Array.prototype.map(callback[, thisArg])
    // callback: function(currentValue, index, array)

    var newArr = [10, 20, 30].map(function(currentValue, index){
        console.log(currentValue, index);
        return currentValue + 5;
    });

    console.log(newArr)
    >>>
    10 0
    20 1
    30 2
    [15, 25, 35]
    ```

# 03. 콜백 함수는 함수다

- 콜백 함수로 어떤 객체의 method를 전달하더라도 그 method는 method가 아닌 **function으로서 호출**됨
- forEach에 obj를 this로 하는 method(logValues)를 그대로 전달한 것이 아니라, obj.logValues가 가리키는 함수만 전달한 것

    이 함수는 method로서 호출할 때가 아닌 한, obj와의 직접적인 연관이 없어짐

    forEach에 의해 콜백이 함수로서 호출되고, 별도로 this를 지정하는 인자를 넘기지 않았으므로 함수 내부에서의 this는 전역객체를 바라봄

    ```jsx
    var obj = {
        logValues: function(v, i){
            console.log(this, v, i)
        }
    };

    obj.logValues(1, 2);
    >>>
    {logValues: f} 1 2

    [1, 2, 3].forEach(obj.logValues);
    >>>
    Window{...} 1 0
    Window{...} 2 1
    Window{...} 3 2
    ```

# 04. 콜백 함수 내부의 this에 다른 값 바인딩하기

- 콜백 함수 내부에서 this가 해당 객체를 바라보게 하고 싶다면?

    ⇒ 전통적으로는 this를 다른 변수에 담아 콜백 함수로 활용할 함수에서는 this 대신 그 변수를 사용하게 하고, 이를 클로저로 만드는 방식을 사용

    그러나 이 방식은 실제로 this를 사용하지 않을뿐더러 번거롭다.

    ```jsx
    var obj = {
        name: 'obj1',
        func: function(){
            var self = this;
            return function(){
                console.log(self.name);
            };
        }
    };

    var callback = obj.func();

    setTimeout(callback, 1000);
    >>> obj1  // 1초(1000ms) 뒤 실행
    ```

- this를 아예 안 쓰는 방법

    그러나 this를 이용해 다양한 상황에 재활용할 수 없다. (처음부터 바라볼 객체를 명시적으로 obj로 지정했기 때문에 어떤 방법으로도 다른 객체를 바라보게끔 할 수 없다.)

    ```jsx
    var obj = {
        name: 'obj1',
        func: function(){
            console.log(obj.name);
        }
    };

    setTimeout(obj.func, 1000);
    >>> obj1  // 1초(1000ms) 뒤 실행
    ```

- ES5에서 등장한 `bind` method 사용 (3장 참조)

    ```jsx
    var obj1 = {
        name: 'obj1',
        func: function(){
            console.log(this.name);
        }
    };

    setTimeout(obj1.func.bind(obj1), 1000);
    >>> obj1  // 1초(1000ms) 뒤 실행

    var obj2 = {name: 'obj2'};
    setTimeout(obj1.func.bind(obj2), 1500);
    >>> obj2  // 1.5초(1500ms) 뒤 실행
    ```

# 05. Callback hell과 비동기 제어

- **콜백 지옥(callback hell)**: 콜백 함수를 익명 함수로 전달하는 과정이 반복되어 코드의 들여쓰기 수준이 감당하기 힘들 정도로 깊어지는 현상

    주로 이벤트 처리나 서버 통신과 같이 비동기적인 작업을 수행하기 위해 이런 형태가 자주 등장

## 1. **Synchronous(동기)**

- 현재 실행 중인 코드가 완료된 후에야 다음 코드를 실행하는 방식
- CPU의 계산에 의해 **즉시** 처리가 가능한 대부분의 코드

    계산식이 복잡해서 CPU가 계산하는 데 시간이 많이 필요한 경우라 하더라도 동기적인 코드

## 2. **Asynchronous(비동기)**

- 현재 실행 중인 코드의 완료 여부와 무관하게 즉시 다음 코드로 넘어가는 방식
- 사용자의 요청에 의해 특정 시간이 경과되기 전까지 어떤 함수의 실행을 보류한다거나(setTimeout),

    사용자의 직접적인 개입이 있을 때 비로소 어떤 함수를 실행하도록 대기한다거나(addEventListener),

    웹브라우저 자체가 아닌 별도의 대상에 무언가를 요청하고 그에 대한 응답이 왔을 때 비로소 어떤 함수를 실행하도록 대기하는 등(XMLHttpRequest),

    **별도의 요청, 실행 대기, 보류** 등과 관련된 코드는 비동기적인 코드

- 현대의 JS는 웹의 복잡도가 높아진 만큼 비동기적인 코드의 비중이 예전보다 훨씬 높아짐 ⇒ 콜백 지옥에 빠지기 쉬움

- **Callback hell 예시**

    들여쓰기 수준이 과도하게 깊음 + 값이 전달되는 순서가 '아래에서 위로' 향하고 있어 어색

    ```jsx
    setTimeout(function(name){
        var coffeeList = name;
        console.log(coffeeList);

        setTimeout(function(name){
            coffeeList += ', ' + name;
            console.log(coffeeList);
        }, 500, 'Latte');
    }, 500, 'Americano');

    >>>
    Americano
    Americano, Latte
    ```

- **Callback hell 해결 (콜백 함수를 모두 기명함수로 전환)**

    코드의 가독성을 높일뿐 아니라 함수 선언과 함수 호출만 구분할 수 있다면 위에서부터 아래로 순서대로 읽어내려가는 데 어려움이 없음

    변수를 최상단으로 끌어올림으로써 외부에 노출하게 됐지만 전체를 즉시 실행 함수 등으로 감싸면 해결 가능

    But, 일회성 함수를 전부 변수에 할당하면 코드명을 일일이 따라다녀야 하므로 헷갈릴 소지가 있음

    ```jsx
    var coffeeList = '';

    var addAmericano = function(name){
        coffeeList = name;
        console.log(coffeeList);
        setTimeout(addLatte, 500, 'Latte');
    };

    var addLatte = function(name){
        coffeeList += ', ' + name;
        console.log(coffeeList);
    };

    setTimeout(addAmericano, 500, 'Americano')

    >>>
    Americano
    Americano, Latte
    ```

## 3. 비동기 작업의 동기적 표현

### 1) ES6의 Promise

- new 연산자와 함께 호출한 Promise의 인자로 넘겨주는 콜백 함수는 호출할 때 바로 실행

    다만 그 내부에 resolve 또는 reject 함수를 호출하는 구문이 있을 경우에는, 둘 중 하나가 실행되기 전까지는 다음(then) 또는 오류 구문(catch)로 넘어가지 않음

- 비동기 작업이 완료될 때 비로소 resolve 또는 reject를 호출하는 방법으로 비동기 작업의 동기적 표현이 가능

    ```jsx
    new Promise(function(resolve){
        setTimeout(function(){
            var name = 'Americano';
            console.log(name);
            resolve(name);
        }, 500);
    }).then(function(prevName){
        return new Promise(function(resolve){
            setTimeout(function(){
                var name = prevName + ', Latte';
                console.log(name);
                resolve(name);
            }, 500);
        });
    });

    >>>
    Americano
    Americano, Latte
    ```

    ```jsx
    // 위의 반복적인 내용을 함수화 (클로저는 5장 참조)

    var addCoffee = function(name){
        return function(prevName){
            return new Promise(function(resolve){
                setTimeout(function(){
                    var newName = prevName ? (prevName + ', ' + name) : name;
                    console.log(newName);
                    resolve(newName);
                }, 500);
            });
        };
    };

    addCoffee('Americano')()
        .then(addCoffee('Latte'));
    >>>
    Americano
    Americano, Latte
    ```

### 2) ES6의 Generator

- `*`이 붙은 함수가 Generator 함수
- Generator 함수를 실행하면 Iterator가 반환되는데, Iterator는 next라는 method를 가지고 있음

    ⇒ next method를 호출하면 Generator 함수 내부에서 가장 먼저 등장하는 yield에서 함수 실행을 멈춤

    ⇒ 이후 다시 next method를 호출하면 앞서 멈췄던 부분부터 시작해서 그 다음에 등장하는 yield에서 함수 실행을 멈춤

- 즉, 비동기 작업이 완료되는 시점마다 next method를 호출하면, Generator 함수 내부의 소수가 위에서부터 아래로 순차적으로 진행됨

    ```jsx
    var addCoffee = function(prevName, name){
        setTimeout(function(){
            coffeeMaker.next(prevName ? prevName + ', ' + name : name);
        }, 500);
    };

    var coffeeGenerator = function*(){
        var americano = yield addCoffee('', 'Americano');
        console.log(americano);
        var latte = yield addCoffee(americano, 'Latte');
        console.log(latte);
    };

    var coffeeMaker = coffeeGenerator();
    coffeeMaker.next();

    >>>
    Americano
    Americano, Latte
    ```

### 3) ES2017의 async/await

- 가독성이 뛰어나면서 작성법도 간단
- 비동기 작업을 수행하고자 하는 함수 앞에 `async`를 표기하고, 함수 내부에서 실직적인 비동기 작업이 필요한 위치마다 `await`를 표기

    ⇒ 뒤의 내용을 Promise로 자동 전환하고, 해당 내용이 resolve된 이후에야 다음으로 진행 (Promise의 then과 흡사한 효과)

    ```jsx
    var addCoffee = function(name){
        return new Promise(function(resolve){
            setTimeout(function(){
                resolve(name);
            }, 500);
        });
    };

    var coffeeMaker = async function(){
        var coffeeList = '';
        var _addCoffee = async function(name){
            coffeeList += (coffeeList ? ',' : '') + await addCoffee(name);
        };
        await _addCoffee('Americano');
        console.log(coffeeList);
        await _addCoffee('Latte');
        console.log(coffeeList);
    };

    coffeeMaker();
    >>>
    Americano
    Americano, Latte
    ```
