# ECMAScript 6

이 문서는 https://github.com/lukehoban/es6features 를 번역한 내용입니다. 번역 문서를 읽는 중, 오타나 어색한 문장이 있으면 [이슈](https://github.com/kkd927/til/issues/new?title=[ECMAScript%206%20Features])를 등록해주세요!

## Introduction

ECMAScript 2015로도 알려져 있는 ECMAScript 6는 ECMAScript 표준의 가장 최신 버전입니다. ES6는 새로운 언어 기능이 포함된 주요 업데이트이며, 2009년도에 표준화된 ES5 이후로 언어 기능에 대한 첫 업데이트이기도 합니다. 현재 주요 JavaScript 엔진들에서 [ES6 기능들을 구현 중](http://kangax.github.io/compat-table/es6/)에 있습니다.

ECMAScript 6 언어의 전체 스펙을 확인하시려면 [ES6 Standard](http://www.ecma-international.org/ecma-262/6.0/)를 확인하세요.

ES6는 아래의 새로운 기능들을 포함하고 있습니다.

- [arrows](#arrows)
- [classes](#classes)
- [enhanced object literals](#enhanced-object-literals)
- [template strings](#template-strings)
- [destructuring](#destructuring)
- [default + rest + spread](#default--rest--spread)
- [let + const](#let--const)
- [iterators + for..of](#iterators--forof)
- [generators](#generators)
- [unicode](#unicode)
- [modules](#modules)
- [module loaders](#module-loaders)
- [map + set + weakmap + weakset](#map--set--weakmap--weakset)
- [proxies](#proxies)
- [symbols](#symbols)
- [subclassable built-ins](#subclassable-built-ins)
- [promises](#promises)
- [math + number + string + array + object APIs](#math--number--string--array--object-apis)
- [binary and octal literals](#binary-and-octal-literals)
- [reflect api](#reflect-api)
- [tail calls](#tail-calls)

## ECMAScript 6 에 추가된 기능

### Arrows

Arrows(화살표) 함수는 `=>` 문법을 사용하는 축약형 함수입니다. C#, Java 8, CoffeeScript의 해당 기능과 문법적으로 유사합니다. Arrows는 표현식의 결과 값을 반환하는 표현식 본문(expression bodies)뿐만 아니라 상태 블럭 본문(statement block bodies)도 지원합니다. 하지만 일반 함수의 자신을 호출하는 객체를 가리키는 `dynamic this`와 달리 arrows 함수는 코드의 상위 스코프(lexical scope)를 가리키는 `lexical this`를 가집니다.

```javascript
var evens = [2, 4, 6, 8,];

// Expression bodies (표현식의 결과가 반환됨)
var odds = evens.map(v => v + 1);   // [3, 5, 7, 9]
var nums = evens.map((v, i) => v + i);  // [2, 5, 8, 11]
var pairs = evens.map(v => ({even: v, odd: v + 1})); // [{even: 2, odd: 3}, ...]

// Statement bodies (블럭 내부를 실행만 함, 반환을 위해선 return을 명시)
nums.forEach(v => {
  if (v % 5 === 0)
    fives.push(v);
});

// Lexical this
// 출력결과 : Bob knows John, Brian
var bob = {
  _name: "Bob",
  _friends: ["John, Brian"],
  printFriends() {
    this._friends.forEach(f =>
      console.log(this._name + " knows " + f));
  }
}
```

printFriends() 함수의 서브루틴은 다음과 문법상 동일하게 동작합니다.

```javascript
this._friends.forEach(function (f) {
    console.log(this._name + " knows " + f));
}.bind(this));
```

화살표 함수의 더 자세한 설명은 [MDN Arrow Functions](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/%EC%95%A0%EB%A1%9C%EC%9A%B0_%ED%8E%91%EC%85%98)를 참고하세요. printFriends() 함수의 선언 표기법이 궁금하시면 [MDN Object Initializer](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Object_initializer)를 참고하세요.

### Classes

ES6 클래스는 포로토타입 기반 객체지향 패턴을 더 쉽게 사용할 수 있는 대체재입니다. 클래스 패턴 생성을 더 쉽고 단순하게 생성할 수 있어서 사용하기도 편하고 상호운용성도 증가됩니다.

```javascript
class SkinnedMesh extends THREE.Mesh {
  constructor(geometry, materials) {
    super(geometry, materials);

    this.idMatrix = SkinnedMesh.defaultMatrix();
    this.bones = [];
    this.boneMatrices = [];
    //...
  }
  update(camera) {
    //...
    super.update();
  }
  get boneCount() {
    return this.bones.length;
  }
  set matrixType(matrixType) {
    this.idMatrix = SkinnedMesh[matrixType]();
  }
  static defaultMatrix() {
    return new THREE.Matrix4();
  }
}
```

더 자세한 설명은 [MDN Classes](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/class)를 참고하세요.

### Enhanced Object Literals

ES6에서 객체 리터럴은 선언문에서 프로토타입 설정, `foo: foo` 선언을 위한 단축 표기법, 메서드 정의, super 클래스 호출 및 동적 속성명을 지원하도록 향상 되었습니다. 그에 따라 객체 리터럴 및 클래스 선언이 더 밀접되어져, 객체기반 설계가 더 편리해졌습니다.

```javascript
var obj = {
    // __proto__
    __proto__: theProtoObj,

    // ‘handler: handler’의 단축 표기
    handler,

    // Methods
    toString() {
     // Super calls
     return "d " + super.toString();
    },

    // Computed (dynamic) property names
    [ 'prop_' + (() => 42)() ]: 42
};
```

더 자세한 설명은 [MDN Grammar and types: Object literals](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Values,_variables,_and_literals#객체_리터럴)를 참고하세요.

### Template Strings

Template Strings(ES6 부터는 Template literals라 부름)는 문법적으로 더 편하게 string을 생성할 수 있게 합니다. 이는 Perl, Python 등의 문자열 보간(string interpolation)과 유사합니다. Tagged template literals는 인젝션 공격 방어 혹은 문자열로 부터 상위 데이터 구조체 재조립 등을 위해 string 생성을 커스터마이징이 가능하게끔 해줍니다.

```javascript
// Basic literal string creation
`In JavaScript '\n' is a line-feed.`

// Multiline strings
`In JavaScript this is
 not legal.`

// String interpolation
var name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`

// Construct an HTTP request prefix is used to interpret the replacements and construction
POST`http://foo.org/bar?a=${a}&b=${b}
     Content-Type: application/json
     X-Credentials: ${credentials}
     { "foo": ${foo},
       "bar": ${bar}}`(myOnReadyStateChangeHandler);
```

더 자세한 설명은 [MDN Template literals](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Template_literals)를 참고하세요.

### Destructuring

