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

## テストコードを書いてみる

### 今回のテストコンポーネント

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
    expect(screen.getByText("入力内容が不正です")).toBeInTheDocument()
  })
})
```

### 結果画面はこんな感じ

## 試してみて解ったこと

### Vitestの通常版との関係性

# まとめ