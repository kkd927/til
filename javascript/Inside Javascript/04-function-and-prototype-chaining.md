# 04. 함수와 프로토타입 체이닝

## 함수 정의

자바스크립트에서 함수를 생성하는 방법은 3가지가 있다. 각각의 방식에 따라 함수 동작이 미묘하게 차이가 난다.

- 함수 선언문(function statement)
- 함수 표현식(function expression)
- Function () 생성자 함수

자바스크립트에서는 함수 리터럴을 이용해 함수를 생성할 수 있다. 함수 선언문이나 함수 표현식 방법 모두 이런 함수 리터럴 방식으로 함수를 생성한다.

```javascript
// 함수 리터럴을 통한 add() 함수 정의
function add(x, y) {
    return x + y;
}
```

__함수 선언문__ 방식은 함수 리터럴 형태와 같다. 하지만 반드시 함수명이 정의되어 있어야 한다.

```javascript
// add() 함수 선언문
function add(x, y) {
    return x + y;
}
```

__함수 표현식__ 은 함수 리터럴로 하나의 함수를 만들고, 여기서 생성된 함수를 변수에 할당하여 함수를 생성하는 방식이다.

```javascript
// add() 함수 표현식
var add = function (x, y) {
    return x + y;
}

var plus = add;

console.log(add(3, 4)); // 7
console.log(plus(3, 4)); // 7
```

add 변수는 함수 리터럴로 생성한 함수를 참조하는 변수이지, 함수 선언문 처럼 함수 이름이 아니다. 함수 표현식 방법에서는 함수 이름이 선택 사항이며, 보통 사용하지 않는다.

위의 예제처럼 익명 함수를 이용한 함수 표현식 방법을 __익명 함수 표현식__ 이라고 부른다. 함수 이름이 포함된 함수 표현식을 __기명 함수 표현식__ 이라 한다.

```javascript
var add = function sum(x ,y) {
    return x + y;
};

console.log(add(3, 4)); // 7
console.log(sum(3, 4)); // Uncaught ReferenceError: sum is not defined 에러 발생
```

함수 표현식에 사용된 함수 이름은 함수 내부에서 해당 함수를 재귀적으로 호출하거나, 디버거 등에서 함수를 구분할 때 사용된다. 함수 외부에서 해당 함수를 호출할 때는 에러가 발생한다.

함수 선언문 형식으로 정의된 함수는 자바스크립트 엔진에 의해 함수 표현식 형태로 변경되기 때문에 외부에서 호출이 가능하다.

```javascript
// 함수 선언문으로 정의된 함수
function add(x, y) {
    return x + y;
}


// 자바스크립트 엔진에 의해 표현식 형태로 변형된 결과
var add = function add(x, y) {
    return x + y;
};
```

> 일반적으로 자바스크립트 코드를 작성할 때 함수 선언문 방식으로 선언된 함수의 경우는 함수 끝에 세미콜론(;)을 붙이지 않지만, 함수 표현식 방식의 경우 세미콜론(;)을 붙이는 것을 권장한다.

자바스크립트 함수도 __Function()__ 이라는 기본 내장 생성자 함수로부터 생성된 객체라고 볼 수 있다. 함수 선언문이나 함수 표현식 방식도 내부적으로는 Function() 생성자 함수로 함수가 생성된다고 볼 수 있다.

```javascript
new Function (arg1, arg2, ... argN, functionBody)

// arg1, arg2, ... argN - 함수의 매개변수
// functionBody - 함수가 호출될 때 실행될 코드를 포함한 문자열
```

```javascript
var add = new Function('x', 'y', 'return x + y');
console.log(add(3, 4)); // 7
```

위에서 살펴본 함수를 생성하는 3가지 방식(함수 선언문, 함수 표현식, 생성자 함수) 사이에는 약간씩 차이가 있다. 그중의 하나가 __함수 호이스팅(Function Hoisting)__ 이다.

자바스크립트 Guru로 알려진 더글라스 크락포드는 함수 생성에 있어서 그의 저서 *더글라스 크락포드의 자바스크립트 핵심가이드* 에서 함수 표현식만을 사용할 것을 권하고 있따. 그 이유 중 하나가 바로 함수 호이스팅 때문이다.

```javascript
add(2, 3); //5

// 함수 선언문 형태로 add() 함수 정의
function add(x, y) {
    return x + y;
}

add(3, 4); //7
```

위의 예제처럼 함수가 자신이 위치한 코드에 상관없이 __함수 선언문 형태로 정의한 함수의 유효 범위는 코드의 맨 처음부터 시작한다.__ 이것을 __함수 호이스팅__ 이라고 부른다.

더글라스 크락포드는 이러한 함수 호이스팅은 코드의 구조를 엉성하게 만들 수도 있다고 지적하며, 함수 표현식 사용을 권장하고 있다.

```javascript
add(2, 3); // uncaught type error

// 함수 표현식 형태로 add() 함수 정의
var add = function(x, y) {
    return x + y;
}

add(3, 4); // 7
```

위의 예제는 함수 표현식 형태로 정의되어 있어 __호이스팅이 일어나지 않는다.__ 이러한 함수 호이스팅이 발생하는 원인은 자바스크립트의 __변수 생성(Instantiation)__ 과 __초기화(Initialization)__ 의 작업이 분리돼서 진행되기 때문이다. (chap5에서 다룸)

## 함수 객체 : 함수도 객체다

자바스크립트에서 함수도 객체이므로 일반 객체처럼 취급될 수 있다. 또한 함수는 다음과 같은 동작이 가능하다.

- 리터럴에 의해 생성
- 변수나 배열의 요소, 객체의 프로퍼티 등에 할당 가능
- 함수의 인자로 전달 가능
- 함수의 리턴값으로 리턴 가능
- 동적으로 프로퍼티를 생성 및 할당 가능

이와 같은 특징이 있으므로 자바스크립트에서는 함수를 __일급 객체__ 라고 부르며, 이런 특성으로 함수형 프로그래밍이 가능하다.

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

위의 예제는 함수에 동적으로 프로퍼티를 생성할 수 있다는 것을 보여준다. 함수 코드는 함수 객체의 __[[Code]] 내부 프로퍼티__ 에 자동으로 저장된다(ECMAScript 명세).

함수는 일반 객체와는 다르게 추가로 함수 객체만의 표준 프로퍼티가 정의되어 있다. ECMA5 스크립트 명세서에는 모든 함수가 __length__ 와 __prototype 프로퍼티__ 를 가져야 한다고 기술하고 있다. 

그 외에 name, caller, arguments, \_\_proto\_\_ 프로퍼티가 있으나 이들은 ECMA 표준이 아니다. ECMA 표준에서는 arguments 프로퍼티와 같은 이름으로 __arguments 객체__ 를 정의하고 있다.

- name : 함수의 이름
- caller : 자신을 호출한 함수
- arguments : 함수를 호출할 때 전달된 인자값
- \_\_proto\_\_ : 객체 자신이 가리키는 프로토타입

*모든 함수들의 부모 객체는 Function Prototype 객체이다.* 그런데 ECMA 명세서에는 __Function.prototype__ 은 함수라고 정의하고 있다. 대신 예외적으로 Function.prototype 함수 객체의 부모는 자바스크립트의 모든 객체의 조상격인 Object.prototype 객체라고 설명하고 있다. Function.prototype 객체가 가지는 프로퍼티나 메서드는 다음과 같다.

- constructor 프로퍼티
- toString() 메서드
- apply(thisArg, argArray) 메서드
- call(thisArg, [,arg1 [,arg2, ]]) 메서드
- bind(thisArg, [,arg1 [,arg2, ]]) 메서드

함수 객체의 __length 프로퍼티__ 는 앞서 설명했듯이 ECMAScript에서 정한 모든 함수가 가져야 하는 표준 프로퍼티로서, 함수가 정상적으로 실행될 때 기대되는 인자의 개수를 나타낸다.

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

모든 함수는 객체로서 __prototype 프로퍼티__ 를 가진다. 

> 모든 객체에 있는 내부 프로퍼티인 __[[Prototype]]__ 는 객체 입장에서 자신의 부모 역할을 하는 프로토타입 객체를 가리키는 반면에, 함수가 가지는 __prototype 프로퍼티__ 는 이 함수가 생성자로 사용될 때 이 함수를 통해 생성된 객체의 부모 역할을 하는 프로토타입 객체를 가리킨다.

prototype 프로퍼티는 함수가 생성될 때 만들어지며, __constructor 프로퍼티__ 하나만 있는 객체를 가리킨다. 그리고 constructor 프로퍼티는 자신과 연결된 함수를 가리킨다. *즉, 자바스크립트에서는 함수를 생성할 때, 함수 자신과 연결된 프로토타입 객체를 동시에 생성하며, 이 둘은 prototype과 constructor라는 프로퍼티로 서로를 참조하게 된다.*

> 함수의 prototype 프로퍼티가 가리키는 프로토타입 객체는 일반적으로 따로 네이밍하지 않고, 자신과 연결된 함수의 prototype 프로퍼티 값을 그대로 이용한다. 가령 add() 함수의 프로토타입 객체는 add.prototype이 된다.

```javascript
function myFunction() {
    return true;
}

console.dir(myFunction.prototype); // myFunction 함수의 프로토타입 객체
console.dir(myFunction.prototype.constructor); // myFunction 함수
```

## 함수의 다양한 형태

자바스크립트 함수 표현식에서 함수 이름은 꼭 붙이지 않아도 되는 선택 사항이다. 이러한 익명 함수의 대표적인 용도가 바로 **콜백 함수** 이다.

대표적인 콜백 함수의 사용의 예가 자바스크립트에서의 이벤트 핸들러 처리이다.

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

함수를 정의함과 동시에 바로 실행하는 함수를 **즉시 실행 함수(immediate functions)** 라고 한다. 최초 한 번의 실행만을 필요로 하는 초기화 코드 부분 등에 사용할 수 있다.

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

자바스크립트에서는 함수 코드 내부에서도 다시 함수 정의가 가능하다. 이렇게 함수 내부에 정의된 함수를 **내부 함수(inner function)** 라고 부른다. 내부 함수는 자바스크립트의 기능을 보다 강력하게 해주는 클로저를 생성하거나 부모 함수 코드에서 외부에서의 접근을 막고 독립적인 헬퍼 함수를 구현하는 용도 등으로 사용한다.

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
