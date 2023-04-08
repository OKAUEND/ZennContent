---
title: "Next/RouterのMock - Vitest"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "vitest"]
published: true
---

# はじめに

Next.js で next/router を利用する場合、Vitest(もしくは Jest)を使った環境化では、next/router の Hook が null になります。
その理由は、next/router の`useRouter`は React Hook でありクライアント環境で動作するため、Vitest や Jest などの Node.js などサーバーサイドでは動作しないためです。
ユニットテスト含め、サーバーサイドで`useRouter`のインスタンスにある要素へアクセスするコードがある場合には、事前に`Mock`化させて上げる必要があります。

```ts
src/index.test.tsx > TEST Component > 次を取得ボタンをクリックした時、関数を呼び出せているか
TypeError: Cannot read properties of null (reading 'query')
 ❯ RouterTEST src/pages/index.tsx:16:39
     15|     const router = useRouter();
     16|     const query = router.query.test
       |                          ^
```

上記の処理は、`useRouter`からパスに含まれている Query を取り出そうとしているコードです。
`Mock`無しで要素へアクセスする場合、当然存在しないので、null になりテストでエラーを吐いちゃいます。

本記事では、next/router の Mock 方法について解説します。

# 解決

## ライブラリを使おう

`Mock`化する方法と書いておきながらなんですが、解決方法の一つとしてライブラリを使うという手段があります。
Google で検索したり、ChatGPT に聞くなりでおすすめのライブラリが出てくると思います。
`Mock`化するのめんどいとか、自分で書いてたらミスをしそうという場合には、素直にライブラリを使ったほうがいいかもしれません。

## 自力で Mock

~~うるせぇ！俺は`Mock`を自力でするんだよ！~~
ライブラリも当然、Next.js のバージョンアップやテスト環境の変化、Node.js のバージョンアップなどで使用不可能になるということもあると思います。
また、ライブラリを使うにも該当箇所が少なく、それなら自力で`Mock`したほうが環境を汚さずにすむなどの利点もあります。

### vi.mock

```ts
// __tests__/sample.test.js
import { vi } from "vitest";

describe("Sample test", () => {
  vi.mock("next/router", () => ({
    useRouter() {
      return {
        route: "/",
        pathname: "",
        query: "MockVitest",
        asPath: "",
      };
    },
  }));
  test("useRouter should return mock data", () => {
    // テスト処理
  });
});
```

`vi.mock`で next/router まるごと Mock する方法です。
ちなみに`Jest.mock`でも同じ方法で Mock 可能です。

### spyOn

```ts
// __tests__/sample.test.js
import { vi } from "vitest";

describe("Sample test", () => {
  test("useRouter should return mock data", () => {
    vi.spyOn(require("next/router"), "useRouter").mockImplementationOnce(
      () => ({
        query: "Mock_Vitest",
      })
    );
    // テスト処理
  });
});
```

`vi.spyOn`を使った方法です。

vi.mock も SpyOn も Mock データ次第では、Query の部分を

```ts
  query: { mock: 'Vitest' },
```

と、オブジェクトのようにネストしたりできるので、実際の使用時に状況におおじて設定してください。

# おわりに

今回は、Vitest を使用した React アプリケーションで next/router の Mock 方法について解説しました。
next/router の Hook が Vitest のテスト時に null なる問題を解決するためには、上記の手順を参考にして、Mock を行ってみてください。

## ちなみに…

この記事は ChatGPT-3.5 を用いて土台を作成し、そこから細かい部分や実例などを追加編集することで作成しました。
