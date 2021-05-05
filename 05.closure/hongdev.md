# 01. Closure의 의미 및 원리

- 함수형 프로그래밍 언어에서 등장하는 보편적인 특성 (JS 고유의 개념은 아님)
- **Closure(클로저):** 어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상

- **외부 함수의 변수를 참조하는 내부함수 1:**

    inner 함수에서는 a를 선언하지 않았기 때문에 `environmentRecord`에서 값을 찾지 못하므로 `outerEnvironmentReference`에 지정된 상위 context인 outer의 `LexicalEnvironment`에 접근해서 a를 찾음

    outer 함수의 실행 context가 종료되면 LexicalEnvironment에 저장된 식별자들(a, inner)에 대한 참조를 지움

    그러면 각 주소에 저장되어 있던 값들은 자신을 참조하는 변수가 하나도 없게 되므로 garbage collector의 수집 대상이 됨

    ⇒ outer 함수의 실행 context가 종료되기 이전에 inner 함수의 실행 context가 종료되어 있으며, 이후 별도로 inner 함수를 호출할 수 없음

    ```jsx
    var outer = function(){
        var a = 1;
        var inner = function(){
            console.log(++a);     //2
        };
        inner();
    };
    outer();
    ```

- **외부 함수의 변수를 참조하는 내부함수 2:**

    위의 예시와 마찬가지로 outer 함수는 inner 함수를 **실행한 결과**를 return 하고 있으므로, 결과적으로 outer 함수의 실행 context가 종료된 시점에는 a 변수를 참조하는 대상이 없어짐

    a와 inner 변수의 값들은 언젠가 garbage collector에 의해 소멸

    ⇒ 역시 outer 함수의 실행 context가 종료되면 이후 별도로 inner 함수를 호출할 수 없음

    ```jsx
    var outer = function(){
        var a = 1;
        var inner = function(){
            return ++a;
        };
        return inner();
    };

    var outer2 = outer();
    console.log(outer2);    //2
    ```

- **외부 함수의 변수를 참조하는 내부함수 3:**

    outer 함수에서 inner 함수의 실행 결과가 아닌 inner **함수 자체**를 return

    outer 함수의 실행 context가 종료될 때, outer2의 변수는 outer의 실행 결과인 inner 함수를 참조

    ⇒ outer2를 호출하면 앞서 반환된 함수인 inner가 실행

    ```jsx
    var outer = function(){
        var a = 1;
        var inner = function(){
            return ++a;
        };
        return inner;
    };

    var outer2 = outer();
    console.log(outer2());    //2
    console.log(outer2());    //3
    ```

- 위의 예제에서, inner 함수에서는 a를 선언하지 않았기 때문에 `environmentRecord`에서 값을 찾지 못하므로 `outerEnvironmentReference`에 지정된 상위 context인 outer의 `LexicalEnvironment`에 접근해서 a를 찾음

    그런데 inner 함수의 실행 시점에는 outer 함수는 이미 실행이 종료된 상태인데 outer 함수의 `LexicalEnvironment`에 어떻게 접근할 수 있는 걸까?

    ⇒ **Garbage Collector**의 동작 방식 때문

- 가비지 컬렉터는 어떤 값을 참조하는 변수가 하나라도 있다면 그 값은 수집 대상에 포함시키지 않음
즉, 어떤 함수의 `LexicalEnvirionment`가 이를 '참조할' 예정인 다른 실행 context가 있는 한 실행 종료 이후에도 GC되지 않음

    ⇒ 위의 예시에서는 outer 함수가 실행 종료 시점에 inner 함수를 반환하므로, inner 함수는 언제가 호출될 가능성이 생겼고, inner 함수의 실행 context가 활성화되면 `outerEnvironmentReference`가 outer 함수의 `LexicalEnvironment`를 필요로 할 것이므로 수집 대상에서 제외됨. 그래서 inner 함수가 이 변수에 접근할 수 있는 것

- 스펙상으로는 (`envrionmentRecord`와 `outerEnvironmentReference`를 포함하는) `LexicalEnvironment` 전체를 GC하지 않고 남겨두도록 되어 있으나, 2019년 기준으로 크롬이나 Node.js 등에서 사용 중인 V8 엔진의 경우, 내부 함수에서 실제로 사용하는 변수만 남겨두고 나머지는 GC하도록 최적화 되어 있음.

    ⇒ 즉, 위의 예시에서는 outer 함수의 `outerEnvironmentReference`와 `environmentRecord`의 inner 함수는 GC 되고, `environmentRecord`의 a 변수만 GC에서 제외됨

- **return 없이도 closure가 발생하는 경우**

    ```jsx
    // setInterval, setTimeout
    // 별도의 외부객체인 window의 method(setTimeout 또는 setInterval)에 전달할 콜백 함수 내부에서 지역변수를 참조

    // 실행???

    (function(){
        var a = 0;
        var intervalId = null;
        var inner = function(){
            if(++a >= 3){
                clearInterval(intervalId);
            }
            console.log(a);
        };
        intervalId = setInterval(inner, 1000);
    })();

    >>>
    1
    2
    3
    ```

    ```jsx
    // eventListener
    // 별도의 외부객체인 DOM의 method(addEventListener)에 등록할 handler 함수 내부에서 지역변수를 참조

    (function(){
        var count = 0;
        var button = document.createElement('button');
        button.innerText = 'click';
        button.addEventListener('click', function(){
            console.log(++count, 'times clicked');
        });
        document.body.appendChild(button);
    })();

    >>>
    1 "times clicked"
    2 "times clicked"
    3 "times clicked"
    ```

## Closure

- 어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상
- 외부 함수의 LexicalEnvironment가 Garbage Collecting 되지 않는 현상 (지역변수를 참조하는 **내부함수가 외부로 전달**된 경우)
- **어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우, A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상**
- 함수를 선언할 때 만들어지는 유효범위가 사라진 후에도 호출할 수 있는 함수 (Ref. <자바스크립트 닌자 비급>, 존 레식)
- 이미 생명 주기가 끝난 외부 함수의 변수를 참조하는 함수 (Ref. <인사이드 자바스크립트>, 송형주 고현준)
- 자신이 생성될 때의 스코프에서 알 수 있었던 변수들 중 언젠가 자신이 실행될 때 사용할 변수들만을 기억하여 유지시키는 함수 (Ref. <함수형 자바스크립트 프로그래밍>, 유인동)

# 02. Closure와 메모리 관리

- 메모리 누수의 위험을 이유로 클로저 사용을 조심해야 한다거나 심지어 지양해야 한다고 주장하는 사람들도 있지만, 메모리 소모는 closure의 본질적인 특성일 뿐이므로, 오히려 이러한 특성을 정확히 이해하고 잘 활용하도록 해야 함
- Closure는 어떤 필요에 의해 의도적으로 함수의 지역변수를 메모리를 소모하도록 함으로써 발생하므로, 그 필요성이 사라진 시점에는 더는 메모리를 소모하지 않게 해주면 됨

    ⇒ 참조 카운트를 0으로 만들어서 언젠가 GC가 수거해 가도록 만듦. 식별자에 참조형이 아닌 기본형 데이터(보통 null이나 undefined)를 할당

    ```jsx
    // return에 의한 closure의 메모리 해제

    var outer = (function(){
        var a = 1;
        var inner = function(){
            return ++a;
        };
        return inner;
    })();

    console.log(outer());
    console.log(outer());
    outer = null;    // outer 식별자의 inner 함수 참조를 끊음
    ```

    ```jsx
    // setInterval에 의한 closure의 메모리 해제

    (function(){
        var a = 0;
        var intervalId = null;
        var inner = function(){
            if(++a >= 3){
                clearInterval(intervalId);
                inner = null;      // inner 식별자의 함수 참조를 끊음
            }
            console.log(a);
        };
        intervalId = setInterval(inner, 1000);
    })();
    ```

    ```jsx
    // eventListener에 의한 closure의 메모리 해제

    (function(){
        var count = 0;
        var button = document.createElement('button');
        button.innerText = 'click';

        var clickHandler = function(){
            console.log(++count, 'times clicked');
            if (count >= 5){
                button.removeEventListener('click', clickHandler);
                clickHandler = null;     // clickHandler 식별자의 함수 참조를 끊음
            }
        };

        button.addEventListener('click', clickHandler);
        document.body.appendChild(button);
    })();
    ```

# 03. Closure 활용 사례

## 1. Callback function 내부에서 외부 데이터를 사용하고자 할 때

### [Example 1]

- fruits의 변수를 순회하며 li를 생성하고, 각 li를 클릭하면 해당 Listener에 기억된 callback 함수를 실행

    함수(A)는 fruits의 개수만큼 실행되며, 그때마다 새로운 실행 context가 활성화될 것

    (A)의 실행 종료 여부와 무관하게 클릭 이벤트에 의해 각 context의 (B)가 실행될 때는 (B)의 outerEnvrionmentReference가 (A)의 LexicalEnvrionment를 참조

    ⇒ 최소한 (B) 함수가 참조할 예정인 변수 fruit에 대해서는 (A)가 종료된 후에도 GC 대상에서 제외되어 계속 참조 가능할 것임

    ```jsx
    var fruits = ['apple', 'banana', 'peach'];
    var $ul = document.createElement('ul');         //공통 코드

    fruits.forEach(function(fruit){                 //(A) 클로저 X
        var $li = document.createElement('li');
        $li.innerText = fruit;
        $li.addEventListener('click', function(){   //(B) 클로저 O
            alert('your choice is ' + fruit);
        });
        $ul.appendChild($li);
    });

    document.body.appendChild($ul);
    ```

### [Example 2]

- Example 1에서 (B) 함수의 쓰임새가 callback 함수에 국한되지 않는 경우라면, 반복을 줄이기 위해 (B)를 외부로 분리하는 편이 나을 수도 있음. 즉, fruit를 인자로 받아 출력하도록 만듦

    그러나 직접 호출할 경우에는 정상적으로 실행되지만, li를 클릭한다면 `you choice is [object MouseEvent]` 라는 메시지가 출력됨

    그 이유는 callback function의 인자에 대한 제어권을 addEventListener가 가진 상태이며, addEventListener는 콜백 함수를 호출할 때 첫 번째 인자에 '이벤트 객체'를 주입하기 때문

    ```jsx
    var fruits = ['apple', 'banana', 'peach'];
    var $ul = document.createElement('ul');

    var alertFruit = function(fruit){
        alert('your choice is ' + fruit);
    };

    fruits.forEach(function(fruit){
        var $li = document.createElement('li');
        $li.innerText = fruit;
        $li.addEventListener('click', alertFruit);
        $ul.appendChild($li);
    });

    document.body.appendChild($ul);

    alertFruit(fruits[1]);
    >>> your choice is banana
    ```

### [Example 3]

- Example 2의 문제점은 bind method를 활용해서 해결 가능

    다만, 이벤트 객체가 인자로 넘어오는 순서가 바뀌는 점 및 함수 내부에서의 this가 원래의 그것과 달라지는 점은 감안해야 함

    (bind method의 첫 번째 인자가 바로 새로 바인딩할 this인데, 이 값을 생략할 수 없기 때문에 일반적으로 원래의 this를 유지하도록 할 수 없는 경우가 많음. 또한 예제에서는 두 번째 인자에 이벤트 객체가 넘어올 것임)

    ```jsx
    var fruits = ['apple', 'banana', 'peach'];
    var $ul = document.createElement('ul');

    var alertFruit = function(fruit){
        alert('your choice is ' + fruit);
    };

    fruits.forEach(function(fruit){
        var $li = document.createElement('li');
        $li.innerText = fruit;
        $li.addEventListener('click', alertFruit.bind(null, fruit));
        $ul.appendChild($li);
    });

    document.body.appendChild($ul);
    ```

### [Example 4]

- Example 3에서 발생하는 변경사항이 발생하지 않도록 **고차함수**(함수를 인자로 받거나 함수를 리턴하는 함수)를 활용

    `alertFruit(fruit)`의 실행 결과가 다시 함수가 되며, 이렇게 반환된 함수를 Listener에 callback 함수로써 전달

    이후 클릭 이벤트가 발생하면 비로소 이 함수의 실행 context가 열리면서 alertFruit의 인자로 넘어온 fruit를 outerEnvironmentReference에 의해 참조 가능 (즉, alertFruit의 실행 결과로 반환된 함수에는 **클로저가 존재**)

    ```jsx
    var fruits = ['apple', 'banana', 'peach'];
    var $ul = document.createElement('ul');

    var alertFruit = function(fruit){
        return function(){
            alert('your choice is ' + fruit);
        };
    };      //익명 함수 자체를 반환

    fruits.forEach(function(fruit){
        var $li = document.createElement('li');
        $li.innerText = fruit;
        $li.addEventListener('click', alertFruit(fruit));
        $ul.appendChild($li);
    });

    document.body.appendChild($ul);
    ```

## 2. 접근 권한 제어 (정보 은닉)

- **Information hiding(정보 은닉):** 어쩐 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서 모듈 간의 결합도를 낮추고 유연성을 높이고자 함
- 접근 권한: public, private, protected
- JS는 기본적으로 변수 자체에 접근 권한을 직접 부여하도록 설계되어 있지 않지만, closure를 이용하면 함수 차원에서 public/private한 값 구분 가능

- outer 함수를 종료할 때 inner 함수를 반환함으로써 outer 함수의 지역변수의 a의 값을 외부에서도 읽을 수 있게 됨

    이처럼 closure를 활용하면 외부 스코프에서 함수 내부의 변수들 중 선택적으로 일부의 변수에 대한 접근 권한 부여 가능

    외부에서는 outer 함수 내부에는 개입할 수 없고 오직 outer 함수가 **return**한 정보에만 접근할 수 있기 때문에, 외부에 제공하고자 하는 정보들은 return하고 내부에서만 사용할 정보는 return하지 않는 것으로 접근 권한 제어 가능

    ```jsx
    var outer = function(){
        var a = 1;
        var inner = function(){
            return ++a;
        };
        return inner;
    };

    var outer2 = outer();

    console.log(outer2());
    console.log(outer2());
    >>>
    2
    3
    ```

### Closure를 활용한 접근 권한 제어

1. 함수에서 지역변수 및 내부함수 등을 생성
2. 외부에 접근권한을 주고자하는 대상들로 구성된 참조형 데이터(대상이 여럿일 때는 객체 또는 배열, 하나일 때는 함수)를 return

    ⇒ return한 변수들은 public member가 되고, 그렇지 않은 변수들은 private member가 됨

### [Example 1: 간단한 자동차 객체]

- **car 변수에 객체를 직접 할당.** fuel를 무작위로 생성하고, run method를 실행할 때마다 fuel 값 변경
- 그러나 `car.fuel = 10000;` 식으로 무작위로 정해지는 값들을 마음대로 바꿀 가능성이 있음

    ```jsx
    var car = {
        fuel: Math.ceil(Math.random() * 100),
        run: function(){
            var km = Math.ceil(Math.random() * 10);
            if (this.fuel < km){
                console.log('이동불가');
                return;
            }
            this.fuel -= km;
            console.log(km + 'km 이동 (남은 연료 ' + this.fuel + ')');
        }
    };
    ```

### [Example 2: Closure로 변수 보호(1)]

- 객체가 아닌 함수로 만들고, 필요한 member만을 return

    외부에서는 오직 run method를 실행하는 것과 현재 남은 연료를 확인하는 left 두 가지 동작만 가능 (fuel 변수는 getter만을 부여함으로써 읽기 전용 속성 부여)

    ```jsx
    var createCar = function(){
        var fuel = Math.ceil(Math.random() * 100);
        return {
            get left(){
                return fuel;
            },
            run: function(){
                var km = Math.ceil(Math.random() * 10);
                if (fuel < km){
    		            console.log('이동불가');
    		            return;
    		        }
    		        fuel -= km;
    		        console.log(km + 'km 이동 (남은 연료 ' + fuel + ')');
            }
    		};
    };

    var car = createCar();
    ```

- 위와 같이 작성함으로써 fuel의 값을 변경하려는 시도는 대부분 실패하게 됨

    비록 run method를 다른 내용으로 덮어씌우는 어뷰징(abusing)은 여전히 가능한 상태이긴 하지만, Example 1보다는 훨씬 안전한 코드

    ```jsx
    console.log(car.left)    // 95
    car.left = 1000;
    console.log(car.left)    // 95

    console.log(car.fuel)    // undefined
    car.fuel = 1000;
    console.log(car.fuel)    // 1000
    car.run();               // 3km 이동 (남은 연료 92)

    car.run = 'aaa'
    console.log(car.run)     // 'aaa'
    car.run();               // Uncaught TypeError: car.run is not a function
    ```

### [Example 3: Closure로 변수 보호(2)]

- run method를 덮어씌우는 abusing까지 막기 위해서, 객체를 return하기 전에 미리 변경할 수 없게끔 조치를 취해야 함

    ```jsx
    var createCar = function(){
        var fuel = Math.ceil(Math.random() * 100);

        var publicMembers = {
            get left(){
                return fuel;
            },
            run: function(){
                var km = Math.ceil(Math.random() * 10);
                if (fuel < km){
    		            console.log('이동불가');
    		            return;
    		        }
    		        fuel -= km;
    		        console.log(km + 'km 이동 (남은 연료 ' + fuel + ')');
            }
    		};
        Object.freeze(publicMembers);
        return publicMembers;
    };

    var car = createCar();
    ```

    ```jsx
    car.run = 'aaa'
    console.log(car.run)    // f(){...}
    car.run();              // 2km 이동 (남은 연료 60)
    ```

## 3. 부분 적용 함수

- **Partially applied function (부분 적용 함수):** n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게끔 하는 함수

- this를 바인딩해야 하는 점을 제외하면, `bind` method 또한 부분 적용 함수 (3장 참조)

    ```jsx
    var add = function(){
        var result = 0;
        for (var i = 0; i < arguments.length; i++){
            result += arguments[i];
        }
        return result;
    };

    var addPartial = add.bind(null, 1, 2);

    console.log(addPartial(3, 4, 5));
    >>> 15
    ```

### [Example 1: 부분 적용 함수]

- `bind`는 this의 값을 변경할 수밖에 없기 때문에 method에서는 사용하기 어려움

    this에 관여하지 않는 별도의 부분 적용 함수를 구현한다면:
    첫 번째 인자에는 원본 함수를, 두 번째 인자 이후부터는 미리 적용할 인자를 전달하고, 반환할 함수(부분 적용 함수)에서는 다시 나머지 인자들을 받아 이들을 concat해서 원본 함수에 apply

    실행 시점의 this를 그대로 반영함으로써 this에는 아무런 영향을 주지 않게 됨

    ```jsx
    var partial = function(){
        var originalPartialArgs = arguments;
        var func = originalPartialArgs[0];
        if (typeof func !== 'function'){
            throw new Error('첫 번째 인자가 함수가 아닙니다.');
        }
        return function(){
            var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
            var restArgs = Array.prototype.slice.call(arguments);
            return func.apply(this, partialArgs.concat(restArgs));
        };
    };
    ```

    ```jsx
    // add 함수에 적용

    var add = function(){
        var result = 0;
        for (var i = 0; i < arguments.length; i++){
            result += arguments[i];
        }
        return result;
    };

    var addPartial = partial(add, 1, 2);

    console.log(addPartial(3, 4, 5));
    >>> 15
    ```

    ```jsx
    // object의 method에 적용

    var dog = {
        name: '강아지',
        greet: partial(function(prefix, suffix){
            return prefix + this.name + suffix;
        }, '왈왈, ')
    };

    dog.greet('입니다!');
    >>>
    "왈왈, 강아지입니다!"
    ```

### [Example 2: 원하는 위치에 따라 인자를 전달할 수 있는 부분 적용 함수]

- 인자들을 원하는 위치에 미리 넣어놓고 나중에는 빈 자리에 인자를 채워넣어 실행할 수 있도록 구현
- '비워놓음'을 표시하기 위해 미리 전역객체에 `_`라는 property를 준비하면서 삭제 변경 등의 접근에 대한 방어 차원에서 여러 가지 property 속성 설정
- 부분 적용 함수를 만들 때 미리부터 실행할 함수의 모든 인자 개수를 맞춰 빈 공간을 확보하지 않아도, 최종 실행 시 인자 개수가 많든 적든 잘 실행됨

    ```jsx
    Object.defineProperty(window, '_', {
        value: 'EMPTY_SPACE',
        writable: false,
        configurable: false,
        enumerable: false
    });

    var partial = function(){
        var originalPartialArgs = arguments;
        var func = originalPartialArgs[0];
        if (typeof func !== 'function'){
            throw new Error('첫 번째 인자가 함수가 아닙니다.');
        }
        return function(){
            var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
            var restArgs = Array.prototype.slice.call(arguments);
            for (var i = 0; i < partialArgs.length; i++){
                if (partialArgs[i] === _) {
                    partialArgs[i] = restArgs.shift();
                }
            }
            return func.apply(this, partialArgs.concat(restArgs));
        };
    };
    ```

    ```jsx
    // add 함수에 적용

    var add = function(){
        var result = 0;
        for (var i = 0; i < arguments.length; i++){
            result += arguments[i];
        }
        return result;
    };

    var addPartial = partial(add, 1, _, 3, _, _);

    console.log(addPartial(2, 4, 5));
    >>> 15
    ```

- ES5 환경에서는 `_`를 '비워놓음'으로 사용하기 위해 어쩔 수 없이 전역공간을 침범

    ES6에서는 `Symbol.for`를 활용 가능

    `Symbol.for` method는 전역 심볼공간에 인자로 넘어온 문자열이 이미 있으면 해당 값을 참조하고, 선언되어 있지 않으면 새로 만드는 방식으로, 어디서든 접근 가능하면서 유일무이한 상수를 만들고자 할 때 적합

    ```jsx
    var partial = function(){
        var originalPartialArgs = arguments;
        var func = originalPartialArgs[0];
        if (typeof func !== 'function'){
            throw new Error('첫 번째 인자가 함수가 아닙니다.');
        }
        return function(){
            var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
            var restArgs = Array.prototype.slice.call(arguments);
            for (var i = 0; i < partialArgs.length; i++){
                if (partialArgs[i] === Symbol.for('EMPTY_SPACE')) {
                    partialArgs[i] = restArgs.shift();
                }
            }
            return func.apply(this, partialArgs.concat(restArgs));
        };
    };
    ```

    ```jsx
    // add 함수에 적용

    var add = function(){
        var result = 0;
        for (var i = 0; i < arguments.length; i++){
            result += arguments[i];
        }
        return result;
    };

    var _ = Symbol.for('EMPTY_SPACE')
    var addPartial = partial(add, 1, _, 3, _, _);

    console.log(addPartial(2, 4, 5));
    >>> 15
    ```

### [Example 3: Debounce]

- **Debounce(디바운스):** 짧은 시간 동안 동일한 이벤트가 많이 발생하는 경우 이를 전부 처리하지 않고 처음 또는 마지막에 발생한 이벤트에 대해 한 번만 처리하는 것으로, Frontend 성능 최적화에 큰 도움을 주는 기능 중 하나

    scroll, wheel, mousemove, resize 등에 적용하기 좋음

- 최초 event가 발생하면 `setTimeout`에 의해 timeout의 대기열에 'wait 시간 뒤에 func을 실행할 것'이라는 내용이 담김

    그런데 wait 시간이 경과하기 전에 다시 동일한 event가 발생하면 `clearTimeout`에 의해 앞서 저장했던 대기열을 초기화하고 다시 새로운 대기열을 `setTimeout`으로 등록

    아래의 디바운스 함수에서 클로저로 처리되는 변수는 eventName, func, wait, timeoutId

    ```jsx
    var debounce = function(eventName, func, wait){
        var timeoutId = null;
        return function(event){                                          //closure로 EventListener에 의해 호출될 함수 반환
            var self = this;                                             //setTimeout를 사용하기 위해 this를 별도의 변수에 담음
            console.log(eventName, 'event 발생');
            clearTimeout(timeoutId);                                     //무조건 대기큐를 초기화
            timeoutId = setTimeout(func.bind(self, event), wait);        //setTimeout으로 wait 시간만큼 지연시킨 후 원래의 func 호출
        };
    };

    var moveHandler = function(e){
        console.log('move event 처리');
    };

    document.body.addEventListener('mousemove', debounce('move', moveHandler, 500));
    ```

## 4. Currying function

- **Currying function(커링 함수):** 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성

    **한 번에 하나의 인자만 전달하는 것이 원칙이며, 중간 과정상의 함수를 실행한 결과는 그 다음 인자를 받기 위해 대기만 할 뿐으로, 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않음**

    (부분 적용 함수는 여러 개의 인자를 전달할 수 있고, 실행 결과를 재실행할 때 원본 함수가 무조건 실행됨)

- **Lazy execution(지연실행):** 당장 필요한 정보만 받아서 전달하고 또 필요한 정보가 들어오면 전달하는 식으로 하면 결국 마지막 인자가 넘어갈 때까지 함수 실행을 미루게 되는 것

    원하는 시점까지 지연시켰다가 실행하는 것이 요긴한 경우, 혹은 자주 쓰이는 함수의 매개변수가 항상 비슷하고 일부만 바뀌는 경우 등에 유용하게 사용될 수 있음

- 부분 적용 함수와 달리 커링 함수는, 필요한 인자 개수만큼 함수를 만들어 계속 리턴해주다가 마지막에만 조합해서 return해주면 되기 때문에 필요한 상황에 직접 만들어 쓰기가 용이

    ```jsx
    var curry = function(func){
        return function(a){
            return function(b){
                return func(a, b);
            };
        };
    };

    var getMaxWith10 = curry(Math.max)(10);
    console.log(getMaxWith10(8));    //10
    console.log(getMaxWith10(25));   //25

    var getMin = curry(Math.min);
    console.log(getMin(15)(30));     //15
    ```

- 다만 인자가 많아질수록 가독성이 떨어진다는 단점이 있는데, ES6에서는 화살표 함수를 활용하여 한 줄 구현 가능

    화살표 순서에 따라 함수에 값을 차례로 넘겨주면 마지막에 func가 호출될 거라는 흐름이 한눈에 파악됨

    각 단계에서 받은 인자들을 모두 마지막 단계에서 참조할 것이므로 GC되지 않고 메모리에 차곡차곡 쌓였다가, 마지막 호출로 실행 context가 종료된 후에야 비로소 한꺼번에 GC의 수거 대상이 됨

    ```jsx
    var getInfo = function(baseUrl){
        return function(path){
            return function(id){
                return fetch(baseUrl + path + '/' + id);
            };
        };
    };

    // ES6
    var getInfo = baseUrl => path => id => fetch(baseUrl + path + '/' + id);

    var imageUrl = 'http://imageAddress.com/';
    var getImage = getInfo(imageUrl);            // http://imageAddress.com/
    var getEmoji = getImage('emoji');            // http://imageAddress.com/emoji
    var getIcon = getImage('icon');              // http://imageAddress.com/icon

    // 실제요청
    var emoji1 = getEmoji(100);                  // http://imageAddress.com/emoji/100
    var icon1 = getIcon(253);                    // http://imageAddress.com/icon/253
    ```
