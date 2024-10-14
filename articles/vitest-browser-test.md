---
title: "Vitest Browser Modeを試してみた"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# はじめに

何度か調べているTESTツールの記事第三段です。
今回もVitestですが、実験中である`Browser mode`を調べてみたいと思います。

## 前回

前回は[Playwright]()と[Vitest UI mode]()を調べて、実際のテストの様子を確認できるかを調べて記事にしました。
Playwrightでは`Playwright Component`を使い、PlaywrightのUI modeと同じく単体テストとコンポーネントテストのテスト状態を確認するかを調べました。
`Vitest UI mode`では、PlaywrightのUI modeと同じように動作するかを確認しましたが、機能としてはCLIでの実行結果などをブラウザ上でUIにし確認が可能になるといった機能でした。
ここまでの確認で、E2Eテスト以外ではテスト実行などを目視することが可能な機能は見つける事ができなかったといった結果でした。

# Vitest Browser Modeとは？

[Vitest Browser Mode](https://vitest.dev/guide/browser/)
Vitestの新しい機能です。
ステータスは`Experimental`となり、まだ実験中の機能となります。

今まではコンポーネントテストを行うためにVitestでもjsdomなどのライブラリを導入して、仮想のブラウザ環境を作りコンポーネントテストを行うといった感じの下準備が必要となっていました。
~~それが果てしなくめんどい設定でいつも大変~~
さらに、仮想のブラウザ環境はあくまで仮想のブラウザ環境であるため、厳密にユーザーが使うブラウザと完全に一致しているわけではないなどの問題もあったと聞いております。
`Vitest Browser Mode`では、Playwrightなどを使用することで実際のブラウザ環境上でテストを行い、環境づくりの簡易化と実際のブラウザ環境でのテスト実施を実現しようとしています。
（まだ実験中のステータスの為この表現

- @vitest/browser : 2.1.2

## 環境設定

では早速環境設定の構築からやっていきましょう。
といいますが、環境構築自体はめちゃくちゃ楽に終わります。

```
npx vitest init browser
```

このCLIを入力すると、次はTypeScriptかJavaScriptかを選びます。
その次はテストをするためのブラウザ環境をどうするかを聞かれます。
PlaywrightかCypressにするかpreviewにするかを選べますが、previewは公式ドキュメント曰くCIで実行ができないためPlaywrightなどを選ぶ必要があるみたいです。
あとは、ReactかVueかなどをを聞かれますが、使いたいフレームワークの環境を選ぶ事で導入は完了です。

あとは、テストを実行するとHello Worldのサンプルテストが実行されます。

```
npm run test:browser
```

すると、このような画面が出現しコンポーネントの様子も見えてテストの成否状態も確認できます。

![テスト結果](/images/vitest-browser-test/initResult.png)

## テストコードを書いてみる

### ブラウザモード用に設定ファイルを編集

コンポーネントのテストをする前に少し設定を変更します。
理由としては、Browser modeのテストと通常のテストが混在した場合にBrowser modeのテストはどちらかは実施出来ないからです。
現状ではBrowser modeで使用するライブラリなどはBrowser modeでしか使用出来ないためです。

`vitest.workspace.browser.ts`のファイルが作成されているはずなので、そちらを編集します。

```
import { defineWorkspace } from 'vitest/config'

export default defineWorkspace([
  // If you want to keep running your existing tests in Node.js, uncomment the next line.
  // 'vite.config.ts',
  {
    extends: 'vite.config.ts',
    test: {
      name: 'browser',
      include: ["src/**/*.browser.test.tsx"],
      browser: {
        enabled: true,
        name: 'chromium',
        provider: 'playwright',
        // https://playwright.dev
        providerOptions: {},
      },
    },
  },
])

```

`include: ["src/**/*.browser.test.tsx"]`と設定することで、`*.browser.test.tsx`のファイル名のみがテスト対象となります。
ちなみに`.browser.`のBrowserは便宜上Browser modeのテストなので総名前付けしているだけで、任意に設定可能です。

### 今回のテストコンポーネント

ではコンポーネントテストを行っていきましょう。
今回のテスト対象は、前回までのを流用しつつ一部構成を変えてあります。
変えた理由としては、要素の構成としてより適切に変えただけなのでテストや動作には特に影響がない部分です。

```:typescript
import {TextField} from "@mui/material"
import {useState,useMemo} from "react"

export default function Browser({ name }: { name: string }) {
    const [text,setText] = useState<string>("")

    const isError = useMemo(() => text.length > 0, [text])


    return (
        <div>
        <h1>Hello {name}!</h1>
        <label>Input Now</label>
        <TextField
                error={isError}
                aria-label="input-field"
                value={text}
                onChange={(e) => setText(e.target.value)}
                helperText={isError ? "入力内容が不正です" : ""}
            />
        </div>
    )
}

```

### テストコード

次にテストコードを作成していきます。

```:typescript
import { describe, expect, test } from 'vitest'
import { screen } from "@testing-library/react"
import { render } from 'vitest-browser-react'
import { userEvent } from "@vitest/browser/context";
import Browser from './browser'

describe('Browser', () => {
  test('ページタイトルがrender出来ているか確認する', async () => {
    const { getByText } = render(<Browser name="Browser" />)
    await expect.element(getByText('Hello Browser!')).toBeInTheDocument()
  })
  test('インプット入力をし、エラーが出現するか確認する', async () =>{
    const {getByRole} = render(<Browser name="Browser" />)
    const input = getByRole("textbox")
    await userEvent.type(input, "a")
    await expect.element(screen.getByText("入力内容が不正です")).toBeInTheDocument()
  })
})
```

通常のVitestのテストでインポートしていたメソッドの一部がインポート先のライブラリが変わっています。

- vitest-browser-react
- @vitest/browser/context

ここらへんは、Browser Modeの初期設定時に追加された内容となります。
Browser Modeではブラウザ上でテストを行うため、いつも使っているtesting-Libraryなどを使うことができません。
基本的な記述はJestやVitestのテストのときと同じですが一部分違っています。

```
await expect.element(getByText('Hello Browser!')).toBeInTheDocument()
```

expectのelementメソッドが読み出されています。
これはBrowser Modeで呼び出しとなっています。
これがないとBrowser Mode上でテストが行われたと確認されないため、必ず設定が必要になります。
ただし、また型定義が揃っていないのでTypeScript上では型定義エラーになるので、回避するにはtesting-Library/preactの型定義ファイルが必要となります。
このテストコードでテストを実行してみましょう。

### 結果画面はこんな感じ

![テスト結果](/images/vitest-browser-test/testResult.png)

テスト結果がブラウザ上に表示され、どのような画面になったかまで把握することができます。
求めていたテスト実施がどのような状態かが視認することができ、なおかつコンポーネントテスト単位で実施できたので、目的を達成している機能だと思います！（やったぜ
問題があるとすれば、実行速度が早すぎるためテスト実施中がどのように動くかは視認することが難しい事。
複数のテストケースがある場合、最後に確認できるのは最後のテストのみ、とかでしょうか。

# まとめ

今回はVitest Browser Modeを試してみました。

## 試してみて解ったこと

やってみた感想としてVitest Browser Modeで良かったこと

- 目的のテスト実施が視認化できた
- Jest/Vitestの資産が流用できる事
- 環境構築がかなり楽で、コマンド1つでほぼ設定可能
 - jsdomみたいな仮想ブラウザの環境設定をする必要がない
- モノレポ構成でコンポーネントテストとロジックテストを完全に分離出来る事

でしょうか。
イマイチだなと思った点については

- 資産が流用できるが、ライブラリの書き換えで完全に流用は無理なので乗り換えコストはある
- モノレポの場合はファイル名や設定を変えるなどで、ディレクトリ設計を見直す必要がある
- そもそもまだ実験中

でした。
実験中のステータスの通り、まだ本番環境では利用ができないので使えない代物ではあります。
しかし、Playwright含め今後はコンポーネントテストはコンポーネントテスト専用な環境で、
ロジックテストはロジックテストの環境でと、より細分化されていく流れを感じ取る事ができたと思います。
なにより、Jestなどよりも環境構築が遥かに楽なのが好きでした。