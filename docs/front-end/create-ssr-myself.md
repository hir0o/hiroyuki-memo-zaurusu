# SSRを自作してみる

## 目的

- エンジニアとしての基礎力を伸ばす
- SSRを理解する
- Webpack,babelのツール周りを理解する
- reactを理解する

## memo

とりあえず以下の記事

https://zenn.dev/takurinton/articles/4c8625a43f024b

ざっくり、fastifyで、h関数を実行して、文字列をクライアントに返すイメージはできた

とりあえずh関数がなんなのかを理解したい。

### h関数

そもそもJSXとは?

- 自分の理解
  - React.createElementのエイリアス的なやつ。
  - babelでコンパイルされ、React.createElementに変換される
- 記事を読んでの理解
  - JSXは、JSをHTML風にかけるシンタックスである。
  - JSXは、babelの`transform-react-jsx`というプラグインを通して、h関数に変換される。
  - h関数は結構抽象的な概念で、JSXを引数にとって、HTML文字列を返す関数のことを指す。(Reactだと、React.createElementとか)
  - なぜh関数と呼ばれるのかというと、界隈でよく使われるかららしい。語源知りたい。

### h関数でHTMLを生成してみる

テキトーなコンポーネント

```js
import React from "react";

export const hello = () => {
  return React.createElement("h1", {}, "Hello, world!");
};
```

HTMLを生成するには、`ReactDomServer.renderToString()`でできる。

[ReactDOMServer – React](https://ja.reactjs.org/docs/react-dom-server.html)

console.logで確認してみる

```js
import ReactDOMServer from "react-dom/server";
import { hello } from "./h-hunc";

function App() {
  const helloJsx = hello();
  const hString = ReactDOMServer.renderToString(helloJsx);
  console.log(hString); // <h1 data-reactroot="">Hello, world!</h1>
  return null;
}

export default App;
```

他にも`renderToStaticMarkup()`というメソッドがあって、こちらは、`data-reactroot`とかを付与しない。これ使ってSGとか作れそう。


[ReactやVueのJSXについて曖昧に理解する - Qiita](https://qiita.com/nabepon/items/87bb3b4f1e7bfa342489)


### expressからreanderToString()したHTML文字列を返してみたい。

単純に実装してみたけど、これだとできないっぽい。


```js
import express from "express";
import React from "react";
import ReactDOMServer from "react-dom/server";

const app = express();

app.get("/", (req, res) => {
  const App = React.createContext("h1", {}, "helloWorld!!!");
  const html = ReactDOMServer.renderToString(App());
  res.send(html);
});

app.listen(3000, () => {
  console.log("Listening on port 3000");
});
```

> Error [ERR_MODULE_NOT_FOUND]: Cannot find module '/Users/shibuyahiroyuki/projects/react/my-react-app/node_modules/react-dom/server' imported from /Users/shibuyahiroyuki/projects/react/my-react-app/src/server.js

できねいー

bubelとwebpackに少し詳しくなってからだとできるかも？

### hydration

hydrationとは

[Hydration (web development) - Wikipedia](https://en.wikipedia.org/wiki/Hydration_(web_development))

> 静的ホスティングまたはサーバーサイドレンダリングによって配信された静的な HTML ウェブページを、クライアントサイド JavaScript が HTML 要素にイベントハンドラを添付して動的ウェブページに変換する手法のこと

動的ページっていうのは、SPA的なやつではなく、単にボタンを押したらなんか要素が動くとか、そういうやつ。

単純にHTMLをクライアントに送ってしまうと、イベントハンドラが付与されずにユーザに表示されるため、操作できそうに見えても操作できないページが表示されてしまう。

Hydrationを使うと表示する前にイベントハンドラを付与するから、ユーザに優しい。

mizchiさん解説

https://twitter.com/mizchi/status/1353615997635227650

なんかの傍聴

[【視聴メモ】Rendering on the Web: Performance Implications of Application Architecture (Google I/O ’19) - Qiita](https://qiita.com/karszawa/items/5a5e591656d5a44c6290)

https://blog.saeloun.com/2021/12/16/hydration.html

[New Suspense SSR Architecture in React 18 · Discussion #37 · reactwg/react-18 · GitHub](https://github.com/reactwg/react-18/discussions/37)

なんとなく理解した。

### webpack

#### 参考記事

「webpack 入門」で検索したらこれが一番最初に出てきた。
[最新版で学ぶwebpack 5入門 - JavaScriptのモジュールバンドラ - ICS MEDIA](https://ics.media/entry/12140/)

いいシリーズ

[webpackとBabelの基本を理解する(1) ―webpack編― - Qiita](https://qiita.com/koedamon/items/3e64612d22f3473f36a4)

なんとなく、CRA使わずにReact開発環境作ると結構知識が手に入りそう。とりあえず基礎を勉強する。

`webpack.config.js`を制するものはwebpackを制する.


#### memo

超基本構成。デフォルトで以下の設定になってる。

```js
module.exports = {
  // メインとなるJavaScriptファイル（エントリーポイント）
  entry: `./src/index.js`,
  // ファイルの出力設定
  output: {
    //  出力ファイルのディレクトリ名
    path: `${__dirname}/dist`,
    // 出力ファイル名
    filename: "main.js"
  }
};
```

**ソースマップ**

開発環境で必要になる、元のファイルとの関係を表すファイルのこと。

modeにdevelopmentを指定すると、ソースマップも一緒に書き出される。

```js
module.exports = {
  mode: "development"
```

Loaderとは

Loaderは、ファイルをバンドルする前に、コードに対して何かしらの処理(babelでバンドルするとか)を行いたい時に使用する。

jsファイル以外のcssとか画像ファイルに対して何かしら処理をしたい時もloaderを使う。

Loaderの指定方法

webpack.config.jsのmoduleに指定する。

```
module.exports = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [ // 配列でloaderを指定
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};
```

翻訳↓

> やあ！webpackコンパイラ、君がimportかrequire文で'.txt'に該当するファイルに遭遇したら、バンドる前にraw-loaderを使ってそのファイルを変換してくれないかな。


オプション

- exclude
  - node_modulesフォルダ内のファイル群など、loaderの対象外としたい場合は、rulesにexcludeを追加します。
- options
  - loaderに設定情報を持たせたい場合は、useの項目をloaderとoptionsに分けて記述します。


例

```
module.exports = {
  module: {
    rules: [
      {
        test: /\.txt$/,
        exclude: /node_modules/,　//対象外ディレクトリ
        use: [
          {
            loader: 'raw-loader', //loader名
            options: {...}        //設定情報
          },
          'a-loader',
          'c-loader'
        ]
      }
    ]
  }
};
```

バベルと併用する場合ば、babel-loaderを使う。

**参考**

[webpackとBabelの基本を理解する(3) ―webpackとBabel編― - Qiita](https://qiita.com/koedamon/items/6cf2201be78c3d79516d)

### babel

枠割

- 新しい文法を古い文法に書き換える
- jsxをh関数に変換する
- TypeScript→JavaScriptの変換をする

#### plugins

babelの変換する処理をするやつ。変換はpluginを通して行われる。

[作って理解する Babel プラグイン - Techtouch Developers Blog](https://tech.techtouch.jp/entry/build-and-understand-babel-plugin)

#### Preset

pluginsをまとめたやつ。

基本的に、babelが用意してる？presetを使って、変換処理を行う。

以下のpresetが提供されてる。([Presets · Babel](https://babeljs.io/docs/en/presets/))

- @babel/preset-env for compiling ES2015+ syntax
- @babel/preset-typescript for TypeScript
- @babel/preset-react for React
- @babel/preset-flow for Flow

**参考**

[webpackとBabelの基本を理解する(2) ―Babel編― - Qiita](https://qiita.com/koedamon/items/92c986456e4b9e845acd)

[【５分でなんとなく理解！】Babel入門 - Qiita](https://qiita.com/Shagamii/items/a87181c22ea777ee2acc)



babelとwebpaclを組み合わせる構成は、`webpack.config.js`のmoduleにbabelの設定を書く必要がある。

```js
...
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/, // babelを通さないディレクトリ
        loader: "babel-loader",
      }
    ]
  }
...
```




#### craやら使わずにReactの環境を作る

[webpack ＋ TypeScript／Babel（JavaScript）の環境でReactを導入する](https://designsupply-web.com/media/programming/6902/)

webpackのts-loaderとbabel-loaderを併用する理由

[【webpack5】ts-loader + babel-loader を併用する【TypeScript】 - Qiita](https://qiita.com/heppokofrontend/items/fbaa85134a89835da82c)

memo


preset-envだけでビルドしてみたけど、できなかった。jsx使ってないからいけると思った。preset-reactが必要そう。

→普通にimport間違ってた。

```js:index.js
import React from "react";
import ReactDOM from "react-dom";

ReactDOM.render(
  React.createElement("h1", null, "Hello, world!"),
  document.getElementById("app")
);
```

```html:index.html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="app"></div>
    <script async defer src="http://localhost:3000/dist/bundle.js"></script>
  </body>
</html>
```

`<script async defer src="http://localhost:3000/dist/bundle.js"></script>`がポイント

webpackの設定は以下のようにbabel-loader通さなくてもちゃんと表示された？

```js
module.exports = {
  mode: "production",
  entry: "./src/index.js",
  output: {
    path: __dirname + "/dist",
    filename: "bundle.js",
  },
};
```

次はjsxを使えるようにする。確かjsx用のプラグインがあった。

`yarn add -D babel-plugin-transform-react-jsx`

プラグインは必要なく、`@babel/preset-react`だけでbuildできた。

ボタンとか作れるかな、、、

`import Button from "./button"`したらファイルないって怒られた。

>  /Users/shibuyahiroyuki/projects/react/react-webpack/src/button doesn't exist

```
  resolve: {
    extensions: ['.js', '.jsx', '.tsx'],
  },
  ```

を追加してみる.



できた

[GitHub - hir0o/react-express-ssr: SSR作ってみる](https://github.com/hir0o/react-express-ssr)


## 参考記事

[How to implement server-side rendering (SSR) in your React application with NodeJS - step by step tutorial - Leber Software](https://lebersoftware.hu/react-server-side-rendering-step-by-step-tutorial/)

[Server Rendering with React and React Router](https://ui.dev/react-router-server-rendering)

[React.jsのSSRをTypeScriptで自前で実装してみた](https://zenn.dev/hotsukai/articles/react_ssr_scratch)
