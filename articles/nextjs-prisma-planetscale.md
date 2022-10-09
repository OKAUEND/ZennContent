---
title: "Next.js + TypeScript + Prisma + PlanetScale環境を構築する"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [next.js, prisma, planetscale]
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

!(Planet Scale Singin)[https://auth.planetscale.com/sign-in]

singup でアカウント登録
e-mail とパスワードで自分で登録するか、Github アカウントのどちらかで登録可能
(Github アカウントを用いて登録)
その後、github アカウントの登録メールアドレスに認証メールが届く。

![アカウント成功時の画面](/images/nextjs-prisma-planetscale/a6e3fbfa0cc5-20220820.png)

認証メールに届いたアドレスへ飛ぶと、このような画面になる。
初期はメールアドレスの名前になっているはずなので、use a different name?にて変更をすることが出来る。
その後、簡単な説明がいくつか続き、最後に create database でデータベースを作成する手順になるが、それは後ほど。

## 設定

### Prisma の設定

### PlanetScale の設定
