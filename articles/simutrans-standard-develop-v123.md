---
title: "フリーゲーム:Simutransの開発環境を整える(Standard版)"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp", "Simutrans"]
published: false
---

# はじめに

# Simutrans について

# 開発環境構築

## 必要ツールを揃える

### Microsoft Visual Studio

https://visualstudio.microsoft.com/ja/

公式より`Visual Studio`を入手するために、Microsoft の公式よりダウンロードをします。
Simutrans は`C++`で作られており、また公式が推奨してる開発環境も`Visual Studio`なため。
`Community`、`Professional`、`Enterprise`の 3 つがありますが、後者 2 つは有料版なので`Community`をダウンロードします。
インストール時には、C++によるデスクトップ開発のみを選択します。

![VisualStudioのインストール時にC++のみを選択する](/images/nextjs-prisma-planetscale/a6e3fbfa0cc5-20220820.png)
:::
他に試したい機能があれば選ぶのもありですが、正直かなり容量を食います。
:::

インストール場所などは、各自の任意の場所へ。

### Git

https://git-scm.com/
ライブラリやソースコードを入手するために、github にあるのをクローンで入手する必要があるため、
Git を導入するため公式よりダウンロードをします。

### vcpkg(※要 Git)

https://github.com/microsoft/vcpkg

以前ですと、ライブラリを入手するために別途 DL していっていましたが、最近のは Microsoft が適用してる
クロスプラットフォームパッケージより入手します。

### Simutrans 本体と Pak を入手

## 構築を始める

### ソースコードを入手する

### vcpkg から必要ライブラリをインストール
