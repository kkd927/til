## 03. 자바스크립트 데이터 타입과 연산자

자바스크립트의 데이터 타입

- 기본타입 : 숫자(Number), 문자열(String), 불린값(Boolean), undefined, null
- 참조타입 : 객체(Object) - 배열(Array), 함수(Function), 정규표현식

자바스크립트에서는 모든 숫자를 64비트 부동 소수점 형태로 저장한다.

```javascript
var intNum = 10;
var floatNum = 0.1;

console.log(typeof intNum); // number
console.log(typeof floatNum); // number
```

문자열은 문자 배열처럼 인덱스를 이용해서 접근할 수 있다. 한 번 생성된 문자열은 읽기만 가능하지 수정은 불가능하다.

```javascript
// 인덱스를 이용하여 접근 가능
var str = 'test';
console.log(srt[0], str[1], str[2], str[3]); // 출력값 test

// 변경 불가능
str[0] = 'T';
console.log(str); // 출력값 test
```
자바스크립트에서 ```undefined```는 타입이자, 값을 나타낸다.

자바스크립트에서 객체는 단순히 '이름(key):값(value)' 형태의 프로퍼리들을 저장하는 컨테이너이다.

자바스크립트에서 객체를 생성하는 방법을 크게 세 가지가 있다.

- 기본 제공 Object() 객체 생성자 함수를 이용하는 방법
- 객체 리터럴을 이용하는 방법
- 생성자 함수를 이용하는 방법

객체 리터럴이란 객체를 생성하는 표기법을 의미한다. 객체 리터럴은 중괄호({})를 이용해서 객체를 생성한다.
프로퍼티값으로는 자바스크립트의 값을 나타내는 어떤 표현식도 올 수 있으며, 이 값이 함수인 경우 이러한 프로퍼티를 메서드라고 부른다.

```javascript
// 객체 리터럴 방식으로 foo 객체를 생성
var foo = {
    name : 'foo',
    age : 30,
    gender : 'male'
};

console.log(typeof foo); // object
console.log(foo); // { name: 'foo', age: '30', gender: 'male' }
```
객체 프로퍼티는 대괄호([]) 표기법, 마침표(.) 표기법을 통해 접근할 수 있다.

```javascript
console.log(foo.name); // foo
console.log(foo['name']); // foo
```

대괄호 표기법에서 접근하려는 프로퍼티 이름을 문자열 형태로 만들지 않으면 모든 자바스크립트 객체에서 호출 가능한 toString()이라는 메서드를 자동으로 호출해서 이를 문자열로 바꾸려는 시도를 한다.

접근하려는 프로퍼티가 표현식이거나 예약어일 경우 대괄호 표기법만을 이용해서 접근해야한다.

```javascript
console.log(foo['full-name']); // O
console.log(foo.full-name);    // X
```

```for in``` 문을 사용하면, 객체에 포함된 모든 프로퍼티에 대해 루프를 수행할 수 있다.

```javascript
var foo = {
    name : 'foo',
    age : 30,
    gender : 'male'
};

// for in 문을 이용한 객체 프로퍼티 출력
var prop;
for (prop in foo) {
	console.log(prop, foo[prop]);
}

// 결과
// name foo
// age 30
// gender male
```

객체의 프로퍼티를 ```delete``` 연산자를 이용해 즉시 삭제할 수 있다. 하지만 객체 자체를 삭제하지는 못한다.

```javascript
console.log(foo.name); // foo
delete foo.name;       // name 프로퍼티 삭제
console.log(foo.name); // undefined

console.log(foo.age);  // 30
delete foo;            // foo 객체 삭제 시도 (삭제 안됨)
console.log(foo.age);  // 30
```

자바스크립트에서는 기본 타입인 숫자, 문자열, 불린값, null, undefined 5가지를 제외한 모든 값은 객체다. 배열이나 함수 또한 객체로 취급한다. 이러한 객체의 모든 연산이 실제 값이 아닌 참조 값으로 처리되기 때문에 참조 타입이라 부른다.

동등 연산자(==)를 사용하여 두 객체를 비교할 때도 객체의 프로퍼티값이 아닌 참조값을 비교한다.

```javascript
var a = 100;
var b = 100;

var objA = { value : 100 };
var objB = { value : 100 };
var objC = objB;

console.log(a == b); // true
console.log(objA == objB); // false
console.log(objA == objC); // true
```

함수 호출 방식의 경우 기본 타입의 경우 값에 의한 호출(Call By Value) 방식으로 동작한다. 참조 타입의 경우 참조에 의한 호출 (Call By Reference) 방식으로 동작한다.

자바스크립트의 __모든 객체는 자신의 부모 역할을 하는 객체와 연결__ 되어 있다. 객체지향의 상속 개념과 같이 부모 객체의 프로퍼티를 마치 자신의 것처럼 쓸 수 있는 것이 특징이 있다. 이러한 부모 객체를 __프로토타입 객체__(짧게는 프로토타입)라고 부른다.

```javascript
var foo = {
	name: 'foo',
    age: 30
};

console.log(foo.toString());
console.dir(foo);
```

객채 리터럴로 생성한 name과 age 프로퍼티 이외에도 foo 객체에 __\_\_proto\_\_ 프로퍼티__ 가 있다. 이 프로퍼티가 객체의 부모인 프로토타입 객체를 가리킨다.

ECMAScript 명세서에는 자바스크립트의 __모든 객체는 자신의 프로토타입을 가리키는 [[Prototype]]라는 숨겨진 프로퍼티를 가진다__ 고 설명하고 있다. 크롬 브라우저에서는 __\_\_proto\_\_ 프로퍼티__ 형태로 구현돼 있다.

모든 객체의 프로토타입은 자바스크립트의 룰에 따라 객체를 생성할 때 결정된다. 객체 리터럴 방식으로 생성된 객체의 경우 Object.prototype 객체가 프로토타입 객체가 된다.

객체를 생성할 때 결정된 프로토타입 객체는 임의의 다른 객체로 변경하는 것도 가능하다. 즉, 부모 객체를 동적으로 바꿀 수 있다. 자바스크립트에서는 이러한 특징을 활용해서 객체 상속 등의 기능을 구현한다.
