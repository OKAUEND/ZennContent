---
title: "Next.js + TypeScript + Prisma + PlanetScale環境を構築する"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [nextjs, prisma, planetscale]
published: false
---

# はじめに

# Prisma とは

# PlanetScale とは

# 実際にやってみた

## 環境を構築

### Next.js の環境を構築

```
yarn create next-app --typescript
```

指示に従い、設定を行っていく。

### Prisma をインストール

```
yarn add -D prisma
```

prisma を導入する。

```
yarn add - D @prisma/client
```

設定した Prisma schema より自動的に型を作り出す PrismaClient を導入する

### PlanetScale の用意

サーバーレス DB である PlanetScale を利用するので、利用できるように諸々の用意を行う。

#### アカウント作成

https://auth.planetscale.com/sign-in

singup でアカウント登録
e-mail とパスワードで自分で登録するか、Github アカウントのどちらかで登録可能
(Github アカウントを用いて登録)
その後、github アカウントの登録メールアドレスに認証メールが届く。

![アカウント成功時の画面](/images/nextjs-prisma-planetscale/a6e3fbfa0cc5-20220820.png)

認証メールに届いたアドレスへ飛ぶと、このような画面になる。
初期はメールアドレスの名前になっているはずなので、use a different name?にて変更をすることが出来る。
その後、簡単な説明がいくつか続き、最後に create database でデータベースを作成する手順になるが、それは後ほど。

#### scoop を導入しておく

PlanetScale CLI を導入するには、Node.js のインストール手段ではインストールできず、
macOS なら`Homebrew `、Windows なら`Scoop`などのコマンドラインインストールツールが必要である。
今回、私は Windows 版を導入するので、事前に`Scoop`を導入する。

https://scoop.sh/

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
```

この 2 行を、順番に`powershell`に入力するだけで、端末にインストールされる。
そのため、事前に`powershell`を導入しておくこと。

#### PlanetScale CLI のインストール

```
scoop bucket add pscale https://github.com/planetscale/scoop-bucket.git
scoop install pscale
```

これも、同じように上から順に `powershell` に入力する。
成功したら CUI 上に`Success!`と表示される。

```
scoop update pscale
```

最新版にバージョンのアップデートをする。
これで、CLI の導入は完了。

## 設定

### Prisma の設定

公式のこのドキュメントを元に進める
https://planetscale.com/docs/tutorials/prisma-quickstart

まず最初にターミナルで prisma の基本設定をする

```
npx prisma init
```

成功するとこのような画面になる
![初期設定完了の通知](/images/nextjs-prisma-planetscale/a6e3fbfa0cc5-20220820.png)

### PlanetScale の設定

#### PlanetScaleCLI でログインを行う

PlanetScale CLI 上でログインをする。

```
pscale auth login
```

`powershell` にこのコマンドを入力をすると、ブラウザ上に認証画面が表示され、認証するとログインが完了。
~~なにこの技術それすごい~~

#### データベースを CUI から作成する

接続する前に、CUI から接続するデータベースを作成する。
:::message
無料プランの場合にて、すでに 1 つデータベースを作成していたら、2 個目は作成できないので削除をしておくこと。
:::

```
pscale db create star-app --region ap-northeast
```

このコマンドを入力すると、`star-app`という名前のデータベースが作成される。
コマンドの中身としては、

```
pscale db create XXXXX --region YYYYY
```

`XXXXX` の部分に、作成したいデータベース名を。
`YYYYY` の部分に、作成を行いたいリージョン先の名前を。
ちなみに、対応してるリージョンは[こ ↑ こ ↓](https://planetscale.com/docs/concepts/regions#available-regions)で確認できる。

テーブルの作成が完了したら、次にブランチを作成する。

```
pscale branch create star-app initial-setup
```

`star-app`に`initial-setup`のブランチを作成するとなる。
作成に成功すると、URL が表示され実際にサイトで確認してねと案内が出る。

### Prisma の Schema を設定する

実際に Prisma を用いて接続を行う前に、接続先の DB の種類や DB の URL などのセットアップ情報である Prisma Schema の設定を行う。
[Prisma Schema](https://www.prisma.io/docs/concepts/components/prisma-schema)

PlanetScale を使用するときの設定については、PlanetScale のドキュメントに設定内容が書かれている。
[ドキュメントの中段あたり](https://planetscale.com/docs/tutorials/prisma-quickstart)

```js
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
  referentialIntegrity = "prisma"
}

generator client {
  provider = "prisma-client-js"
  previewFeatures = ["referentialIntegrity"]
}

model Star {
  id              Int       @default(autoincrement()) @id
  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt
  name            String    @db.VarChar(255)
  constellation   String    @db.VarChar(255)
}
```

ファイルの拡張子は javascript の拡張子ではなく、`schema.prisma`といった専用のものとなる

#### datasource について

ここでは、接続先の情報を設定する

- `provider`
  で接続先の DB の種類となり、PlanetScale は mysql を使用しているので、mysql となる。
  他だと、`PostgreSQL`や`MongoDB`など。

- `url`
  そのなの通り、DB の URL。
  今回は環境変数に記載しているが、そのまま直接書き込む方法もある。
  　ちなみに、環境変数に書き込む方はこの通り。
  　`mysql://root@127.0.0.1:3309/<DATEBASE_NAME>`
  <DATEBASE_NAME>には、上の方で作成したデータベースの名前が入る。
  そのため、今回は、
  　`mysql://root@127.0.0.1:3309/star-app`
  となる

  - `referentialIntegrity`
    いわゆる参照整合性のこと。
    設定するかは任意ではあるが、PlanetScale などの外部キーがサポートされていない場合には、参照整合性の設定が必須となる。

#### generator client について

- `provider`
  Prisma Client で使用する Provider を指定する。
  ただ、Client を使用する場合は`prisma-client-js`を指定する。

- `previewFeatures`
  機能が Previews で提供してるものを指定する項目。
  今回の場合は、PlanetScale のドキュメント通り、参照整合性の`referentialIntegrity`を指定する。
