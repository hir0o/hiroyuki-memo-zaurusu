# Reactの再レンダリングについて

## コンポーネントが再レンダリングするタイミング

### 1. stateが更新された時

```jsx
import "./App.css";
import Parent from "./Parent.jsx";

function App() {
  return (
    <div className="App">
      <Parent />
    </div>
  );
}

export default App;
```

```jsx
import { useState } from "react";

const Parent = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>count: {count}</p>
      <button
        onClick={() => {
          setCount((prev) => prev + 1);
        }}
      >
        count up
      </button>
    </div>
  );
};

export default Parent;
```

buttonを押すと、Parentコンポーネントだけ再レンダリングする

### 2. 親コンポーネントのstateが更新された時

```jsx
import { useState } from "react";
import Children from "./Children";

const Parent = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>count: {count}</p>
      <button
        onClick={() => {
          setCount((prev) => prev + 1);
        }}
      >
        count up
      </button>
      <div>
        <Children />
      </div>
      <div>
        <Children />
      </div>
      <div>
        <Children />
      </div>
      <div>
        <Children />
      </div>
      <div>
        <Children />
      </div>
    </div>
  );
};

export default Parent;
```

```jsx:Children.jsx
const Children = () => {
  return (
    <>
      <span style={{ backgroundColor: "pink" }}>Children</span>
    </>
  );
};

export default Children;
```

こうすると、こコンポーネントまで全部再レンダリングされちゃう

```jsx:Children.jsx
import { memo } from "react";

const Children = memo(() => {
  return (
    <>
      <span style={{ backgroundColor: "pink" }}>Children</span>
    </>
  );
});

export default Children;
```

memo化してあげると、再レンダリングが防げる



### 3. propsが更新された時


```jsx
import { useState } from "react";
import Children from "./Children";

const Parent = () => {
  const [count, setCount] = useState(0);
  const [count2, setCount2] = useState(0);
  return (
    <div>
      <h2>count: {count2}</h2>
      <button
        onClick={() => {
          setCount((prev) => prev + 1);
        }}
      >
        count up child
      </button>
      <button
        onClick={() => {
          setCount2((prev) => prev + 1);
        }}
      >
        count up parent
      </button>
      <div>
        <Children count={count} />
      </div>
    </div>
  );
};

export default Parent;
```

`count up child`がクリックされた時だけこコンポーネントが再レンダリングされる。

`count up parent`がクリックされても、Childrenはmemo化してるから、再レンダリングされない


## 再レンダリングのパフォーマンス削減

[React レンダリング最適化（useMemo, useCallback, React.Memo）](https://zenn.dev/maktub_bros/articles/da94649de294f3)

### 1. useCallback

[【React】再レンダリングの仕組みと最適化](https://zenn.dev/b1essk/articles/react-re-rendering#2.-usecallback)
[useCallbackはとにかく使え！　特にカスタムフックでは - uhyo/blog](https://blog.uhy.ooo/entry/2021-02-23/usecallback-custom-hooks/)

memo化されたコンポーネントに渡すときに力をはっきするフックす

カスタムフックで返すメソッドは常にuseCallbackを使えというのがuhyoさんのお考え

カスタムフックは外の仕様を知らないワケだから、使われているコンポーネントの都合で仕様を変えてはいけない。だから、常にuseCallbackを使う。


### 2. useMemo

計算結果を返す。

### 3. React.memo

親のstateとかpropsが更新しても、自分自身は再レンダリングしない。
使いすぎると、パフォーマンスがよくないという噂もあるが、実際は不明。
大量のコンポーネントを使ってない限りは大丈夫だと思われる。


## 再レンダリングを調べるツール

https://tmegos.hatenablog.jp/entry/react-re-render-why-did-you-render

### 1. react developper tool

この設定で再レンダリングが確かめられる

[React に優しい僕でありたい - Qiita](https://qiita.com/nabeliwo/items/de0bc076ed7105dda2ca)****

## そのほか

### React Profiler

[React Profilerの活用](https://zenn.dev/maktub_bros/articles/814a14312d2393)

コンポーネントのレンダリング時間がわかる。

どのコンポーネントのパフォーマンスを完全すればいいのかがわかる。