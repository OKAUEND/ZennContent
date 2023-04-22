---
title: "ESLintなんもわからんから調べた"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react, vite, eslint]
published: false
---

## ESLint なんもわからん

フロントエンドでほぼほぼデフォルトスタンダードで使われてる ESLint。
とりあえず導入して ESLint は使ってるけど、設定とかなんもわからんので整理してみることに。
~~ルールもそうだけどいろんなプラグインの組み合わせが魔境すぎるねん~~

## ESLint ってそもそもなんなん

https://eslint.org/
ESLint とは静的なコード検証ツールの一つで、JavaScript 環境では最も使われてるツールの一つ。
個人開発でもチーム開発でも重要なコーディング規約違反やバグの発見などをレビューで~~目 grep~~探したり指摘したりせず、
ツールに任せる事で人によるブレをなくし、一定のコード品質を維持するために必要、ということらしい。

つまり、バグが起きそうな部分のチェックやコードスタイルを統一することで、作業中に危険性に気づき作業の品質を向上させ、
他人が書いたコードでもコードスタイルは同じなのでコードを読む時でも~~古代文字解析~~理解のための時間を減らすという効果を期待するためってこと。
最近だと、CI/CD や Github とかのソースコード管理ツールへ Commit する時に、自動発動して行うなどもあるため、
~~めんどくさい事~~チェック作業や修正作業や品質管理はツールに任せて、作業者自身はよりプロダクトの生産へ注力するって効果もあるのかも。

### ほなつかうで

ということで早速導入して実際のファイルを確認してみましょう。
今回はフロントエンドで使うことを想定してるので、`React`環境でどうなってるかを確認します。

```
yarn create vite
```

~~なにさらっと Vite 環境で構築しとんねん~~
https://vitejs.dev/guide/
Vite は Node.js が一定以上のバージョンだと使えるため、特に事前にインストールが必要なのは Node.js 以外はない…はず。
すると、いくつか対話形式でどうするかを確認された後に、任意のフォルダに React+Vite のプロダクトが生成されます。
今回は途中の選択肢で`React`と`TypeScript`を選択しました。

`package.json`を覗いてみると、すでに`eslint`の本体と React と TypeScript の ESLint のプラグクインが初期で導入されています。
`package.json`と同じ階層に`.eslintrc.cjs`が生成されていますが、これが`ESLint`の設定ファイルになります。
ちなみに、`package.json`内にも設定をかけますが、管理しづらかったり見づらかったりで分けたほうがいいというのが個人の感想。
`.eslintrc.cjs`の`*.cjs`は CommonJS のファイルで、他には通常の JavaScrip 形式や YAML や JSON でも設定ファイルを作れます。

```
.eslintrc.js or .eslintrc.yml or .eslintrc.json
```

### 設定ファイルの中複雑すぎやろ

```json
{
  "env": { "browser": true, "es2020": true },
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react-hooks/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": { "ecmaVersion": "latest", "sourceType": "module" },
  "plugins": ["react-refresh"],
  "rules": {
    "react-refresh/only-export-components": "warn"
  }
}
```

最初からある程度記述されてますね。
普段使ってたりするのである程度ざっくりは認識してたりしますが、ここでちゃんと確認してみます。
(~~この時点でもうすでに種類多いねん~~)

#### env

https://eslint.org/docs/latest/use/configure/language-options#specifying-environments
env ってことなので ESLint を動かす上での利用する環境での設定を指定します。
何故必要かといえば、ESLint からしたらブラウザ環境や ECMAScript の設定などは知らないので使おうとすると、ESLint がなんじゃこりゃ！って起こってくるからですね。
Node 環境で動かす場合とかでも、追加に指定が必要なので、必須項目に近いかな？

#### extends

エンジニアをしていたらよく見る単語。
つまり設定の上書きにしての設定の変更や追加を行う項目になります。
extends の仕様は、下に記載されているものが上のの重複している設定を上書きしていきますので、

```json
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react-hooks/recommended"
  ],
```

`eslint:recommended`の設定をそれぞれ`plugin:@typescript-eslint/recommended`が上書きし、さらにそれらを`plugin:react-hooks/recommended`が上書きします。
ただし、重複している部分のみなので、それぞれ重複しない部分の設定は反映されるので、前の設定がまったく上書きされて消されるとかではないです。
後述する plugin を導入した時は、かならずこちらにも記載しましょう。

#### parser

https://eslint.org/docs/latest/use/configure/parser

JavaScript のコードを ESLint が評価するために構文解析を行う方法やモジュールなどを設定する箇所です。
デフォルトでは ECMAScript パーサーである Espree になっています。
TypeScript 環境だった場合には、TypeScript を構文解析する必要があるため、変更する必要があるというわけですね。

#### parserOptions

https://eslint.org/docs/latest/use/configure/language-options#specifying-parser-options
ここは ESLint がサポートする言語設定を行う項目になります。
つまり解析したい構文とかが ECMAScript2020 に合った場合には、それを設定しないと対応しないってことかなと。
この項目自体はオブジェクトの入れ子構造なので、さらにネストして設定する必要があります。

- ecmaVersion : ECMAScript のサポートバージョン
- sourceType : script/module
- allowReserved : 予約語を識別子として使用できるようにする
- ecmaFeatures : さらに追加で使用したい言語機能の追加

parserOptions の設定詳細はさらにこの 4 つになり、必要な場合に設定しなければいけません。(なければデフォルト値を使用)
主に設定するのは`ecmaVersion`,`sourceType`,`ecmaFeatures`の 3 つかと思います。
特に import/export のモジュール形式を使いたい場合は必ず`sourceType`を`module`にしなければならず、
React などで`JSX`を使いたい場合は`ecmaFeatures`をさらにネストし`jsx`プロパティを有効にする必要があります。

```json
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module",
    "ecmaFeatures": { "jsx": true }
  },
```

つまりこうなります。

#### plugins

https://eslint.org/docs/latest/use/configure/plugins

プラグインを導入して ESLint を拡張する場合は、この欄で指定します。

```json
  "plugins": ["react-refresh"],
```

この場合は、`eslint-plugin-react-refresh`のプラグインを指定してる意味になります。
`eslint-plugin-`は省略できるため、プリフィクスを除いた部分の`react-refresh`のみを指定するだけでよくなります。
プリフィクスを含めていても機能するため、フルネームを指定しても問題はありません。

この部分で指定した名前が、そのまま`rules`や`env`どころか、`extends`でも使えるようになるので気をつけるポイントかと。

https://blog.ojisan.io/eslint-plugin-and-extend/
複雑なので、上記の記事も参考にしてください。

#### rules

https://eslint.org/docs/latest/use/configure/rules
静的解析でどうするかをこの欄に記載していきます。
ただし、plugin を入れてる場合には無記入でも plugin 側で設定されているのを、`extends`や`plugin`で導入しているため、設定は不要となっている場合が多いです。

## そもそも Lint のルールはどうするん？

### ルールの数多すぎやろ

### 設定のプラグインってなんや

### トレンドを見てみるで

### 結局どうすればいいんや

## でもそろそろ変わるみたいやで
