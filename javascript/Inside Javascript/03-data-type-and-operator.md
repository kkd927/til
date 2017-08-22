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
