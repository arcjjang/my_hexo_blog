---
title: Electron + React + TypeScript
date: 2018-02-02 15:50:29
tags: [electon, react, typescript]
cover: /images/elec_ts_react.png
categories: [tip]
subtitle: 이제 C#은 묻어두자...
---
# 개발환경 구축

※ 기본 개발환경
- Visual Studio Code
- Node.js

### 1) npm 모듈 설치하기
``` bash
npm init
npm install electron --save-dev
npm install react react-dom --save
npm install webpack --save-dev
npm install typescript --save-dev
npm install tslint --save-dev
npm install @types/electron --save
npm install @types/react --save
npm install @types/react-dom --save
npm install awesome-typescript-loader --save-dev
npm install source-map-loader --save-dev
npm install css-loader --save-dev
npm install style-loader --save-dev
```

### 2) typescript config 설정하기 (tsconfig.json)
``` json
{
    "compilerOptions": {
        "allowSyntheticDefaultImports": true,
        "outDir": "./dist/",
        "sourceMap": true,
        "noImplicitAny": true,
        "module": "commonjs",
        "target": "es6",
        "jsx": "react"
    }
}
```

### 3) typescript tslint 설정하기 (tslint.json)
``` json
{
  "rules": {
      "class-name": true,
      "comment-format": [
          true,
          "check-space"
      ],
      "indent": [
          true,
          "spaces"
      ],
      "no-duplicate-variable": true,
      "no-eval": true,
      "no-internal-module": true,
      "no-trailing-whitespace": true,
      "no-unsafe-finally": true,
      "no-var-keyword": true,
      "one-line": [
          true,
          "check-open-brace",
          "check-whitespace"
      ],
      "quotemark": [
          true,
          "single"
      ],
      "semicolon": [
          true,
          "always"
      ],
      "triple-equals": [
          false,
          "allow-null-check"
      ],
      "typedef-whitespace": [
          true,
          {
              "call-signature": "nospace",
              "index-signature": "nospace",
              "parameter": "nospace",
              "property-declaration": "nospace",
              "variable-declaration": "nospace"
          }
      ],
      "variable-name": [
          true,
          "ban-keywords"
      ],
      "whitespace": [
          true,
          "check-branch",
          "check-decl",
          "check-operator",
          "check-separator",
          "check-type"
      ]
  }
}
```

### 4) webpack config 설정하기 (webpack.config.js)
``` javascript
module.exports = {
  target: "electron",
  node: {
    __dirname: false,
    __filename: false,
  },
  entry: {
    "main/main": "./src/main/main.ts",
    "renderer/index": "./src/renderer/index.tsx"
  },
  output: {
    filename: "dist/[name].js"
  },
  // Enable sourcemaps for debugging webpack's output.
  devtool: "source-map",
  resolve: {
    // Add '.ts' and '.tsx' as resolvable extensions.
    extensions: [".ts", ".tsx", ".js", ".json"]
  },
  module: {
    rules: [
       // All files with a '.ts' or '.tsx' extension will be handled by 'awesome-typescript-loader'.
       { test: /\.tsx?$/, loader: "awesome-typescript-loader" },
       // All output '.js' files will have any sourcemaps re-processed by 'source-map-loader'.
       { enforce: "pre", test: /\.js$/, loader: "source-map-loader" },
       {test: /\.css$/, loaders: ["style-loader","css-loader"]}
    ]
  }
};
```

### 5) 최상위 폴더에 index.html 추가 (Main 프로세스에서 사용)
``` html
<!DOCTYPE html>
<htm>
<head>
  <meta charset="UTF-8">
  <title>Title...</title>
</head>
<body>
  <div>
    <div id="root"></div>
  </div>
  <script>require("./dist/renderer/index.js")</script>
</body>
</html>
```

### 6) 프로젝트 폴더 구성하기

#### 6-1) src -> main -> createMainWindow.ts
``` javascript
import { BrowserWindow } from "electron";

class MainWindow {
  window: BrowserWindow;
  constructor() {
    this.window = new BrowserWindow({ width: 800, height: 600 });
    this.window.loadURL(`file://${__dirname}/../../index.html`);
    this.window.on("closed", () => {
      this.window = null;
    })
  }
}

function createMainWindow() {
  return new MainWindow();
}

export default createMainWindow;
```

#### 6-2) src -> main -> main.ts
``` javascript
import { app } from "electron";
import createMainWindow from "./createMainWindow";

let mainWindow = null;

app.on("ready", () => {
  mainWindow = createMainWindow();
});

app.on("window-all-closed", () => {
  if (process.platform !== "darwin") {
    app.quit();
  }
});

app.on("activate", (_e, hasVisibleWindows) => {
  if (!hasVisibleWindows) {
    mainWindow = createMainWindow();
  }
});
```

#### 6-3) src -> renderer -> components -> app.tsx
``` javascript
import * as React from 'react';

class App extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello World</h1>
      </div>
    );
  }
}

export default App;
```

#### 6-4) src -> renderer -> index.tsx
``` javascript
import * as React from 'react';
import * as ReactDOM from "react-dom";
import App from "./components/App";

ReactDOM.render(<App />, document.getElementById('root') as HTMLElement);
```

### 6) npm scripts 추가
``` json
"start": "electron .",
"build": "webpack",
"watch": "webpack --watch"
```

### 7) npm electorn 진입점 변경하기
``` json
  "main": "index.js",
```
-> 
``` json
  "main": "dist/main/main.js",
```

### 8) 배포 환경 설정하기 (target: win10)
``` bash
npm install asar --save-dev
npm install electron-packager --save-dev
```
npm script 추가
``` json
"package": "npm run build && electron-packager . \"Electron Delpoy App\" --overwrite --asar --platform win32 --arch x64 --out release"
```

> https://github.com/arcjjang/electron_react_typescript
