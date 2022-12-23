# React 18について

- cuncurent modeとは
- Suspenseについて

## Suspence


Promiseがthrowされて、folbackされる

Proimiseがthrowされてる状態を**レンダリングがサスペンドされた**(今はまだレンダリングできない)という。

Promiseが解決されたら、際レンダリングされる。？

errorが起きた時は、Error Boundaryで処理する。エラーハンドリングも宣言的に行える。

[はじめに｜ReactのSuspense対応非同期処理を手書きするハンズオン](https://zenn.dev/uhyo/books/react-concurrent-handson/viewer/introduction)

### suspence使うと何がいいの？

- コンポーネント同士が疎結合になる
  - レンダリングの状態を気にしなくてよくなる
- よって、コンポーネントが宣言的になる。

## uncurent mode

[Concurrent Mode時代のReact設計論 (1) Concurrent Modeにおける非同期処理 - Qiita](https://qiita.com/uhyo/items/4a6315bfccf387407631)

[🎊Reactの2種類の新フック「useTransition」と「useDeferredValue」を最速で理解する（プレビュー版） - Qiita](https://qiita.com/uhyo/items/6be96c278c71b0ddb39b)

### useTransition

- Suspenceの使用が前提としてある。
- stateの変更で、promiseをthrowするコンポーネントがある時、
- stateの変更を検知して、stateの変更後のコンポーネントの描画をしつつ、stateの変更前のコンポーネントを表示する。


具体例: [Concurrent Mode時代のReact設計論 (3) SuspenseやuseTransitionが何を解決するか - Qiita](https://qiita.com/uhyo/items/27c1282addc06b17a980)


[Concurrent Mode時代のReact設計論 (5) トランジションを軸に設計する - Qiita](https://qiita.com/uhyo/items/52ab8e2cab30b0811294)

useTransitionをどのように使うかの設計論。後半読んでない。

> 、ユーザーへのフィードバックを素早く返しつつ非同期処理を行うというユースケースでは、フィードバックを返すために必要なステート更新はstartTransitionの外で行い、非同期処理に相当するステート更新はstartTransitionの中で行う必要があります。

> それは、onClickにコールバックでstartTransitionを渡すというものです。この方針では、トランジションの所持者をButtonにしたままで前述の2つの問題を解決します。具体的にはこのようなコードになるでしょう。
