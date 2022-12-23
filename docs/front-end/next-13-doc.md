# Next13のドキュメントを読む

### appディレクトリ

- 新しいルーティングシステムに対応したディレクトリ。
- next.conf.jsの experiomental.appDirをtrueに設定する必要がある。
- pagesとappに同じ名前のルートがあった場合、appの方が優先される
- app内のコンポーネントはデフォルトでRSCとなる。CCを使うこともできる
- データフィッチに従来の`getHogeProps`は使えない。

### app内のスペシャルファイル

- page.tsx(必須)
	- ルートのUIを定義するために使用される.パスがアクセスできるようにするために必要
- layout.tsx(必須)
	- 複数のページで共有されるUI.他のレイアウトやページを小として受け取る
- loading.tsx(オプション)
	- layout.tsxのページ部分をSuspenceで自動でラップし、loading propsとして渡されるコンポーネント.
	- 最初のローディング時や、兄弟ルート間の移動時にローディングを表示できる.
- error.tsx(オプション)
	- layoutのページ部分をReact Error Boundartyで囲み、このコンポーネントが渡される
- template.tsx(オプション)
	- ナビゲーション時にコンポーネントの新しいインスタンスがマウントされ、ステートが共通されない。

多分こんな感じの関係性

```tsx
    <Layout>
      <Head />
      <Template>
        <ErrorBoundary fallback={<Error />}>
          <Suspense fallback={<Loading />}>
            <Page />
          </Suspense>
        </ErrorBoundary>
      </Template>
    </Layout>
```

また、app/以下のファイルは特殊ファイル以外のファイル(ロコケーションされたコンポーネント含め)もRSCになる。

### **サーバーセントリックルーティング**

従来のCSRとは異なるらしい。
サーバーセントリックルーティングはCSRでのページ遷移と同様にブラウザを再読み込みせずにページ遷移できる。
コンポーネントはサーバでレンダリングされ、サーバから送られる。(RSCペイロード) HTMLストリーミング? この結果はインメモリにキャッシュされ、同じコンポーネントをレンダリングする際にはこのキャッシュが再利用され、パフォーマンスがさらに向上する。

### ルートの定義

#### TODO ルートグループ

### リンクとナビゲート

RSCはレンダリング結果をクライアントにキャッシュする。ので、2回目いこうはキャッシュされたコンポーネントがレンダリングされる。
このきゃっしを無効化するには`router.refresh()`を使う。将来的にはミューテーションでキャッシュを無効化できる。mswのmutateみたいなやつ？

### プリフェッチ

ページ遷移まにバックグラウンドでプリロードする。RSCがレンダリングされてクライアントキャッシュに追加される。

- プリフェッチされるタイミング
	- staticなルートの場合
		- 全てのRSCがプリフェッチされる
	- dinamicなルートの場合
		- `loading.js` がプリフェッチされる
	- useRouter()のprefetchメソッドが使用されたとき

`<Link prefetch={false} />` でプリフェッチを無効にできる


### ハードナビゲーションとソフトナビゲーション

- ハードナビゲーション
	- キャッスが無効されて、サーバでデータを再取得し、変更されたセグメント(Pageコンポーネント？)を再レンダリングする
- ソフトナビゲーション
	- 変更されたセグメントのキャッシュが再利用され（存在する場合）、データのためにサーバーに新たな要求が行われることはない。
	- 条件
		- リート先がプリフェッチされており、動的セグメントを含まない
		- 現在のルートと同じ動的パラメータを持っている場合
			- ex) dashboard/red/\* → dashboard/red/\* はソフトナビゲーションになる
			- ex) dashboard/blue/\* → dashboard/ref/\* はハードナビゲーションになる
	- 戻る・進むナビゲーション[（popstateイベント](https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event)） はソフトナビゲーションになる。つまり、クライアント側のキャッシュが再利用される

## レンダリング

- statigレンダリング
	- SCとCSが事前にレンダリングされる
	- SSG,ISRなど、
- Dynamicレンダリング
	- SCとCSがリクエスト時にサーバでレンダリングされる

- レンダリングされるランタイム
	- Node.js
	- Edge Runtime
	- どちらもストリーミングデータをサポートしている？？ TODO

### SCとCCの使い分け

- SC
	- データの取得
	- 機密情報を保存する
	- 大きな依存関係をサーバに残す
- CC
	- onClickなどを追加する
	- useState、useEffectなどを使用
	- ブラウザAPIを使用

### サードパーティ制パッケージ

- CCを提供するnpmパッケージの多くは'use client';を記述していない。
- CCでそのパッケージを使うとうまくいくが、SCで使うとうまく動かない
- 独自のコンポーネントでラップすればうまく動く

```jsx
'use client';

import { AcmeCarousel } from 'acme-carousel';

export default AcmeCarousel;
```

- プロバイダーはどうするのかって話だけど、それはまた後で.

### SCからCCへのpropsの受け渡し

SC→CCのPropsはシリアライズ可能である必要がある.
`getServerSideProps`とおんなじ条件

### サーバー専用/クライアント専用のコードを分離する

- `client-only`、`server-only`パッケージで、サーバでのみ動作する関数、クライアントでのみ動作する関数を明示的にできる
- このことにより、ビルド時にエラーを発生させ、誤った場所での使用を避けられる

```
$ yarn add server-only
```

```js
import "server-only";

export async function getData() {
  let resp = await fetch("https://external-service.com/data", {
    headers: {
      authorization: process.env.API_KEY,
    },
  });

  return resp.json();
}
```

これで、`getData`はサーバでのみ動作する関数になる。

### コンテキスト

Context用のCCを作る

```jsx
'use client';

import { ThemeProvider } from 'acme-theme';
import { AuthProvider } from 'acme-auth';

export function Providers({ children }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        {children}
      </AuthProvider>
    </ThemeProvider>
  );
}
```

layoutでレンダリングする


```jsx
import { Providers } from './providers';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

### 静的レンダリングと動的レンダリング

### ランタイム

設定で切り替えれるらしい

[Rendering: Edge and Node.js Runtimes | Next.js](https://beta.nextjs.org/docs/rendering/edge-and-nodejs-runtimes)

## データフィッチ

pagesでは、getXXXPropsでデータ取得していたものを、SCの中で`fetch()`を使い、データ取得する

### fetch()

- Next.jsがglobalfetchを拡張している。
- client componentではまだ実装されてない
	- 今後、fetchやuseがCCで動作するようになるかも

デフォルトでデータをキャッシュする

```ts
fetch('https://...'); // cache: 'force-cache' is the default
```

再検証するには、optionsに、next.revaludateを加える

```ts
fetch('https://...', { next: { revalidate: 10 } });
```

一切キャッシュをしたくなければ、catch: no-store を使用

```ts
fetch('https://...', { cache: 'no-store' });
```

#### データ取得パターン

**並列データフィチ**

できるだけ並列で取得した方が表示が速くなる

```tsx
import Albums from './albums';

async function getArtist(username) {
  const res = await fetch(`https://api.example.com/artist/${username}`);
  return res.json();
}

async function getArtistAlbums(username) {
  const res = await fetch(`https://api.example.com/artist/${username}/albums`);
  return res.json();
}


export default async function Page({ params: { username } }) {
  // Initiate both requests in parallel
  const artistData = getArtist(username);
  const albumsData = getArtistAlbums(username);

  // Wait for the promises to resolve
  const [artist, albums] = await Promise.all([artistData, albumsData]);

  return (
    <>
      <h1>{artist.name}</h1>
      <Albums list={albums}></Albums>
    </>
  );
}
```

データの一部を先に見せたいときは、suspense境界を作る

```tsx
export default async function Page({ params: { username } }) {
  // Initiate both requests in parallel
  const artistData = getArtist(username);
  const albumData = getArtistAlbums(username);

  // Wait for the artist's promise to resolve first
  const artist = await artistData;

  return (
    <>
      <h1>{artist.name}</h1>
      {/* Send the artist information first,
      and wrap albums in a suspense boundary */}
      <Suspense fallback={<div>Loading...</div>}>
        <Albums promise={albumData} />
      </Suspense>
    </>
  );
}

// Albums Component
async function Albums({ promise }) {
  // Wait for the albums promise to resolve
  const albums = await promise;

  return (
    <ul>
      {albums.map((album) => (
        <li key={album.id}>{album.name}</li>
      ))}
    </ul>
  );
}
```

### fetchを使わないデータフィッチ

ルートのセグメントに応じて静的か動的かが決まる

[Segment Config Options | Next.js](https://beta.nextjs.org/docs/api-reference/segment-config)


```ts
import type { Post } from '@prisma/client';
import prisma from './lib/prisma';

export const revalidate = 3600; // revalidate every hour

async function getPosts() {
  const posts: Post[] = await prisma.post.findMany();
  return posts;
}

export default async function Page() {
  const posts = await getPosts();
  // ...
}
```