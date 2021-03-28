---
title: "Firestoreのリファレンス型を扱おう"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["firebase", "fireastore", "javascript", "typescript"]
published: false
---

# はじめに

SQL のリレーションを Firestore でもしたかった
そこで調べてみた

# Firestore とは

そもそも Firestore とは
mBaaS の一つで Google がサービス展開を行っている
NoSQL の Firestore や Hosting や Function 等の機能があり、複雑な設定や組み合わせなどが不要で
素早くサービスの開発を行えたりするのが特徴

## Refarence 型とは

Firestore では、リレーションなどはサブドキュメントを使い、データの紐付けを行っていくが、
多階層になったり、1 つのドキュメントがサブドキュメントとして複数存在した場合に、管理が大変になる
その場合に、Refarence 型とよばれる Firestore の機能を使い、サブドキュメントなどの圧縮が行える

## 変数に代入する方法(Firestore 含め)

Refarence 型の作り方と、DB に入れ方

## 使い方

# おわりに
