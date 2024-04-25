---
title: "MSW2.0になった経緯をブログを読んで考えてみた"
emoji: "📌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [msw]
published: false
---

# はじめに

MSW のメインバージョンが 2.0 へバージョンアップをされました。
かなり破壊的な変更を伴う更新でしたがブログに経緯が書かれていたので、
それを読み解きながら、今回なぜそれが行われたかと OSS を使う側のエンジニアとしての視野を考えてみたいと思います。

# MSW2.0 とは

簡単にですが MSW とは何かを振り返っておこうと思います。
Mock Service Worker (MSW) は、API の振る舞いをモック化するためのライブラリです。
開発中のフロントエンドアプリケーションに対して、実際の API を呼び出すことなく、あらかじめ用意したレスポンスデータを返すことができます。
仮のデータを用意し設定することで、API の実装に依存することなく開発を進めることができます。
特に、バックエンドと平行で開発が進んでいた場合、まだ API が完成しておらずデータも返ってこない状態で、
フロントエンド側では本番で使用する形での API を呼び出す関数などを実装出来るため、わざわざ仮な状態を作らずともいいという利点があります。

## なにが変わったのか

では主にどういう風に変わったのかを、Migration のページを見ながら簡単に確認してみます。
https://mswjs.io/docs/migrations/1.x-to-2.x

### 今までの API

今までの MSW のハンドラーの作り方は、形にはよりますがベースは以下の通りです。

```ts
import { rest } from 'msw';

const Handler = () => {
  return rest.get('/api/user', (req, res, ctx) => {
    return res(ctx.status(200), ctx.json({ message: 'Hello World' }));
  });
};
```

ではこれが 2.0 でどう変更したのかを確認してみましょう。

### これからの API

```ts
import { http } from "msw";

const Handler = () => {
  return http.get("/api/user", () => {
  return new Response('Hello World')
  });
};
```

2.0 ではこのようにインポートするメソッドや関数などが変化しています。
2.0 以前では rest(おそらく REST API の REST が由来？)が http に変化しており、
また戻り値を設定するのも、引数で定義するのではなくインポートしたのを利用する方法に変化しております。

# どうして変わったのか

OSS で破壊的変更はあまり好まれない傾向にありますが、MSW の OSS チームがなぜこういう判断をしたのか。
MSW の公式ブログにその経緯や考えが記事になっていたので、確認しつつ自分がどう扱っていたかを確認してみます。

ということで、公式ブログの記事を見て確認していきましょう。
https://mswjs.io/blog/introducing-msw-2.0

## 2.0 以前に OSS チームが感じていた問題点

```ts
(req, res, ctx) => res(...)
```

公式では Response Resolve 関数と読んでいる関数は、非常にうまく実装できており、
Response の抽象化が適切であり、コードリーディングがしやすいと評価しています。
しかし、それが故に公式が目標としたことと、環境に順応できていないという問題が発生してとされています。

### 根本的な Web 技術の理解を促せなかった

> It failed to educate.
>
> The degree of abstraction in the res() function is far too high to teach you, the developer, anything about actual responses on the web. As a maintainer with >thousands of projects depending on the software I build, I feel it’s my responsibility to care about what developers learn from that software. I want them to >achieve their goals but I also want them to learn concepts and APIs they can apply even outside of MSW because I firmly believe that’s what a good software does.

実装した Response Resolve 等の API が高度に抽象化がされすぎ、教育に失敗しましたと書かれています。
たしかに、MSW を使って Web の Response や Request 周りの実装を能動的に学びにいった事はほぼありませんでした。
というのも、私も個人プロダクト及び業務でも使用していますが、MSW の利用方法を調べたり実装方法を工夫したりしましたが、
MSW を通して Web の Request や Response を感じ取れたことはなかったなとの振り返りでした。
MSW チームとしては、MSW を詳しくなると同時に Web の API とかを学べるようにするのが理想だったようです。

#### どう変えたのか

ではどう変えたのかを見ながら、MSW チーム

```ts
import { http } from "msw";

const Handler = () => {
  return http.get("/api/user", async ({ request }) => {
    const user = await request.json();
    return new Response(`Hello World! ${user}`)
  });
};
```

or

```ts
import { http, HttpResponse } from "msw";

export const Handler = () => {
  return http.get("/api/user", () => {
    return HttpResponse.json({ message: "Hello World" });
  });
};
```

上のほうでも書きましたが、2.0 になってからは戻り値で返す値の指定を`rest`の関数で行うのではなく、
Fetch API の`Response`のインターフェイスを利用して行うようになっています。
MSW のライブラリからインポートしている`HttpResponse`もクラスとして宣言されていますが、元となった型は Fetch API の`Response`なので、
MSW の`HttpResponse`を使おうが、両方とも Fetch API の`Response`を使用するってことになります。
`request`の方も Fetch API のインターフェイスを利用しています。

`http.get`などのリクエストメソッド部分を Mock する宣言などは、以前の`rest.get`等と同じですが、
内部の Fetch API のインターフェイルになっているので、今までの MSW で慣れていた人からしたら仕様変更に最初は慣れないかもです。
ただ、MSW の独自仕様から Fetch API のインターフェイスになったので、MSW チームが目標としている Web の通信周りの知識やスキルを身につける事ができる事は確かです。
~~筆者も MSW の仕様に慣れてて、最初結構戸惑っていました~~

# おわりに

## MSW 2.0 のブログと OSS から学び取れる事、感じた事

Web 界隈ですと、それこそ毎日のように新しいライブラリが公開されていて、これ便利だから使おうって感じで使うことが個人としては多いと思います。
そうなると、ライブラリを組み合わせる職人みたいになって、Web のスキルが伸びるというよりかはライブラリに依存しないと何もできないになっちゃう事が多いのかなと。
~~若干私もそうなっている部分がある…~~
プロダクトで実装していく時には、ライブラリの組み合わせで実装が行えるかもそうですが、そうではない時にはどうすればよいかも考慮していく必要があるので、
そうなると Web の知識をベースとした設計力が必要だなと最近痛感しているので、MSW のこのブログ記事を読んで思いました。
回り道にはなるかもしれませんが、ある程度実装力がついたら、今一度振り返って基礎を勉強し直すのはとても重要かもしれませんね。
