---
title: "Playwright UI mode を使いコンポーネントテストのテスト動作を視覚的に確認できるかを試してみた"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, playwright]
published: false
---

## 1. はじめに

近年、フロントエンド開発においてコンポーネントテストの重要性が高まっています。私が携わっているプロダクトでも、QAチームの負荷軽減と品質向上を目的として、フロントエンドの単体テストおよび統合テストを正式に導入することになりました。
それまで個人で自由に記述していたテストを、より組織全体で実施する方針に移行したのです。
しかし、コンポーネントテストを本格的に書いていく中で、いくつかの個人的な課題に直面しました。
特に大きな問題となったのは、テストの動作を視覚的に確認できないことによる不安です。具体的にはプロダクトでテストを記述している中で問題がありました.

- テストコードで対象の要素が取得できず、テストを実行できない場合がある
- 要素は取得できているものの、本当に意図通りに動作しているのか自信が持てない
- 偽陽性などが発生しているのではないかという不安

これらの問題を解決する方法を探る中で、Microsoftが提供しているテストライブラリのPlaywrightで、UI modeを使用すると、実際にテスト中のステップ毎に疑似ブラウザ上で動作を確認できるといった特徴があります。
また、直近で実験中にはなりますが、Playwrightでコンポーネントテストを行える[Playwright Components](https://playwright.dev/docs/test-components)もあるので、
この2つのPlaywrightの機能を使用して単体テスト・統合テスト単位でコンポーネントテストの動作を視覚的に確認できるかどうかを試してみることにしました。

### Playwright Components とは

[Playwright Components](https://playwright.dev/docs/test-components)

Playwrightをコンポーネントテストで、単体テストや統合テスト単位で使えるようになる機能です。
Playwright自体はe2eテストツールであるため、これを使おうとするとかなりFatなツールでした。

```
npm init playwright@latest -- --ct
```

こちらで導入でき、後は設定ファイルが現段階では通常のPlaywrightとは違う、`playwright-ct.config.ts`と別物になります。
現段階では実験中な物のため今後変更がはいる可能性も高いですし、プロジェクト自体がクローズになる可能性があるため現状の情報が古くなる可能性があります。

## 2. コンポーネントテストをやってみる

### まずはPlaywright Componentsでコンポーネントテストを作る

 ``` 
import {TextField} from "@mui/material"
import {useState,useMemo} from "react"

export const Mui = () =>{
    const [text,setText] = useState<string>("")

    const isError = useMemo(() => text.length > 0, [text])


    return (
        <div>
            <h1>Material UI</h1>
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

今回は試しにこのようなコンポーネントを作成をし、実際にコンポーネントテストを実装してみましょう。
Input要素はMaterial UIのTextFiledを今回は利用してみました。
作成したコードは[こちら](https://github.com/OKAUEND/react-playwright-ui)

```
import { test, expect } from '@playwright/experimental-ct-react';
import { Mui } from './mui';

test('Mui', async ({ mount }) => {
  const components = await mount(<Mui />);
  await expect(components).toContainText('Material UI');
});
```
試しに作ったコンポーネントの要素が表示出来るかどうかの単純なテストとなります。
これで以下のコマンドを入力することでテストを実行してみましょう。

```
npm run test-ct
```

Passedするので成功となります。
では次にInput要素の確認をしてみたいので、そちらのテストを作成します。

```
test("MuiにKey入力を行う",async({mount })=>{
  const component = await mount(<Mui />);

  // 初期状態の確認
  const textField = component.getByRole("textbox");
  await expect(textField).toHaveValue('');
  await expect(textField).not.toHaveAttribute('aria-invalid', 'true');
  await expect(component.getByText('入力内容が不正です')).not.toBeVisible();

  // キー入力
  await textField.press('a');
  await textField.press('b');
  await textField.press('c');
  await textField.press('d');
  await textField.press('3');

  // 入力後の状態を確認
  await expect(textField).toHaveValue('abcd3');
  await expect(textField).toHaveAttribute('aria-invalid', 'true');
  await expect(component.getByText('入力内容が不正です')).toBeVisible();

  // テキストを削除
  await textField.fill('');

  // 削除後の状態を確認
  await expect(textField).toHaveValue('');
  await expect(textField).not.toHaveAttribute('aria-invalid', 'true');
  await expect(component.getByText('入力内容が不正です')).not.toBeVisible();
})

```

出来たらコマンドを入力して確認すると成功しました。

## 3. Playwright UI modeを試してみる

コンポーネントテストができたので、このコンポーネントテストを使ってUI modeで確認出来るかを試してみましょう。
…と言いたいところですが、まず結論としてはうまくいきませんでした。

UI modeはE2E側のPlaywrightの機能なため、Playwright Components側ではまだ機能がないためです。
E2E側であるため、テストを行うにはテスト対象のページにアクセスする必要があるため、
そのページが存在しないnode上で行われる単体テストや統合テストは組み合わせが現状できないといった状態です。

もしかしたら、将来的にはPlaywright ComponentsにもUI modeが実装されて確認出来るようになるかもしれませんが、今は無理ぽって感じでした。

#### Playwright ComponentsとStorybook
[Portable stories for Playwright Component Tests](https://storybook.js.org/blog/portable-stories-for-playwright-ct/)
こちらの方でPlaywright ComponentsとStorybookを紐づけて利用する方法が記載されています。

`*.stories.tsx`などを使うために、`*.stories.portable.ts`で、composeStories APIを呼び出し定義してあげる必要性があります。
同テストファイル内で呼び出しが不可能であるため、ファイルを切り分けてAPIの呼び出しがあるのでひと手間かかる現状です。
この手順自体は課題とされている部分なので、将来的には不要になる可能性が高めかもしれません。

## 4. コンポーネントテストの視認性についてはどうなのか

UI modeで確認しながらはできないとしたらどうしたらよいのかという点が残ります。
これに関しては、目的を達成出来るテストコードを書き、経験則で良し悪しが解るようになるが今の答えなのかもしれません。

### 目的に沿った関数やメソッドを使う

基本機能だけでテストが出来そうに感じますが、それだけではテストが上手くいかないので目的を達成出来る機能を使うのが重要だと思います。
例えばモーダル画面を開く場合や、画面内で要素が増えたり消えたりする場合などは`waitForElementToBeRemoved`を使う事で要素が消えるまで待機してくれたりなど。

Playwright Componentsでキー入力をする場合は、PlaywrightのE2E版とも違うので注意が必要でした。
```
  // キー入力
  await textField.press('a');
  await textField.press('b');
  await textField.press('c');
  await textField.press('d');
  await textField.press('3');
```

上記のテストでこのように一つづつ入力していましたが、これは実際にまとめて入力することではなくユーザーの一文字づつ入力するのを再現してるような挙動でした。
PlaywrightのE2E版の方では、(こちら)[https://playwright.dev/docs/api/class-keyboard]を確認するとわかりますが、少し違う感じです。

```
await page.keyboard.type('Hello World!');
await page.keyboard.press('ArrowLeft');
```

このようにtypeで一括で入れれる事ができたり、pressでも一括入力が可能です。
これに関しては、単純に私がPlaywright Componentsの方で組んだコードでは、対象がコンポーネントに対して、
PlaywrightのE2E版のはpageに対してなので、pageとcomponent(mount)の挙動の違いかもしれません。

## おわりに

今回調べた結果の答えとしては、適した関数やメソッドを使い、ライブラリ毎の挙動の癖を把握し後は経験則で頑張るみたいな、少し根性論が入った部分がありました。
適した関数やメソッドに関してはネットの情報でも探しきれない事が多いので、ChatGPTのようなAIに質問をし情報の再確認を行いつつやるのが一番よい開発体験かなと思いました。
（実際に`waitForElementToBeRemoved`はAI君に教えてもらった関数）

不安に思うのは知らないから適切に行うことができないので実装がうまくいっているか自信がない、に起因するのが原因だと思うので、
AIなどで不安な部分を徹底的にどうすればいいかを聞いていき、後はPRのレビューなどで先輩方のノウハウを吸収していくっていうのが、今回の答えかなと思います。

もしかしたら、数年後にはPlaywright ComponentsでUI modeが実装される可能性があるので、その時は視認しながら出来ると思います。
ただ、その時でも結局は最適な機能や選択を使うことが重要かなと。