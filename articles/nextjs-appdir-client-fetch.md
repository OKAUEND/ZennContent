---
title: 'Next.js 13 App RouterのRoute HandlerでQueryを取る'
emoji: '🤖'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['react', 'nextjs']
published: true
---

## App Router

Next.js 13.4 で新機能の App Router が安定版となりましたが、皆様はいかがお過ごしでしょうか。
私は開発途中だった Web アプリを、機能的に App Router のほうがよさそうと感じたので移植作業をしています。
App Router では、Next.js 固有の構文などが減り React に近い書き方が可能になったのは大きいと思います。

とはいえ、Pages Router でやれたことが App Router ではどうするの？って部分が若干あったので、今回はそれを調べて確認してみました。

## Route Handlers

Pages Router で API Router がありましたが、App Router では Route Handlers に名称が変更され、構造も微妙に変わっています。

https://nextjs.org/docs/app/building-your-application/routing/router-handlers

```ts:pages/api/hello
export default function handler(req, res) {
  if (req.method === 'GET') {
    // Process a GET request
  } else {
    // Handle any other HTTP method
  }
}
```

API Router での一番基本の形はこのようになるでしょうか。
これが App Router の Route Handlers になるとガラッと構造が変わります。

```ts:app/api/hello/route.ts
export async function GET() {
  return NextResponse.json({ data:"Hello Next.js 13 !!" });
}
```

一番の変更点は、関数名が`handler`から`GET`に変わったことでしょうか。
API Router では`handler`の Request からメソッドを取得し、そこからメソッド毎の処理を分岐させていましたが、
Route Handlers では、それぞれのメソッド毎に関数が用意されており、必要に応じて関数を定義する形となりました。

https://nextjs.org/docs/app/api-reference/file-conventions/route

### Query Parameter を取る

API Router では、Query Parameter を取得するには、Request から query を取得し、さらにそこから Parameter を…という形でした。

```ts:API Router
export const handler: Handler = async (request, response) => {
    const { method, query } = request;
    const zenn = query.zenn;
}
```

Route Handlers でも Request から Query を取得できますが、Slug を別途取得することができるのも特徴です。
https://nextjs.org/docs/app/building-your-application/routing/router-handlers#dynamic-route-handlers

```ts:Route Handlers
export async function GET(
  request: Request,
  {
    params,
  }: {
    params: { slug: string };
  }
) {
  const { searchParams } = new URL(request.url);
  const zenn = searchParams.get('zenn');
}
```

Request.url から新しく URL インスタンスを作成し、そこから get 関数を使い目的のクエリパラメーターを取得する形となります。
さらに、Slug の場合は、URL から取得せずとも params を引数で宣言することで、取得が可能です。

```ts:Route Handlers
import { NextRequest } from 'next/server';

export async function GET(
  request: NextRequest,
  {
    params,
  }: {
    params: { slug: string };
  }
) {
    request.nextUrl.searchParams.get('zenn')
}
```

ちなみに、NextRequest で型指定を行うと、URL インスタンスを宣言せずとも取得ができるので、こちらの方が楽だとは思います。

## 終わりに

App Roter はまだそこまで扱いきれてませんが、ふんわり触った感じでもとてもいい感じに組めるので、積極的に試していきたいですね。
