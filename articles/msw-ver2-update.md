---
title: "MSW2.0になった経緯をブログを読んで考えてみた"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
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
import { http, HttpResponse } from "msw";

const Handler = () => {
  return http.get("/api/user", () => {
    return HttpResponse.json({ message: "Hello World" });
  });
};
```

2.0 ではこのようにインポートするメソッドや関数などが変化しています。
2.0 以前では rest(おそらく REST API の REST が由来？)が http に変化しており、
また戻り値を設定するのも、引数で定義するのではなくインポートしたのを利用する方法に変化しております。

# どうして変わったのか

## 2.0 以前に OSS チームが感じていた問題点

### API の高い抽象度

### 根本的な Web 技術の理解が難しい

# MSW 2.0 のブログと OSS から学び取れる事

# おわりに
