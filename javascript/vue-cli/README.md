# [vue-cli] Webpack 템플릿으로 vue.js 개발환경 구축하기

## Introduction

이 환경 설정은 대형 프로젝트에 적합하며, Webpack 과 `vue-loader`에 대한 사전지식이 있다고 가정하에 진행됩니다. 시작하기 전에 [vue-loader 문서](https://vue-loader.vuejs.org/kr/start/spec.html)를 먼저 읽어보는 것을 추천합니다.

Vue 프로젝트 템플릿은 빠르게 어플리케이션 코드를 작성할 수 있도록 대부분의 기능을 갖춘 개발 도구 설정을 제공합니다. `vue list`를 실행하여 사용가능한 공식 템플릿을 확인할 수 있습니다. 

현재 사용할 수 있는 템플릿 목록은 다음과 같으며 이 튜토리얼에서는 **webpack 템플릿**을 이용하여 진행합니다. 만약 단순히 vue-loader를 경험하려고 하거나 빠른 프로토타입을 만드려는 상황이라면 [webpack-simple](https://github.com/vuejs-templates/webpack-simple) 템플릿을 사용하길 추천드립니다.

- [webpack](https://github.com/vuejs-templates/webpack) - hot-reload, linting, 테스트 및 CSS 추출 기능을 갖춘 대부분의 기능을 갖추고 있는 Webpack + vue-loader 설정입니다.
- [webpack-simple](https://github.com/vuejs-templates/webpack-simple) - 단순히 Webpack과 vue-loader만 포함합니다. 빠르게 프로토타입을 만들 때 사용합니다.
- [browserify](https://github.com/vuejs-templates/browserify) - hot-reload, linting 및 단위 테스팅 등 대부분의 기능을 갖춘 Browserify + vueify 설정입니다.
- [browserify-simple](https://github.com/vuejs-templates/browserify-simple) 단순히 Browserify와 vueify만 포함합니다. 빠르게 프로토타입을 만들 때 사용합니다.
- [simple](https://github.com/vuejs-templates/simple) - 가장 단순하게 한 HTML 파일에 Vue 설정을 담고 있습니다.

## Quickstart

[vue-cli](https://github.com/vuejs/vue-cli)를 이용하여 스캐폴딩을 진행할 것 입니다. 안정적인 디펜던시를 위해서 npm 3+를 사용하는 것을 추천합니다.

```bash
$ npm install -g vue-cli
$ vue init webpack my-project
$ cd my-project
$ npm install
$ npm run dev
```

쉘에서 `vue init webpack my-project`를 입력하면 아래와 같이 설정 인터페이스 화면이 나옵니다. 본인의 프로젝트에 맞게 설정하시면 됩니다.

```bash
bash$ vue init webpack vue-test

? Project name vue-test
? Project description A Vue.js project
? Author kkd927@gmail.com
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Airbnb
? Set up unit tests Yes
? Pick a test runner karma
? Setup e2e tests with Nightwatch? Yes

   vue-cli · Generated "vue-test".

   To get started:

     cd vue-test
     npm install
     npm run dev

   Documentation can be found at https://vuejs-templates.github.io/webpack
```

`npm run dev` 까지 실행하였다면 기본 준비는 모두 끝납니다. 커맨드 화면에 `Your application is running here: http://localhost:8080`가 출력되는 것을 확인하실 수 있습니다. 브라우저에서 `http://localhost:8080`에 접속하면 자동으로 구성된 화면이 뜨는 것을 볼 수 있습니다. 이제부터 추가설명을 진행할 것입니다.

## 프로젝트 구조

```
.
├── build/                      # webpack config files
│   └── ...
├── config/
│   ├── index.js                # main project config
│   └── ...
├── src/
│   ├── main.js                 # app entry file
│   ├── App.vue                 # main app component
│   ├── components/             # ui components
│   │   └── ...
│   └── assets/                 # module assets (processed by webpack)
│       └── ...
├── static/                     # pure static assets (directly copied)
├── test/
│   └── unit/                   # unit tests
│   │   ├── specs/              # test spec files
│   │   ├── eslintrc            # config file for eslint with extra settings only for unit tests
│   │   ├── index.js            # test build entry file
│   │   ├── jest.conf.js        # Config file when using Jest for unit tests
│   │   └── karma.conf.js       # test runner config file when using Karma for unit tests
│   │   ├── setup.js            # file that runs before Jest runs your unit tests
│   └── e2e/                    # e2e tests
│   │   ├── specs/              # test spec files
│   │   ├── custom-assertions/  # custom assertions for e2e tests
│   │   ├── runner.js           # test runner script
│   │   └── nightwatch.conf.js  # test runner config file
├── .babelrc                    # babel config
├── .editorconfig               # indentation, spaces/tabs and similar settings for your editor
├── .eslintrc.js                # eslint config
├── .eslintignore.js            # eslint ignore rules
├── .gitignore                  # sensible defaults for gitignore
├── .postcssrc.js               # postcss config
├── index.html                  # index.html template
├── package.json                # build scripts and dependencies
└── README.md                   # Default README file
```

#### `buld/`

이 디렉터리는 개발용 서버와 배포용 웹팩 빌드에 대한 설정 파일들이 위치합니다. 웹팩 로더를 커스터마이징하지 않는다면 일반적으로 이 설정파일들 건들 필요가 없습니다. 필요하다면 `build/webpack.base.conf.js` 파일을 확인해보시면 됩니다.

#### `config/index.js`

이 파일은 빌드 단계에 대한 대부분의 설정들을 포함하고 있는 메인 설정 파일입니다.

더 자세한 내용은 [API Proxying During Development](https://vuejs-templates.github.io/webpack/proxy.html)와 [Integrating with Backend Framework](https://vuejs-templates.github.io/webpack/backend.html)를 참고하세요.

#### `src/`

여러분의 어플리케이션 소스 코드가 들어갈 디렉터리입니다. 이 디렉터리 하위의 구조를 어떻게 설계할지는 여러분 마음입니다. Vuex를 사용하려면 [Vuex 어플리케이션을 위한 권고사항](https://vuex.vuejs.org/en/structure.html)을 참고하세요.

#### `static/`

이 디렉터리는 Webpack에 의해 처리될 필요가 없는 정적 파일들을 두는 곳입니다. 이 파일들은 웹펙이 파일들을 빌드할 때 같은 디렉터리로 단순히 복사만 됩니다.

더 자세한 내용은 [Handling Static Assets](https://vuejs-templates.github.io/webpack/static.html)를 참고하세요.

#### `test/unit`

유닛 테스트 파일들이 위치하는 디렉터리입니다. 자세한 내용은 [Unit Testing](https://vuejs-templates.github.io/webpack/unit.html)을 참고하세요.

#### `test/e2e`

e2e 테스트 파일들이 위치하는 디렉터리입니다. 자세한 내용은 [End-to-end Testing](https://vuejs-templates.github.io/webpack/e2e.html)을 참고하세요.

#### `index.html`

SPA(Single Page Application)을 위한 `index.html` 템플릿입니다. 개발과 빌드를 진행할 때, 웹팩이 어플리케이션에 필요한 파일들을 생성하고 해당 파일들에 대한 URL들을 자동으로 이 템플릿 파일에 주입하여 최종 HTML 파일을 생성합니다.

#### `package.json`

빌드 디펜던시와 [빌드 명령어](#빌드-명렁어)들이 포함되는 NPM 패키지 메타 파일입니다.

## 빌드 명령어

빌드 명령어는 [NPM Scripts](https://docs.npmjs.com/misc/scripts)를 통해 실행됩니다.

#### `npm run dev`

Node.js 로컬 개발 서버를 실행시킵니다. 자세한 내용은 [API Proxying During Development](https://vuejs-templates.github.io/webpack/proxy.html)를 참고하세요.

#### `npm run build`

배포를 위한 빌드. 자세한 내용은 [Integrating with Backend Framework](https://vuejs-templates.github.io/webpack/backend.html)를 참고하세요.

- UglifyJS v3를 이용한 Javascript 압축
- html-minifier를 이용한 HTML 압축
- 각 컴포넌트에 존재하는 CSS들을 하나의 파일로 병합 후 cssnano를 이용한 압축
- 효율적인 캐싱을 위해 모든 정적 파일들을 버전 해슁을 통해 컴파일하여 해당 URL들을 index.html에 자동 적용

#### `npm run unit`

유닛 테스트를 실행. 자세한 내용은 [Unit Testing](https://vuejs-templates.github.io/webpack/unit.html)을 참고하세요.

#### `npm run e2e`

Nightwatch를 이용하여 end-to-end 테스트 실행. 자세한 내용은 [End-to-end Testing](https://vuejs-templates.github.io/webpack/e2e.html)을 참고하세요.

#### `npm run lint`

eslint를 실행하여 코드의 린트 에러를 보여줍니다. 자세한 내용은 [Linter Configuration](https://vuejs-templates.github.io/webpack/linter.html)를 참고하세요.

## 정적 자원 처리

프로젝트 구조를 보면 정적 자원들을 위한 `src/assets`와 `static/` 2개의 디렉터리가 잇는 것을 확인할 수 있습니다. 이 둘의 차의점은 웹펙에 의해 처리되느냐 안되느냐의 차이점입니다.

#### 웹팩에 처리되는 자원들

우선 웹팩이 정적 자원들을 어떻게 처리하는지 이해할 필요가 있습니다. `*.vue` 컴포넌트 안에서 모든 html 템플릿과 CSS들은 URL 경로를 찾기 위해 `vue-html-loader`와 `css-loader`에 의해 분석됩니다. 예를들어 `<img src="./logo.png">`, `background: url(./logo.png)`에서 `"./logo.png"`는 상대 경로를 나타내며 모듈 디펜던시로 웹팩에 의해 처리됩니다.

왜냐하면 `logo.png`는 자바스크립트가 아니기 때문에 모듈 디펜던시로 취급되어 `url-loader`와 `file-loader`로 처리할 필요가 있습니다. 이 로더들은 기본적으로 탑재되어 있기 때문에 배포를 위해 상대/모듈 경로를 신경쓸 필요가 없습니다.

#### 실제 정적 자원들

위와는 반대로 `static/`에 위치하는 파일들은 웹팩에 의해 처리되지 않습니다. 이들은 최종 경로에 같은 이름으로 복사됩니다. `config.js`에 있는 `build.assetsPublicPath`와 `build.assetsSubDirectory`에 의해 결정되어지는 절대경로를 이용하여 이 파일들에 접근할 수 있습니다.

아래는 기본 값으로 설정되어 있는 예시입니다.

```javascript
// config/index.js
module.exports = {
  // ...
  build: {
    assetsPublicPath: '/',
    assetsSubDirectory: 'static'
  }
}
```

위의 구성을 따르면 `static/`에 위치하는 파일들은 절대 경로인 `/static/[filename]`으로 접근할 수 있습니다. 만약 `assetsSubDirectory`를 assets으로 바꾼다면 `/assets/[filename]`으로 파일에 접근할 수 있습니다.

자세한 내용은 [Integrating with Backend Framework](https://vuejs-templates.github.io/webpack/backend.html)를 참고하세요.

## 환경 변수

보통 test, development, production 환경에 따라 다른 구성이 필요할 때가 많습니다. 자바 개발자라면 maven profile와 같다고 생각하시면 됩니다.

예를들어 아래와 같이 설정하면

```javascript
// config/prod.env.js
module.exports = {
  NODE_ENV: '"production"',
  DEBUG_MODE: false,
  API_KEY: '"..."' // this is shared between all environments
}

// config/dev.env.js
module.exports = merge(prodEnv, {
  NODE_ENV: '"development"',
  DEBUG_MODE: true // this overrides the DEBUG_MODE value of prod.env
})

// config/test.env.js
module.exports = merge(devEnv, {
  NODE_ENV: '"testing"'
})
```

> 주의: string 변수는 `'"..."'`로 사용해야합니다.

위의 설정은 다음과 같이 해석됩니다.

- Production
  - NODE_ENV = 'production',
  - DEBUG_MODE = false,
  - API_KEY = '...'
- Development
  - NODE_ENV = 'development',
  - DEBUG_MODE = true,
  - API_KEY = '...'
- Testing
  - NODE_ENV = 'testing',
  - DEBUG_MODE = true,
  - API_KEY = '...'

즉, `test.env`는 `dev.env`를 상속받으며, `dev.env`는 `prod.env`를 상속 받습니다.

코드에서 다음과 같이 환경 변수를 사용할 수 있습니다.

```javascript
Vue.config.productionTip = process.env.NODE_ENV === 'production'
```

## Backend 프레임워크와 통합하기

만약 정말 순수한 정적앱(backend API와 독립적인)을 만든다면, `config/index.js`를 수정할 필요가 없습니다. 만약 Rails/Django/Laravel/Spring 같이 자신만의 프로젝트 구조를 갖는 backend 프레임워크와 통합한다면, frontend 파일들을 backend 프로젝트에 넣기 위해서는 `config/index.js`를 수정해야합니다.

아래는 `config/index.js`의 기본 값입니다.

```javascript
// config/index.js
'use strict'
const path = require('path')

module.exports = {
  dev: {
    // Paths
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {},

    // Various Dev Server settings
    host: 'localhost',
    port: 8080, 

    // skipping other options as they are only convenience features
  },
  build: {
    // Template for index.html
    index: path.resolve(__dirname, '../dist/index.html'),

    // Paths
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',

    productionSourceMap: true,

    // skipping the rest ...
  },
}
```

`build` 부분에는 아래와 같은 옵션 값들을 가집니다.

#### `build.index`

> 로컬 파일시스템의 절대경로이어야 합니다.

`index.html` (asset URL이 주입되는) 템플릿이 생성되는 위치를 지정합니다.

만약 backend 프레임워크와 함께 사용중이라면, backend 앱의 view 파일들을 두는 디렉터리 안으로 수정해야 합니다. 예들들어, Rails의 경우 `app/views/layouts/application.html.erb`와 같이 변경해야되고, Laravel의 경우 `resources/views/index.blade.php`입니다.

#### `build.assetsRoot`

> 로컬 파일시스템의 절대경로이어야 합니다.

앱의 모든 정적 자산들을 포함하는 최상위 디렉터리 경로입니다. 예를들어, Rails/Laravel의 경우 `public/`이 되어야 합니다.

#### `build.assetsSubDirectory`

`build.assetsRoot`의 하위에 위치하는 웹팩에 의해 처리되는 자원들의 경로입니다. 예를들어, `build.assetsRoot`를 `/path/to/dist`로 설정하고 `build.assetsSubDirectory`를 `static`으로 설정하면, 웹팩에 의해 처리되는 자원들은 `path/to/dist/static`에 위치하게 됩니다.

이 디렉터리는 항상 빌드 되기전에 내용이 전부 비워지고 빌드에 의해 생성되는 자원들이 새로 들어가게 됩니다.

`static/`에 위치하는 정적 자원들은 빌드 시 이 디렉터리로 복사 되어집니다. 그래서 만약 이 설정을 바꾼다면, `static/` 하위에 있는 정적 자원들에 접근하기 위한 모든 절대 경로들도 수정해줘야 합니다. 자세한 내용은 [정적 자원 처리](#정적-자원-처리)를 참고하세요.

#### `build.assetsPublicPath`

HTTP를 통해 `build.assetsRoot` 자원들에 접근할 수 있는 경로를 설정하는 옵션입니다. 대부분의 경우 `/`를 사용하면 됩니다. 만약 여러분이 사용하고 있는 backend 프레임워크가 정적 자원들의 접근하기 위해 별도의 경로 prefix를 가지는 경우에만 수정하세요.

#### `build.productionSourceMap`

배포용 빌드에 대한 소스 맵을 생성할지 여부입니다.

#### `dev.port`

개발용 서버의 포트를 지정합니다.

#### `dev.proxyTable`

개발용 서버를 위한 proxy 규칙들을 정의합니다. 자세한 내용은 [개발 환경에서의 API Proxy 설정](#개발-환경에서의-API-Proxy-설정)를 참고하세요.

## 개발 환경에서의 API Proxy 설정

## 참고

https://vuejs-templates.github.io/webpack/

https://github.com/vuejs-kr/vue-cli

https://vue-loader.vuejs.org/kr/start/spec.html

