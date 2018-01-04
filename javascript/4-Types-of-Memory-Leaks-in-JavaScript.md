# 자바스크립트에서 메모리 누수의 4가지 형태

> 이 글은 [4 Types of Memory Leaks in JavaScript and How to Get Rid Of Them](https://auth0.com/blog/four-types-of-leaks-in-your-javascript-code-and-how-to-get-rid-of-them/)를 번역한 글입니다. 번역 문서를 읽는 중, 오타나 어색한 문장이 있으면 부담없이 댓글 부탁드립니다.

이번 글에서 클라이언트단 JavaScript 코드의 일반적인 메모리 누수 유형에 대해 살펴보겠습니다. 또한 [크롬 개발 도구(Chrome DevTools)](https://developers.google.com/web/tools/chrome-devtools/?hl=ko)를 사용하여 메모리 누수를 찾는 방법에 대해서도 알아보겠습니다.

## Introduction

메모리 누수(Memory leaks)는 모든 개발자들이 직면하는 문제입니다. 심지어 메모리를 관리해주는 프로그래밍 언어(Java, C# 등)를 사용하는 경우에도 메모리 누수 문제에서 벗어날 수 없습니다. 메모리 누수는 어플리케이션의 속도 저하, 충돌, 지연 시간 증가뿐만 아니라 다른 어플리케이션에게도 악영향을 끼치는 등 전반적인 문제들의 원인이 됩니다.

### 메모리 누수란?

메모리 누수는 어떠한 이유로 애플리케이션에서 더이상 사용되지 않음에도 불구하고, 운영체제나 사용가능한 메모리 풀에 반환되지 않는 메모리라고 정의할 수 있습니다. 프로그래밍 언어들은 각기 다른 방법으로 메모리를 관리합니다. 이런 프로그래밍 언어 차원에서의 메모리 관리는 메모리 누수의 가능성을 많이 줄여줍니다. 하지만 프로그래밍 언어의 메모리 관리 시스템이 특정 메모리가 실제 사용중인지 미사용중인지 완벽히 구분해내는 것은 사실상 불가능에 가깝습니다. 오직 그 코드를 작성한 개발자들만이 해당 메모리 조각을 운영체제로 반환시킬 수 있는지 여부를 명확히 알 수 있습니다. 특정 프로그래밍 언어들은 개발자들이 좀 더 편리하게 메모리 반환을 할 수 있도록 기능들을 제공하기도 합니다. 개발자들이 명시적으로 미사용 메모리를 반환해야하는 언어들도 있습니다. 위키피디아에 [수동](https://en.wikipedia.org/wiki/Manual_memory_management) 과 [자동](https://en.wikipedia.org/wiki/Manual_memory_management) 메모리 관리에 관해 잘 설명해논 글이 있습니다.

### 자바스크립트에서의 메모리 관리

자바스크립트는 **garbage collected** 언어 중 하나입니다. Garbage collected 언어들은 이전에 할당한 메모리를 애플리케이션에서 여전히 사용 중인지를 주기적으로 검사해 개발자들이 메모리를 관리에 덜 신경쓸 수 있도록 도움을 줍니다. 다른 말로, garbage collected 언어들은 메모리 관리 문제를 "어떤 메모리가 여전히 필요한가?" 에서 "어떤 메모리가 애플리케이션의 다른 코드에서 접근할 수 있는가?"로 관점을 축소할 수 있게 해줍니다. 이 둘의 차이점은 미묘해보이지만 매우 중요합니다. 할당된 메모리가 미래에 사용되는 지의 여부는 오직 개발자만이 알지만, 다른 코드에서 더 이상 접근되지 않은 메모리는 알고리즘적으로 결정할 수 있어 OS에 반환될 수 있도록 표시할 수 있습니다.

Garbage collected가 아닌 언어들은 명시적 관리라는 다른 방법으로 메모리를 관리합니다. 개발자가 메모리를 더이상 사용하지 않을 때, 컴파일러가 메모리를 회수할 수 있도록 명시적으로 선언해야합니다. 이러한 기법은 잠재적으로 메모리 누수의 위험을 가질 수 있습니다.

## 자바스크립트에서의 메모리 누수

Garbage collected 언어에서 메모리 누수의 주요 원인은 **예상치 못한 참조** 입니다. 예상치 못한 참조(unwanted references)가 무엇인지 이해하기 위해선, 먼저 garbage collector가 어떤 방식으로 해당 메모리가 다른 코드에서 접근될 수 있는지 여부를 판단하는지를 알 필요가 있습니다.

### Mark-and-sweep

대부분의 garbage collector는 **mark-and-sweep** 라고 알려진 알고리즘을 사용합니다. 이 알고리즘은 다음과 같은 절차를 가집니다.

1. Garbage collector는 "roots"의 목록을 생성합니다. 루트들은 일반적으로 코드에서 참조가 계속 유지되는 전연 변수들입니다. 자바스크립트에서는 "window" 객체가 root가 되는 글로벌 변수의 대표적인 예입니다. window 객체는 항상 유지되기 때문에, garbage collector는 window 객체뿐만 아니라 그 자식 객체들도 항상 유지될 것이라 판단하여 폐기되지 않도록 합니다. 

2. 모든 루트들을 검사해 폐기되지 않도록 활성화 상태임을 표시합니다. 루트의 자식들도 재귀적으로 검사합니다(자식의 자식... 반복). 결국 루트에서 도달될 수 있는 자식 객체들은 폐기되지 않습니다.

3. 할성화 상태로 표시되지 않은 모든 메모리 조각들은 이제 폐기될 수 있는 것으로 판단합니다. 그래서 garbage collector는 이 메모리들을 해제하여 OS에 반환합니다.

최신 garbage collector들은 이 알고리즘을 다른 형태로 더 진화시켰지만, 기본 베이스는 동일합니다. 접근될 수 있는 메모리 조각들은 할성화 상태로 표시하고 그 외는 폐기되도록 고려되어집니다.

예상치 못한 참조는 개발자는 더 이상 사용되지 않을 것이라 생각했지만, 어떠한 이유로 활성화 상태인 루트 트리 안에 존재하는 메모리 조각들입니다. 자바스크립트에서 예상치 못한 참조는 더이상 사용되지 않지만 코드 상 어딘가에 유지되어 해제되지 못한 변수들입니다. 어떤 이들은 이를 개발자의 실수라고 말하기도 합니다.

그래서 자바스크립트에서 발생할 수 있는 일반적인 메모리 누수 형태들을 이해하기 위해서는 흔히 까먹기 쉬운 참조들을 먼저 알 필요가 있습니다.

## 자바스크립트 메모리 누수의 일반적인 3가지 형태

### 1. 우발적으로 생성된 전역 변수

자바스크립트 언어의 목표 중 하나는 Java와 유사하지만 초보자들도 쉽게 사용할 수 있는 언어를 만드는 것이었습니다. 그 방법 중 하나가 자바스크립트가 선언되지 않은 변수들을 처리할 수 있도록 하는 것이었습니다. 선언되지 않은 변수는 **global** 객체 내부에 새로운 변수로 생성됩니다. 브라우저 환경에서 global 객체는 `window` 입니다.

아래의 코드는

```javascript
function foo(arg) {
    bar = "this is a hidden global variable";
}
```

실제 다음과 같이 동작됩니다.

```javascript
function foo(arg) {
    window.bar = "this is an explicit global variable";
}
```

만약 `bar` 변수가 `foo` 함수 범위 내에서만 참조가 유지되도록 하려고 했는데 실수로 `var` 로 선언 하는 것을 깜빡했다면, 예상지 못한 전역 객체가 생성되어진 것입니다. 위 예제에서 이 간단한 문장이 큰 악영향을 끼치지 않을 수 도 있지만, 좋지 않은 것은 분명합니다.

또 다른 우발적인 전역 객체는 `this` 를 통해 생성될 수 있습니다.

```javascript
function foo() {
    this.variable = "potential accidental global";
}

// foo 함수를 호출하면, this는 window 전역 객체를 가리키게 됩니다.
foo();
```

자바스크립트 파일의 시작 부분에 `'use strict';`를 추가하여 이러한 실수를 방지할 수 있습니다. 이는 자바스크립트 엔진이 우발적인 전역 객체 생성을 방지하도록 더 엄격한 모드로 자바스크립트를 파싱하게 해줍니다.

예상치 못한 전역 변수에 대해 이야기했지만, 코드 여기저기서 필요에 의해 명시적으로 전역 변수를 선언하여 사용하는 곳이 많습니다. 이들은 null로 처리하거나 재할당하지 않는 한 garbage collector에 수집되지 않습니다. 특히, 대용량 데이터를 일시적으로 저장하고 처리하기 위해 사용된 전역 변수는 더 신중하게 다뤄야합니다. 전역 변수의 사용이 끝났다면, null로 처리하거나 재할당을 반드시 해야합니다. 전역 변수와 관련하여 메모리 사용 증가를 야기하는 일반적인 원인 중 하나는 캐시입니다. 캐시는 자주 사용되는 데이터들을 저장합니다. 이를 효줄적으로 처리하기 위해 데이터가 커진다면 캐시 사이즈도 커지게 됩니다. 캐시는 수집되지 않기 때문에 캐시 사이즈가 점점 커진다면 방대한 메모리 사용을 야기시킬 수 있습니다.

### 2. 잊혀진 타이머와 콜백

자바스크립트에서 `setInterval` 는 매우 흔하게 사용됩니다. 많은 라이브러리에서 observer를 제공하거나, callback을 가지는 기능들을 가지고 있습니다. 이러한 라이브러리의 대부분은 더 이상 사용이 안되면 자체적으로 callback에 대한 참조를 해제하도록 구현되어 있습니다. 하지만 setInterval의 경우 아래와 같은 코드 형태를 많이 사용합니다.

```javascript
var someResource = getData();
setInterval(function() {
    var node = document.getElementById('Node');
    if(node) {
        // Do stuff with node and someResource.
        node.innerHTML = JSON.stringify(someResource));
    }
}, 1000);
```

위 예제를 통해 타이머 내부에서 참조된 노드 혹은 데이터가 더 이상 사용되지 않을 때 발생할 수 있는 문제를 살펴보겠습니다. `node` 는 미래에 제거되어질지도 모르는 객체를 나타냅니다. 만약 객체가 제거되어지면 interval 내부의 핸들러는 더 이상 필요가 없게 되지만, 여전히 계속 동작되어 collector에 의해 수집되지 않게 됩니다. 만약 interval 핸들러가 수집되지 않는다면, 이 핸들러에 의존되는 객체들도 수집되지 않게 됩니다. 이 말은 대량의 데이터를 저장하고 있을 수도 있는 `someResource` 도 수집되지 않게됨을 의미합니다.

이러한 observer 형태의 경우, 더 이상 사용되지 않을 때 명시적으로 제거하는 것이 중요합니다. 이는 과거의 경우, 특정 브라우저(Internet Explorer 6 같은)들이 순환 참조를 잘 관리하지 못해 매우 중요한 요소 중 하나였습니다. 현재 대부분의 브라우저들은 observer 객체가 더 이상 사용되지 않으면 명시적으로 제거하지 않더라도 수집합니다. 하지만 이러한 observer들을 명시적으로 제거하는 것이 좋은 관행으로 남아 있습니다. 예를들어 다음 코드와 같습니다.

```javascript
var element = document.getElementById('button');

function onClick(event) {
    element.innerHtml = 'text';
}

element.addEventListener('click', onClick);
// Do stuff
element.removeEventListener('click', onClick);
element.parentNode.removeChild(element);
// Now when element goes out of scope,
// both element and onClick will be collected even in old browsers that don't
// handle cycles well.
```

Observer와 관련 참조들은 자바스크립트 개발자들에게 골칫거리가 되곤 했습니다. 이는 Internet Explorer의 garbage collector의 버그(혹은 애초에 그렇게 설계된) 때문이었습니다. 구버전 Internet Explorer는 DOM 노드와 자바스크립트 코드 사이의 순환 참조를 탐지하지 못 했습니다. 이로 인해 메모리 누수가 발생하게 되었고 개발자들은 명시적으로 참조를 제거하기 시작했습니다. 현재의 최신 브라우저들은(Internet Explorer와 Microsoft Edge를 포함하여) 이들을 정확하게 탐지하여 수집해갑니다. 즉, 노드가 제거되기 전에 `removeEventListener`를 호출할 필요가 없어졌습니다.

*jQuery* 와 같은 프레임워크과 라이브리러들은 노드를 폐기하기 전에 listener들을 명시적으로 제거합니다. 이는 라이브러리 내부에서 수행되며, 구버전 Internet Explorer와 같이 문제가 생길법한 브라우저에서도 메모리 누수가 발생하지 않도록 구현되어 있습니다.

### 3. DOM 외부에서의 참조

종종 DOM 노드들을 자료구조 안에 저장하는 것이 유용할 때가 있습니다. 테이블에서 여러 행의 내용을 빠르게 업데이트하려는 경우를 가정합시다. 각 행의 DOM 노드들에 대한 참조를 맵이나 배열에 저장하는 것이 좋습니다. 이 경우 DOM 요소에 대한 참조는 DOM 트리와 맵, 2군데에서 유지됩니다. 만약 나중에 이 행들을 제거해야할 경우 두 참조 모두 제거를 해야합니다.

```javascript
var elements = {
    button: document.getElementById('button'),
    image: document.getElementById('image'),
    text: document.getElementById('text')
};

function doStuff() {
    image.src = 'http://some.url/image';
    button.click();
    console.log(text.innerHTML);
    // Much more logic
}

function removeButton() {
    // The button is a direct child of body.
    document.body.removeChild(document.getElementById('button'));

    // 여기서 elements 에서 여전히 button 참조를 가지고 있습니다.
    // 이 경우 button element는 여전히 메모리에 상주하게 되며 GC에 의해 수집될 수 없습니다.
}
```

여기에 추가적으로 고려해야되는 사항은 DOM 트리 안에서 내부 혹은 말단 노드의 참조입니다. 자바스크립트 코드에서 테이블의 특정 셀(<td> 태그)에 대한 참조를 가지고 있다고 가정합시다. 나중에 DOM으로 부터 테이블을 제거하기로 결정했지만 여전히 셀에 대한 참조를 가지고 있게 됩니다. 직관적으로 GC가 해당 셀을 제외한 나머지는 수집할 것이라 생각됩니다. 하지만 현실은 그렇지 않습니다. 셀은 테이블의 자식 노드이고 자식 노드는 부모 노드에 대한 참조를 유지합니다. 그래서 테이블의 셀에 대한 참조로 인해 테이블 전체가 메모리에 유지되게 됩니다. DOM 요소에 대한 참조를 유지할 때는 이 점을 주의해야 합니다.

### 4. 클로저(Closures)

자바스크립트 개발에서 주요 요소 중 하나는 상위 스코프의 변수에 접근가능한 클로저입니다. Meteor 개발자들은 자바스크립트 런타임의 구현 방법으로 인해 메모리 누수가 가능한 [특정한 사례를 발견](https://blog.meteor.com/an-interesting-kind-of-javascript-memory-leak-8b47d2e7f156)했습니다.

```javascript
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var unused = function () {
    if (originalThing)
      console.log("hi");
  };
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log('someMessage');
    }
  };
  // 만약 여기에 `originalThing = null` 를 추가한다면, 메모리 누수는 사라질 것 입니다.
};
setInterval(replaceThing, 1000);
```

위 코드에서 `replaceThing` 이 호출될 때마다 큰 사이즈의 배열과 `someMethod` 클로저를 생성합니다. 동시에 `unused` 변수는 `originalThing` 를 참조하는 클로저를 가지게 됩니다(`originalThing`는 `replaceThing` 선언 위에 있는 `theThing`을 참조). 벌써 헷갈리기 시작하시죠? 중요한 것은 클로저가 생성되면 해당 부모 스코프에 더 상위의 스코프에 대한 참조가 있는 경우 참조고리가 함께 공유됩니다. 즉, 비록 `unused` 가 절대 사용되지 않더라도 `usused` -> `orginThing` -> `theThing` 참조 고리에 의해  이 코드가 반복적으로 실행될 때 마다 메모리 사용량이 꾸준히 증가하는 것을 관찰할 수 있습니다. GC가 실행되더라도 메모리 사용량이 줄어들지 않게 됩니다. 본질적으로 클로저의 참조고리가 생성되고(`theThing` 변수를 루트로), 이 클로저의 범위에는 큰 사이즈의 배열에 대한 간접적인 참조를 동반하기 때문에 상당한 양의 메모리 누수가 발생하게 됩니다.

## Garbage Collectors의 비직관적인 동작

Garbage Collectors는 편리하지만 그 들만의 특정 메커니즘에 의해 동작됩니다. 그 특징 중 하나는 비결정성(nondeterminism)입니다. 이 말은 GC는 예측이 불가능하단 뜻입니다. 언제 수집이 수행되는지 정확하게 예측할 수 없습니다. 즉, 경우에 따라서 프로그램에 요구되는 메모리보다 더 많은 메모리가 사용되고 있을 수 있습니다. 또 다른 경우, 민감함 어플리케이션에서는 짧은 일시정지 현상이 보이기도 합니다. 비록 비결정성은 수집이 언제 수행될지 모른다는 것을 의미하지만, 대부분의 GC는 일반적으로 메모리 할당이 이뤄지는 경우에만 수집을 수행합니다. 만약 메모리 할당이 이뤄지지 않았으면 대부분의 GC는 유휴상태에 있게 됩니다. 아래와 같은 시나리오를 살펴봅시다.

1. 사이즈가 큰 데이터 할당을 여러번 수행합니다.
2. Garbage Collector에 의해 대부분(혹은 전부)은 더 이상 접근되지 않는다라고 표시가 됩니다.(더 이상 사용하지 않은 경우 null로 초기화 했다고 가정)
3. 더 이상의 할당을 수행하지 않습니다.

이 시나리오에서 대부분의 GC들은 더 이상 수집을 수행하지 않습니다. 즉, 더 이상 접근되지 않는 데이터 셋들이 남아있음에도 불구하고 수집이 일어나지 않습니다. 이는 엄격히 메모리 누수는 아니지만, 일반적인 메모리 사용량보다 더 많은 메모리를 사용하게 됩니다.

구글은 이 동작에 대한 훌륭한 예제를 [JavaScript Memory Profiling docs, example #2](https://developer.chrome.com/devtools/docs/demos/memory/example2)에서 제공합니다.

## Chrome Memory Profiling Tools Overview

크롬은 자바스크립트 코드의 메모리 사용을 프로파일링할 수 있는 좋은 도구들을 제공합니다. 메모리에 관련 도구로 Performance 메뉴와 Memory메뉴가 있습니다.

### Performance 메뉴 (구 Timeline)

Performance 메뉴는 코드에서 비정상 메모리 사용 패턴을 발견하는데 필수적입니다. 이 스크린샷에서 메모리 누수가 계속 커지는 것을 볼 수 있습니다. Major GC가 수행 후에도 메모리 사용량이 줄어들지 않습니다. 노드의 수도 점차 증가하고 있습니다. 이것들으 종합해보면 코드 어딘가에 DOM 노드의 누수임을 짐작할 수 있습니다.

### Memory 메뉴 (구 Profiles)

앞으로 자주 봐야할 메뉴입니다. Memory 메뉴에서 스냅샷들을 찍을 수 있고 자바스크립트 코드의 메모리 사용을 비교해볼 수 있습니다. 또한 시간에 따라 메모리 할당을 기록할 수 있습니다. summary 와 comparison 목록을 주로 살펴보시면 됩니다.

## 크롬 개발자 도구를 이용한 메모리 누수 찾기 예제

### Find out if memory is periodically increasing

### Get two snapshots

### Recording heap allocations to find leaks


### Another useful feature
