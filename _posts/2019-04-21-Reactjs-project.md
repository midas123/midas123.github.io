---
  layout: single
  title: React 프로젝트 세팅 방법
  tag: [javascript, react]
---

react 프로젝트 개발 환경 세팅을 정리했다. react는 유저 인터페이스를 만드는 라이브러리이므로
프론트 엔드만을 담당하기 때문에 백엔드에서 사용자 요청을 처리하기 위한 웹 서버가 추가로 필요하다. 

그러므로 개발 단계에서 손 쉽게 사용할 수 있는 nodejs를 먼저 설치한다. 그리고 react 프로젝트 설치 하는데 쉽고 빠른 방법([create-react-app](#create-react-app))과 복잡한 방법([webpack](#webpack) 설치 및 설정)이 있다.방법마다 장단점이 있으므로 상황에 맞는 방법을 선택하여 프로젝트를 설치를 진행하면 된다.



# nodejs & npm 설치

윈도우에서 nodejs를 [다운로드](https://nodejs.org/ko/download/) 후 설치시 npm도 같이 설치된다. node는 자바스크립트 기반의 오픈 소스 서버이다. 같이 설치되는 [npm](https://docs.npmjs.com/cli/npm)은 자바스크립트 패키지 매니져이다. CLI를 지원하므로 윈도우 명령 프롬프트에서 오픈소스 설치 명령어를 입력하면 다운로드 받은 후 설치가 진행된다.



# create-react-app 

create-react-app는 별다른 설정 없이 커맨드 한줄로 리액트 앱을 빌드해준다. 단 SPA를 고려해서 만들어졌으므로 다수의 웹페이지를 개발할 경우 적합하지 않다.

**설치**

```
npm install -g create-react-app
```

**리액트 프로젝트 생성**

```
create-react-app helloworld
```

**리액트 프로젝트 실행**

```
cd helloworld
yarn start
```

<br>

# webpack

[webpack](https://webpack.js.org/concepts)은 자바스크립트 앱을 위한 static module bundler이다. 어플리케이션의 모든 [module](https://webpack.js.org/concepts/modules/)을 맵핑하는 dependency graph를 만든다. 그리고 이를 하나 이상의 bundle로 묶어준다. bundle은 html, css 등을 포함하는 자바스크립트 파일이다. 웹 서버가 이 파일을 보내고 사용자의 웹브라우저가 출력한다.

또한, 개발 단계에서 live-reloading이 가능한 [webpack-dev-server](https://webpack.js.org/configuration/dev-server)를 제공한다. webpack은 4.0버전 이후로 기본적인 설정이 되어있다. 세부적인 설정이 필요할 경우 **webpack.config.js** 파일을 루트 경로에 생성한다. 이 파일에 저장된 설정을 webpack이 자동으로 사용한다.



## 1.프로젝트 초기화

해당 프로젝트 경로에서 아래 명령어를 입력한다.

```
npm init -y
```

npm init 실행 후 폴더 안에 package.json 파일이 생성된다.

*-package.json*

```
{
  "name": "react-test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

<br>

## 2.webpack 설치

로컬 영역에 설치(아래 코드)가 권장된다. 만약 글로벌(코드에 -g 추가)로 설치할 경우 webpack 버전이 다른 프로젝트 간에 충돌이 일어날 수 있다.

```
npm install --save webpack

//버전 입력시
npm install --save-dev webpack@<version>
```

※webpack 4.0 이상을 사용할 경우 [CLI](https://webpack.js.org/api/cli/)를 추가로 설치해야 한다.

```
npm install --save-dev webpack-cli
```

 ※ --save-dev는 -D와 같다.

<br>

## 3.프로젝트 모듈 설치

**react 설치**

```
npm install --save react react-dom
```

**개발 모듈 설치**

```
npm install --save-dev react-hot-loader webpack webpack-dev-server

//6.x
npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react
//7.x이후 최신버전
npm install --save-dev @babel/core babel-loader @babel/preset-env @babel/preset-react

npm install css-loader style-loader --save-dev
```

<br>

## 4.package.json 설정

이 파일에서는 주로 프로젝트에서 사용하는 dependency를 정의한다.

**scripts**: 로컬 webpack을 CLI에서 쉽게 사용할 수 있도록 [npm script](https://docs.npmjs.com/misc/scripts)를 작성한다. 

```javascript
{
  "name": "react-test2",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack-dev-server --mode development --open --hot",
    "build": "webpack --mode production"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "react": "^16.8.6",
    "react-dom": "^16.8.6"
  },
  "devDependencies": {
    "@babel/core": "^7.4.3",
    "@babel/preset-env": "^7.4.3",
    "@babel/preset-react": "^7.0.0",
    "babel-loader": "^8.0.5",
    "css-loader": "^2.1.1",
    "html-webpack-plugin": "^3.2.0",
    "react-hot-loader": "^4.8.4",
    "style-loader": "^0.23.1",
    "webpack": "^4.30.0",
    "webpack-cli": "^3.3.1",
    "webpack-dev-server": "^3.3.1"
  }
}
```

위 설정 중 scripts 안에서 "start"에 할당한 명령어를 커맨드 창에서 아래처럼 실행할 수 있다.

```
npm start
```

<br>

## 5.webpack.config.js

webpack은 entry에 있는 모듈의 의존 관계를 이용해서 번들 작업을 진행한다. webpack.config.js에서는 entry 파일과 output 파일을 정하고 프로젝트의 asset 파일(css, js 등)을 모듈로 등록한다.

**entry**: dependency graph의 시작점, webpack은 entry point에서 직/간접적으로 의존하고 있는 모듈 또는 라이브러리를 알아내서 빌드한다.

**output**: webpack이 만들어내는 bundle의 저장 위치와 이름을 정한다.

**[loaders](https://webpack.js.org/concepts/loaders/)**: webpack이 다른 타입의 파일을 처리하도록 돕는다. (ex. babel-loader, style-loader, css-loader)

- test 프로퍼티는 require() / import statement 안에 있는, 변환되야할 파일을 가리킨다. 

  참고:[asset-management](https://webpack.js.org/guides/asset-management#setup)

- use 프로퍼티는 사용될 loader를 가리킨다.

*※loader 사용시 주의사항*(출처 [링크](https://webpack.js.org/concepts/#loaders))

> It is important to remember that when defining rules in your webpack config, you are defining them under `module.rules` and not `rules`. For your benefit, webpack will warn you if this is done incorrectly.

> Keep in mind that when using regex to match files, you may not quote it. i.e `/\.txt$/` is not the same as `'/\.txt$/'`/ `"/\.txt$/"`. The former instructs webpack to match any file that ends with .txt and the latter instructs webpack to match a single file with an absolute path '.txt'; this is likely not your intention

**[plugin](https://webpack.js.org/api/plugins/)**: 번들 최적화, asset 관리, 환경 변수 injection 등의 업무를 수행하는 플러그인 설정. 아래 코드의HtmlWebpackPlugin처럼 상수를 만들고 plugins 배열에 추가한다.

```react
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.join(__dirname, "/dist"),
    filename: "index-bundle.js"
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: ["babel-loader"]
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html"
    })
  ]
};
```



