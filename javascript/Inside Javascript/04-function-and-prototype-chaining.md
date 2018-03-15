# 04. 함수와 프로토타입 체이닝

## 함수 정의

자바스크립트에서 함수를 생성하는 방법은 3가지가 있다. 이들 방식은 모두 같은 함수를 생성하지만, 각각의 방식에 따라 함수 동작이 미묘하게 차이가 난다.

- 함수 선언문(function statement)
- 함수 표현식(function expression)
- Function() 생성자 함수

함수 선언문이나 함수 표현식 방법은 함수 리터럴 방식으로 함수를 생성한다.


**1. 함수 선언문 방식**

```javascript
function add(x, y) {
    return x + y;
}

console.log(add(3, 4)); // 7
```

**2. 함수 표현문 방식**

```javascript
var add = function (x, y) {
    return x + y;
}

var plus = add;

console.log(add(3, 4)); // 7
console.log(plus(5, 6)); // 11
```

함수 표현식에서 사용된 함수 이름은 정의된 함수 내부에서 재귀적으로 호출하거나, 디버거 등에서 함수를 구분할 때 사용되며 외부 코드에서 접근이 불가능 하다. **함수 선언문** 형식으로 정의된 add() 함수는 자바스크립트 엔진에 의해 다음과 같이 함수 표현식 형태로 변경된다.

```javascript
// 익명 함수 표현식(기명 함수도 가능)
var add = function add(x, y) {
    return x + y;
}
```

**3. Function() 생성자 함수 방식**

```javascript
var add = new Function('x', 'y', 'return x + y');

console.log(add(3, 4)); // 7
```

함수 선언문이나 함수 표현식 방식도 결국 내부적으로는 Fuction() 생성자 함수로 함수가 생성된다.

**함수 호이스팅**

위에서 살펴본 함수를 생성하는 3가지 방식(함수 선언문, 함수 표현식, 생성자 함수) 사이에는 약간씩 차이가 있다. 그 중의 하나가 함수 호이스팅(Function Hoisting) 이다.

자바스크립트 Guru로 알려진 더글라스 크락포드는 함수 생성에 있어서 그의 저서 더글라스 크락포드의 자바스크립트 핵심가이드 에서 함수 호이스팅은 코드의 구조를 엉성하게 만들 수도 있다고 지적하며, **함수 표현식만을 사용할 것을 권하고 있다.**

```javascript
add(2, 3); //5

// 함수 선언문 형태로 add() 함수 정의
function add(x, y) {
    return x + y;
}

add(3, 4); //7
```

위의 예제처럼 함수가 자신이 위치한 코드에 상관없이 **함수 선언문 형태로 정의한 함수의 유효 범위는 코드의 맨 처음부터 시작한다.** 이것을 **함수 호이스팅** 이라고 부른다.

```javascript
add(2, 3); // uncaught type error

// 함수 표현식 형태로 add() 함수 정의
var add = function(x, y) {
    return x + y;
}

add(3, 4); // 7
```

위의 예제는 함수 표현식 형태로 정의되어 있어 호이스팅이 일어나지 않는다. 이러한 함수 호이스팅이 발생하는 원인은 **자바스크립트의 변수 생성(Instantiation) 과 초기화(Initialization) 의 작업이 분리돼서 진행** 되기 때문이다(chap5에서 다룸).

## 함수 객체: 함수도 객체다

자바스크립트에서 함수도 객체이므로 일반 객체처럼 취급될 수 있다. 또한 함수는 다음과 같은 동작이 가능하다.

- 리터럴에 의해 생성
- 변수나 배열의 요소, 객체의 프로퍼티 등에 할당 가능
- 함수의 인자로 전달 가능
- 함수의 리턴값으로 리턴 가능
- 동적으로 프로퍼티를 생성 및 할당 가능

이와 같은 특징이 있으므로 자바스크립트에서는 함수를 일급 객체 라고 부르며, 이런 특성으로 함수형 프로그래밍이 가능하다.

```javascript
// 함수 선언 방식응로 add() 함수 정의
function add(x, y) {
    return x + y;
}

add.result = add(2, 3);
add.status = 'OK';

console.log(add.result); // 5
console.log(add.status); // 'OK'
```

위의 예제는 함수에 동적으로 프로퍼티를 생성할 수 있다는 것을 보여준다. **함수 코드는 함수 객체의 [[Code]] 내부 프로퍼티 에 자동으로 저장된다(ECMAScript 명세).**

함수는 일반 객체와는 다르게 추가로 함수 객체만의 표준 프로퍼티가 정의되어 있다. ECMA5 스크립트 명세서에는 **모든 함수가 length와 prototype 프로퍼티를 가져야 한다고** 기술하고 있다. 그 외에 name, caller, arguments, \_\_proto\_\_ 프로퍼티가 있으나 이들은 ECMA 표준이 아니다. ECMA 표준에서는 arguments 프로퍼티와 같은 이름으로 arguments 객체를 정의하고 있다.

- name : 함수의 이름
- caller : 자신을 호출한 함수
- arguments : 함수를 호출할 때 전달된 인자값
- \_\_proto\_\_ : 객체 자신이 가리키는 프로토타입

모든 함수들의 부모 객체는 Function Prototype 객체이다. 그런데 ECMA 명세서에는 Function.prototype도 역시 함수라고 정의하고 있다. 대신, 예외적으로 Function.prototype 함수 객체의 부모는 자바스크립트의 모든 객체의 조상격인 Object.prototype 객체라고 설명하고 있다. Function.prototype 객체가 가지는 프로퍼티나 메서드는 다음과 같다.

- constructor 프로퍼티
- toString() 메서드
- apply(thisArg, argArray) 메서드
- call(thisArg, [,arg1 [,arg2, ]]) 메서드
- bind(thisArg, [,arg1 [,arg2, ]]) 메서드

함수 객체의 length 프로퍼티는 앞서 설명했듯이 ECMAScript에서 정한 모든 함수가 가져야 하는 표준 프로퍼티로서, 함수가 정상적으로 실행될 때 기대되는 인자의 개수를 나타낸다.

```javascript
function func0() {}
function func1(x) { return x; }
function func2(x, y) { return x + y; }
function func3(x, y, z) { return x + y + z; }

console.log(func0.length); // 0
console.log(func1.length); // 1
console.log(func2.length); // 2
console.log(func3.length); // 3
```

모든 객체에 있는 내부 프로퍼티인 [[Prototype]]는 객체 입장에서 자신의 부모 역할을 하는 프로토타입 객체를 가리키는 반면에, 함수 표준 prototype 프로퍼티는 이 함수가 생성자로 사용될 때 이 함수를 통해 생성된 객체의 부모 역할을 하는 프로토타입 객체를 가리킨다.

prototype 프로퍼티는 함수가 생성될 때 만들어지며, constructor 프로퍼티 하나만 있는 객체를 가리킨다. 그리고 constructor 프로퍼티는 자신과 연결된 함수를 가리킨다. 즉, 자바스크립트에서는 함수를 생성할 때, 함수 자신과 연결된 프로토타입 객체를 동시에 생성하며, 이 둘은 prototype과 constructor라는 프로퍼티로 서로를 참조하게 된다.

> 함수의 prototype 프로퍼티가 가리키는 프로토타입 객체는 일반적으로 따로 네이밍하지 않고, 자신과 연결된 함수의 prototype 프로퍼티 값을 그대로 이용한다. 가령 add() 함수의 프로토타입 객체는 add.prototype이 된다.

```javascript
function myFunction() {
    return true;
}

console.dir(myFunction.prototype); // myFunction 함수의 프로토타입 객체
console.dir(myFunction.prototype.constructor); // myFunction 함수
```

지금까지의 내용을 도식화하면 다음과 같다.

## 함수의 다양한 형태

**1. 콜백 함수**

자바스크립트 함수 표현식에서 함수 이름은 꼭 붙이지 않아도 되는 선택 사항이다. 이러한 익명 함수의 대표적인 용도가 바로 콜백 함수이다. 대표적인 콜백 함수의 사용의 예가 자바스크립트에서의 이벤트 핸들러 처리이다.

```html
<!DOCTYPE html>
<html>
<body>
    <script>
        // 페이지 로드 시 호출될 콜백 함수
        window.onload = function() {
            alert('This is the callback function.');
        };
    </script>
</body>
</html>
```

**2. 즉시 실행 함수**

함수를 정의함과 동시에 바로 실행하는 함수를 즉시 실행 함수(immediate functions)라고 한다. 최초 한 번의 실행만을 필요로 하는 초기화 코드 부분 등에 사용할 수 있다.

```javascript
(function (name) {
    console.log('This is the immediate function --> ' + name);
    // This is the immediate function --> foo
})('foo');
```

jQuery와 같은 자바스크립트 라이브러리에 종종 익명함수가 사용되는 것을 볼 수 있다.

```javascript
(function( window, undefined) {
    // ...
})( window );
```

기본적으로 자바스크립트는 변수를 선언할 경우 프로그램 전체에서 접근할 수 있는 전역 유효 범위를 가지게 된다. 그러나 함수 내부에서 정의된 매개변수와 변수(var로 선언된)들은 함수 코드 내부에서만 유효할 뿐 함수 밖에서는 유효하지 않다.

따라서 라이브러리 코드를 이렇게 즉시 실행 함수 내부에 정의해두게 되면, 라이브러리 내의 변수들은 함수 외부에서 접근할 수 없다. **전역 네임스페이스를 더럽히지 않으므로, 다른 자바스크립트 라이브러리들이 동시에 로드가 되더라도 라이브러리 간 변수 이름 충돌 같은 문제를 방지할 수 있다.**

**3. 내부 함수**

자바스크립트에서는 함수 코드 내부에서도 다시 함수 정의가 가능하다. 이렇게 함수 내부에 정의된 함수를 내부 함수(inner function) 라고 부른다. 내부 함수는 자바스크립트의 기능을 보다 강력하게 해주는 클로저를 생성하거나 부모 함수 코드에서 외부에서의 접근을 막고 독립적인 헬퍼 함수를 구현하는 용도 등으로 사용한다.

```javascript
// parent 함수 정의
function parent() {
    var a = 100;
    var b = 200;

    // child 내부 함수 정의
    function child() {
        var b = 300;

        console.log(a);
        console.log(b);
    }

    child();
}

parent();   // 100 300
child();    // Uncaught ReferenceError: child is not defined
```

이것이 가능한 이유는 자바스크립트의 **스코프 체이닝** 때문이다.

부모 함수에서 내부 함수를 외부로 리턴하면, 부모 함수 밖에서도 내부 함수를 호출하는 것이 가능하다.

```javascript
function parent() {
    var a = 100;

    // child 내부 함수
    var child = function() {
        console.log(a);
    }

    // child 함수 반환
    return child;
}

var inner = parent();
inner(); // 100
```

이와 같이 실행이 끝난 parent()와 같은 부모 함수 스코프의 변수를 참조하는 inner()와 같은 함수를 **클로저** 라고 한다.

**4. 함수를 리턴하는 함수**

자바스크립트에서는 함수도 일급 객체이므로 일반 값처럼 함수 자체를 리턴할 수도 있다.

```javascript
var self = function() {
    console.log('a');

    return function() {
        console.log('b');
    }
}

self = self(); // a
self(); // b
```

## 함수 호출과 this

**arguments 객체**

C와 같은 엄격한 언어와 달리, 자바스크립트에서는 함수를 호출할 때 함수 형식에 맞춰 인자를 넘기지 않더라도 에러가 발생하지 않는다. 정의된 함수의 인자보다 적게 함수를 호출했을 경우, 넘겨지지 않은 인자에는 **undefined** 값이 할당된다. 이와 반대로 정의된 인자 개수보다 많게 호출했을 경우 초과된 인수는 무시된다.

이러한 특성 때문에, 런타임 시에 호출된 인자의 개수를 확인하고 이에 따라 동작을 다르게 해줘야 할 경우가 있다. 이를 가능케 하는게 함수를 호출할 때 암묵적으로 전달되는 arguments 객체다. 이 객체는 실제 배열이 아닌 유사 배열 객체이다.

```javascript
function add(a, b) {
    console.dir(arguments);
    return a + b;
}

console.log(add(1)); // NaN
console.log(add(1, 2)); // 3
console.log(add(1, 2, 3)); // 3
```

**호출 패턴과 this 바인딩**

자바스크립트에서 함수를 호출할 때 기존 매개변수로 전달되는 인자값에 더해, 앞서 설명한 arguments 객체 및 **this 인자가 함수 내부로 암묵적으로 전달된다.** this가 이해하기가 어려운 이유는 자바스크립트의 여러 가지 함수가 호출되는 방식(호출 패턴)에 따라 this가 다른 객체를 참조하기(this 바인딩) 때문이다.

**1. 객체의 메서드 호출할 때 this 바인딩**

객체의 프로퍼티가 함수일 경우, 이 함수를 메서드라고 부른다. 메서드 내부 코드에서 사용된 this는 **해당 메서드를 호출한 객체로 바인딩 된다.**

```javascript
var myObject = {
    name: 'foo',
    sayName: function () {
        console.log(this.name);
    }
}

var otherObject = {
    name: 'bar'
}

otherObject.sayName = myObject.sayName;

myObject.sayName(); // foo
otherObject.sayName(); // bar
```

**2. 함수를 호출할 때 this 바인딩**

함수를 호출할 경우는, 해당 함수 내부 코드에서 사용된 **this는 전역 객체에 바인딩된다.** 브라우저에서 자바스크립트를 실행하는 경우 전역 객체는 window 객체다.

```javascript
var test = 'This is test';
console.log(window.test); // 'This is test'

var sayFoo = function() {
    console.log(this.test); // 'This is test'
}

sayFoo();
```

이러한 함수 호출에서의 this 바인딩 특성은 **내부 함수(inner function)를 호출했을 경우에도 동일하게 적용되므로** 유의해서 사용해야 한다.

```javascript
// 전역 변수
var value = 100;

var myObject = {
    value : 1,
    func1 : function() {
        this.value += 1;
        console.log('func1() called. this.value : ' + this.value); // 2

        // func1의 내부 함수 func2
        func2 = function() {
            this.value += 1;
            console.log('func2() called. this.value : ' + this.value); // 101

            // func2의 내부 함수 func3
            func3 = function() {
                this.value += 1;
                console.log('func3() called. this.value : ' + this.value); // 102
            }

            func3();
        }

        func2();
    }
};

myObject.func1();
```

자바스크립트에서 **내부 함수 호출 패턴을 정의해 놓지 않았기 때문에,** 함수로 취급되어 함수 호출 패턴 규칙에 따라 내부 함수의 this는 전역 객체에 바인딩된다. 그래서 흔히들 that 변수를 이용하여 this 값을 저장한다.

```javascript
var value = 100;

var myObject = {
    value : 1,
    func1 : function() {
        // 현 상태의 this를 that 변수에 저장
        var that = this;

        this.value += 1;
        console.log('func1() called. this.value : ' + this.value); // 2

        // func1의 내부 함수 func2
        func2 = function() {
            that.value += 1;
            console.log('func2() called. this.value : ' + that.value); // 3

            // func2의 내부 함수 func3
            func3 = function() {
                that.value += 1;
                console.log('func3() called. this.value : ' + that.value); // 4
            }

            func3();
        }

        func2();
    }
};

myObject.func1();
```

이와 같은 this 바인딩의 한계를 극복하려고, this 바인딩을 명시적으로 할 수 있도록 call과 apply 메서드를 제공한다.

**3. 생성자 함수를 호출할 때 this 바인딩**

자바스크립트에서 **기존 함수에 new 연산자를 붙여서 호출하면 해당 함수는 생성자 함수로 동작한다.** 일반 함수에 new를 붙여 호출하면 원치 않는 생성자 함수로 동작할 수 있기 때문에, 대부분의 자바스크립트 스타일 가이드에서는 특정 함수가 생성자 함수로 정의되어 있음을 알리려고 함수 이름의 첫 문자를 대문자로 쓰기를 권하고 있다.

생성자 함수에서의 this는 **생성자 함수를 통해 새로 생성되어 반환되는 객체에 바인딩된다.** (이는 생성자 함수에서 명시적으로 다른 객체를 반환하지 않는 일반적인 경우에 적용됨)

```javascript
// Person 생성자 함수
var Person = function(name) {
    this.name = name;
}

// foo 객체 생성
var foo = new Person('foo');
console.log(foo.name); // foo
```

- 객체 리터럴 방식과 생성자 함수를 통한 객체 생성 방식의 차이

객체 리터럴 방식으로 생성된 객체는 생성자 함수처럼 다른 인자를 넣어 형식은 같지만 다른 값을 가지는 객체를 생성할 수 없다.

```javascript
var foo = {
    name : 'foo'
}

function Person(name) {
    this.name = name;
}

var bar = new Person('bar');
var baz = new Person('baz');
```

또 다른 차이점은 객체 리터럴 방식으로 생성한 foo 객체의 \_\_proto\_\_ 프로퍼티는 Object.prototype를 가르키며, 생성자 함수를 통해 생성한 bar, baz 객체의 \_\_proto\_\_ 프로퍼티는 Person.prototype를 가르킨다는 점이다.

- 생성자 함수를 new를 붙이지 않고 호출할 경우

생성자 함수를 new를 붙이지 않고 호출할 경우 위에서 살펴본 *2. 함수를 호출할 때 this 바인딩* 규칙이 적용되어 this가 전역객체에 바인딩된다. 이런 위험성을 피하려고 널리 사용되는 패턴이 있다.

```javascript
function A(arg) {
    if (!(this instanceof A)) {
        return new A(arg);
    }

    this.value = arg ? arg : 0;
}
```

**4. call과 apply 메서드를 이용한 명시적인 this 바인딩**

Function.prototype 객체의 메서드인 call()과 apply()를 통해 명시적으로 this를 바인딩 가능하다.

```javascript
// Person 생성자 함수
function Person(name) {
    this.name = name;
}

// foo 빈 객체 생성
var foo = {};

Person.apply(foo, ['foo']);

console.log(foo.name); // foo
console.log(foo.__proto__ === Object.prototype) // true
```

예제에서 알 수 있듯이 apply()를 통해 호출한 경우, 생성자 함수 방식이 아닌 this 가 foo 객체에 바인딩되어 \_\_proto\_\_ 프로퍼티가 Person.prototype이 아닌 Object.prototype이다.

이처럼 apply()나 call() 메서드는 this를 원하는 값으로 명시적으로 매핑해서 특정 함수나 메서드를 호출할 수 있다는 장점이 있다. 그리고 이들의 대표적인 용도가 arguments 객체와 같은 유사 배열 객체에서 배열 메서드를 사용하는 경우이다. arguments 객체는 실제 배열이 아니므로 pop(), shift() 같은 표준 메서드를 사용할 수 없지만 apply() 메서드를 이용하면 가능하다.

```javascript
function myFunction() {
    // arguments.shift(); 에러 발생

    var args = Array.prototype.slice.apply(arguments);
}

myFunction();
```

