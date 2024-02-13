---
title: "StoryBookのAI生成アドオンを検証してみる"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react","storybook","openai"]
published: false
---

# AI 時代

Github Copilot で開発体験が向上し、なかった時代に戻れない体になってしまいました。
予測候補でどのようなコードが想定されるかを表示してくれるのは、使わなかったとしてもどの方向性に目指すべきか指標とはなります。
その中で、StoryBook 向けの AI 生成のライブラリを見つけたので、試してみようと思います。

# StoryBook Genie

https://github.com/eduardconstantin/storybook-genie

StoryBookGenie は、OpenAI を利用した AI 生成のライブラリです。

```
npx storybook-genie
```

CLI を入力すると、候補のコンポーネントが CLI 上に表示され、選択をすると`StoryBook`のファイルが対象ファイルの階層に生成されます。
使い方としては非常にシンプルですので、使い方の記事ではなく、複数のケースでどのようなファイルが生成されるかを試してみます。

# 検証とは何するの？

フロントエンドでよく使われるパターンをいくつか想定してコンポーネントと Hook を作成してみました。
これらの対象に対して、どのような StoryBook のファイルが生成されるかを試してみます。
また、StoryBook-Genie は OpenAI を利用しているため、使った Token 分だけ費用がかかります。
ですので、対象に対してどれぐらいの Token と費用がかかるかもチェックをしてみたいと思います。

### 期待している事

そこで、今回は各ケースを試すときに出来てほしい事を箇条書きにして、それと比較してどれぐらいなのかを確認したいと思います。

※例

- ファイルが生成されている
- 使用 Token 数がどれぐらいでいてほしいか
- StoryBook の使用方法を筆者のレベルより上のを出力できているか

## Case1

構成としては非常にシンプルで、タイトルと 1 行程度の文章があるコンポーネントです。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case1/CaseOne.tsx

### 確認項目

- Token の使用数の最小数を把握
- 作成される StoryBook の状態を把握

では試してみましょう。

## Case2

ケース 1 のコンポーネントよりもテキスト内容が多いコンポーネントです。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case2/CaseTwo.tsx

### 確認項目

- ケース 1 と比較して Token 数がどれぐらい増えるか

(ちなみに、文章は Google の Gemini で生成したのを使用しています)

では試してみましょう。

## Case3

ケース 2 の内容を親から Props で受け取り表示をする方法にしたコンポーネントです。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case3/CaseThree.tsx

### 確認項目

- Props は認識しているか
- StoryBook の args にダミーデータを作成するか

では試してみましょう。

## Case4

入力をしボタンをクリックしたら入力内容が画面へリスト表示されるロジックを持つコンポーネントです。
ロジックは Hook に、useState で値を持つ React の基本みたいな構成にしてみました。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case4/CaseFour.tsx

### 確認項目

- Hook は認識するのか
- Hook を持つコンポーネントはどういうファイルが生成されるのか
- ロジックがあるコンポーネントの場合 Token 数がどれぐらいか

では試してみましょう。

## Case5

API 通信が行われるロジックを持つコンポーネントです。
今回の API 通信は TanstackQuery を利用してみました。

https://github.com/OKAUEND/storybook-genie/blob/main/src/feature/Case4/CaseFour.tsx

なお、今回は事前に msw や storybook の msw アドオンを導入してあります。

### 確認項目

- API 通信を MSW で Mock してくれるか
- StoryBook 上に msw の項目を追加してくれるか

では試してみましょう。

## 振り返り

# あとがき
