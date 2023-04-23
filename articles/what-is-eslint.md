---
title: "ESLintなんもわからんから調べた"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [eslint]
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

```json
  "rules": {
    "react-refresh/only-export-components": "warn"
  }
```

記述方法自体は、rules のオブジェクトにどんどん追加していくって方法になります。
予測変換機能もあるみたいですが、plugin とかで追加したのは出ないので、もとより ESLint が設定としてあるのだけなのかもしれません。（もしかしたらおま環

## そもそも Lint のルールはどうするん？

ルールは、Rule Severities(重大度)によって機能するかオフにするかをルール付けできます。

- "off" or 0
- "warn" or 1
- "error" or 2

`off`はその名の通りチェックをスルー。
`warn`は注意表示はさせるで留める。
`error`はエラーさせて警告が発生します。

CI/CD などと組み合わせると、おそらく`error`が出てる状態だとデプロイなどを行わないなどが出来るのかなと(筆者は未経験)

ちなみに、ルールの中にはオプションで更に指定できるのもあります。
https://eslint.org/docs/latest/rules/no-else-return

```json
'no-else-return': ["error",
    {
        "allowElseIf": false,
    },
],
```

このように、2 つ目にオブジェクト形式で指定することで、オプション指定でさらに細かくルール付けをすることができます。
なので、設定するルールなどは一度公式ドキュメントで確認するなどすると、細部までルール付けで制約をかけることができるかと。

### ルールの数多すぎやろ

デフォルトでもある程度のチェックはしてくれますが、より詳細にするには自ら設定していく必要があります。
が！！！！
そもそものルール数が 200 個以上もあるので、自分でどれが必要なのかを精査して設定していくと 1 日潰れちゃいます。
さらにオプションの有無なんて確認してたらいつ開発ができるのかとか以前に、開発環境を整えるまでで物凄い労力になってしまう。~~やべぇぞ！~~

https://qiita.com/khsk/items/0f200fc3a4a3542efa90

ということで、世界の出来る兄貴達が作ってくれている OSS の ESLint のルールプラグインを導入しましょう

### 設定のプラグインってなんや

200 個以上もある設定を OSS で設定してくれているライブラリの事。
とはいえ！！！
このライブラリ自体も世の中に数百あるので、よさそうなのを探して導入するも相当難しいわけで。
なので、こういう場合は世の中で最もよく使われているのを導入してみるのがいいかと思われます。

- eslint-config-airbnb
- eslint-config-standard
- eslint-config-google

有名所の主要なライブラリはこの 3 つだと思われる。

https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb
`airbnb`は airbnb 社が公開してる設定ライブラリで、他の 2 つに比べて特に制約が厳しい事で有名。

https://github.com/standard/eslint-config-standard
`standard`はシンプルな設定が行われているライブラリです、ただしこのライブラリのルールの 1 つとしては、任意の設定を都合で変更してはいけないということになります。

https://github.com/google/eslint-config-google
`google`は Google 社が公開している設定ライブラリです。他の 2 つよりかは情報が少ない印象が…。

この 3 つの設定の違いは以下の記事で詳しく載っています。

https://zenn.dev/tapioca/articles/5685d794f6452b

#### react や vue や next 向けの

他に、

- eslint-config-react
- eslint-config-vue
- eslint-config-next

などもありますが、これらはスタイルの一括設定ライブラリというよりかは、それらが提供している固有の構文などを精査できるようにするライブラリ。
なので、React を使う場合は`eslint-config-react`を使わないとエラー吐きまくりますし、`TypeScript`を使う場合は専用のをいれないと ESLint 君が激おこします。
必ずいれましょう

### トレンドを見てみるで

で、結局どれを使えばいいやってところで、各ライブラリの使用数とかのトレンドを見て確認してみましょう。

![ライブラリのトレンド](/images/what-is-eslint/152642b5f748193d0a1c259027597bc1.png)

圧倒的 airbnb
他の 2 つと大きく引き離してますね…。

![eslintとライブラリのトレンド](/images/what-is-eslint/ce7f312d0a8952b97a1a8dcb116a9388.png)

eslint とライブラリの使用率をみると…(トレンド君の仕様なのか追加したら項目の色が変わりました)
eslint は導入するけどライブラリを使わないって感じのプロジェクトが多いのが読み取れます。
まぁ、個人開発やテストプロダクトで導入する意味がないって例もあると考えられますが、半分ぐらいがそうだとしても結構突き放してますね。

### 結局どうすればいいんや

eslint を導入するのは必須。
では、設定はどうするかと言えば自前で一から設定するのなら`airbnb`を使ったほうがいいと思います。
ただし、会社ごとのルールを決め、そのライブラリを会社で使うとするのなら、自前で設定して使い回せるようにしたほうがいいと思います。

どのみちですが、プロダクト作成毎にルールを精査して決めるのは時間の無駄なのでまじでやめましょう。
個人開発なら素直に`airbnb`などの公開されてる設定ライブラリを使おうな、~~約束だぞ~~

## でもそろそろ変わるみたいやで

ここまで長々と調べてきましたが、実はこの設定もうすぐ変わるっていう悲しいお知らせがあるんですよね。
それが FlatConfig。

https://eslint.org/blog/2022/08/new-config-system-part-1/

つよつよエンジニアの先輩方の記事。
https://zenn.dev/makotot/articles/0d9184f3dde858
https://zenn.dev/babel/articles/eslint-flat-config-for-babel

何故変わるかといえば、純粋に今の設定ファイルが複雑怪奇になっちゃったからですね。
というのも、今では様々なプラグインやライブラリを導入するのが当たり前になり、それの`extends`だったり`plugin`を設定したりパーサーをなんとかしたりと
上書き変更追加が複雑に絡み合って、ルール以外の設定をするだけでもなんじゃこりゃナンジャモになってます。
ESLint の環境をライブラリ導入だけでを整えるのに 1 日かかったりすることは皆さんも経験あるんじゃないでしょうか
(~~私はあります~~)

なので、それらを解消してシンプルにしようぜ！てのが FlatConfig。
現在実験段階でまだ正式採用には至ってませんが、おそらく近い将来導入されるのは確実な状況です。
(おそらく次のメインバージョンがあがったらとかの噂はあるみたい)

その時にこの記事は無用の長物にはなると思いますが、そもそも移行するためには既存の設定を読み解く必要があるので、そういう需要はあるかなと…。
(未来の自分向けでもある)

## 終わりに

導入は必須ですが、設定が超めんどくさい ESLint。
今回はその設定やそもそもの設定の機能自体を確認してみました。
そして、私は記事を書くことで ESLint の設定ファイル自体をより深く理解することができました。
皆さんも是非 ESLint の設定ファイルなどを見返してみましょう！
