### Introduction

이 튜토리얼은 대형 프로젝트에 적합하며, Webpack 과 `vue-loader`에 대한 사전지식이 있다고 가정하에 진행됩니다. 시작하기 전에 [vue-loader 문서](https://vue-loader.vuejs.org/kr/start/spec.html)를 먼저 읽어보는 것을 추천합니다.

Vue 프로젝트 템플릿은 빠르게 어플리케이션 코드를 작성할 수 있도록 대부분의 기능을 갖춘 개발 도구 설정을 제공합니다. `vue list`를 실행하여 사용가능한 공식 템플릿을 확인할 수 있습니다. 

현재 사용할 수 있는 템플릿 목록은 다음과 같으며 이 튜토리얼에서는 webpack 템플릿을 이용하여 진행합니다. 만약 단순히 vue-loader를 경험하려고 하거나 빠른 프로토타입을 만드려는 상황이라면 [webpack-simple](https://github.com/vuejs-templates/webpack-simple) 템플릿을 사용하길 추천드립니다.

- [webpack](https://github.com/vuejs-templates/webpack) - hot-reload, linting, 테스트 및 CSS 추출 기능을 갖춘 대부분의 기능을 갖추고 있는 Webpack + vue-loader 설정입니다.

- [webpack-simple](https://github.com/vuejs-templates/webpack-simple) - 단순히 Webpack과 vue-loader만 포함합니다. 빠르게 프로토타입을 만들 때 사용합니다.

- [browserify](https://github.com/vuejs-templates/browserify) - hot-reload, linting 및 단위 테스팅 등 대부분의 기능을 갖춘 Browserify + vueify 설정입니다.

- [browserify-simple](https://github.com/vuejs-templates/browserify-simple) 단순히 Browserify와 vueify만 포함합니다. 빠르게 프로토타입을 만들 때 사용합니다.

- [simple](https://github.com/vuejs-templates/simple) - 가장 단순하게 한 HTML 파일에 Vue 설정을 담고 있습니다.

### Quickstart

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

### 프로젝트 구조

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

- `buld/`

이 디렉터리는 개발용 서버와 배포용 웹팩 빌드에 대한 설정 파일들이 위치합니다. 웹팩 로더를 커스터마이징하지 않는다면 일반적으로 이 설정파일들 건들 필요가 없습니다. 필요하다면 `build/webpack.base.conf.js` 파일을 확인해보시면 됩니다.

- `config/index.js`

이 파일은 빌드 단계에 대한 대부분의 설정들을 포함하고 있는 메인 설정 파일입니다.

더 자세한 내용은 [API Proxying During Development](https://vuejs-templates.github.io/webpack/proxy.html)와 [Integrating with Backend Framework](https://vuejs-templates.github.io/webpack/backend.html)를 참고하세요.

- `src/`

여러분의 어플리케이션 소스 코드가 들어갈 디렉터리입니다. 이 디렉터리 하위의 구조를 어떻게 설계할지는 여러분 마음입니다. Vuex를 사용하려면 [Vuex 어플리케이션을 위한 권고사항](https://vuex.vuejs.org/en/structure.html)을 참고하세요.

- `static/`

이 디렉터리는 Webpack에 의해 처리될 필요가 없는 정적 파일들을 두는 곳입니다. 이 파일들은 웹펙이 파일들을 빌드할 때 같은 디렉터리로 단순히 복사만 됩니다.

더 자세한 내용은 [Handling Static Assets](https://vuejs-templates.github.io/webpack/static.html)를 참고하세요.

- `test/unit`

유닛 테스트 파일들이 위치하는 디렉터리입니다. 자세한 내용은 [Unit Testing](https://vuejs-templates.github.io/webpack/unit.html)을 참고하세요.

- `test/e2e`

e2e 테스트 파일들이 위치하는 디렉터리입니다. 자세한 내용은 [End-to-end Testing](https://vuejs-templates.github.io/webpack/e2e.html)을 참고하세요.

- `index.html`

SPA(Single Page Application)을 위한 `index.html` 템플릿입니다. 개발과 빌드를 진행할 때, 웹팩이 어플리케이션에 필요한 파일들을 생성하고 해당 파일들에 대한 URL들을 자동으로 이 템플릿 파일에 주입하여 최종 HTML 파일을 생성합니다.

- `package.json`

빌드 디펜던시와 [빌드 명령어](./#빌드-명렁어)들이 포함되는 NPM 패키지 메타 파일입니다.

### 빌드 명령어

빌드 명령어는 [NPM Scripts](https://docs.npmjs.com/misc/scripts)를 통해 실행됩니다.

- `npm run dev`

- `npm run build`

- `npm run unit`

- `npm run e2e`

- `npm run lint`

### 참고

https://vuejs-templates.github.io/webpack/

https://github.com/vuejs-kr/vue-cli

https://vue-loader.vuejs.org/kr/start/spec.html

