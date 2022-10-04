---
title: "Firestoreのリファレンス型を扱おう"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["firebase", "fireastore", "javascript", "typescript"]
published: false
---

# はじめに

Firestore を使っての開発を行っているときに、リレーションでデータを紐付けさせたくなる場面が出てきたので、
Firestore でリレーションに近い機能がないかを調べてみました。

# Firebase と Firestore

そもそも Firestore とは mBaaS の一つで Google が開発とサービス展開を行ている。
AWS や Azure とかの他の BaaS との違いは、複雑な設定や組み合わせなどが不要で素早くサービスの開発を行えたりするのが特徴。
Firestore,Hosting,Cloud Function,Storage などがあります。

Firestore は NoSQL データベース。
使用方法は Firebase 公式が提供している、ネイティブ SDK を使うか、REST API の形でアクセスをするなどの方法がある。

## Refarence 型とは

多階層になったり、1 つのドキュメントがサブドキュメントとして複数存在した場合に、管理が大変になる
その場合に、Refarence 型とよばれる Firestore の機能を使い、サブドキュメントなどの圧縮が行える

## 変数に代入する方法(Firestore 含め)

Refarence 型の作り方と、DB に入れ方

## 使い方

Firestore より取得したドキュメントのキーに対して get を行うだけで利用可能

# おわりに
