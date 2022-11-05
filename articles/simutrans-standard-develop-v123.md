---
title: "フリーゲーム:Simutransの開発環境を整える(Standard版)"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp", "Simutrans"]
published: false
---

# はじめに

フリーゲームの Simutrans の開発環境を構築してみましたので、今回はその手順を記事にしてみました。

# Simutrans について

Simutrans とは、オープンソースで開発・公開がされている経営シュミレーションゲームです。
1 企業となり鉄道・自動車・船舶・航空機をなどを使い、乗客と貨物を目的地へ運び利益を出しながら都市を発展させていく内容となっています。

新機能の追加やバグ修正などは、有志のプログラマーが公式フォーラムを通じて開発を行っています。
今回は、その Simutrans の開発環境を構築していきましょう。

# 必要ツールを揃える

## Microsoft Visual Studio

https://visualstudio.microsoft.com/ja/

公式より`Visual Studio`を入手するために、Microsoft の公式よりダウンロードをします。
Simutrans は`C++`で作られており、また公式が推奨してる開発環境も`Visual Studio`なため。
`Community`、`Professional`、`Enterprise`の 3 つがありますが、後者 2 つは有料版なので`Community`をダウンロードします。
インストール時には、C++によるデスクトップ開発のみを選択します。

![VisualStudioのインストール時にC++のみを選択する](/images/simutrans-standard/b43295599e701ff9e259e737be52301e.png)
:::message
他に試したい機能があれば選ぶのもありですが、正直かなり容量を食います。
:::

インストール場所などは、各自の任意の場所へ。

## Git

https://git-scm.com/
ライブラリやソースコードを入手するために、github にあるのをクローンで入手する必要があるため、
Git を導入するため公式よりダウンロードをします。
公式サイトの Download for Windows より実行ファイルをダウンロードして、インストールを行ってください。

## vcpkg(※要 Git)

https://github.com/microsoft/vcpkg

以前ですと、ライブラリを入手するために別途 DL してビルドしていましたが、最近のは Microsoft が提供してる
クロスプラットフォームパッケージより入手します。

Code の部分より直接ダウンロードをし任意のフォルダに解凍するか、git の clone で入手するかの 2 つの手段があります。
今回は、git の clone を使ってやってみましょう。

まずは`コマンドプロンプト`を起動します。
次にパッケージを保存したいフォルダの位置へ、`cd`のコマンドを使い階層を移動します。
:::message
もし、別のドライブに移動したい場合は`D:`や`E:`などでドライブを指定する必要があります。
:::

![Simutransフォルダへ階層を移動する](/images/simutrans-standard/ba30b667049527d89784116c6e23874c.png)

今回は、simutrans のフォルダ階層内に`build`という開発ファイルを格納するフォルダを作り、そこへダウンロードを行います。
なので、`mkdir`コマンドを使い、フォルダを作成します。

```
mkdir simutrans
mkdir build
```

そして、cd コマンドを使い作成した build フォルダへ移動します。

```
cd build
```

```
<USER_DRIVE>:<USER_PATH>/simutrans/build
```

ドライブより下のパスがこのようになっていたら準備は完了です。

次に`git clone`コマンドでダウンロードを行いますので、github の`code`ウィンドウにある、
`https...`から始まる URL をコピーし、以下のコマンドを入力します。

```
git clone https://github.com/microsoft/vcpkg.git
```

git のインストールがうまくいっていれば、パッケージのクローニングが始まります。

![GitCloning](/images/simutrans-standard/af2e5116cc110484a257f418f38f780e.png)

## Simutrans の Pak とソースコード を入手

Simutrans は実行ファイルなどだけではプレイすることができず、モデルやゲーム内容などの数値が設定されている`Pak`が必要となります。
:::message
pak64 や pak128、pak128jp など、内容によって種類が結構存在します。
:::
開発やデバッグにもゲームをプレイするために`Pak`が必要なため、`Pak`を入手する必要があります。

今回は Wa 氏が作成されている、日本風景感の`Pak`である、`pak.nippon`を導入しましょう。
https://github.com/wa-st/pak-nippon/releases/tag/v0.6.1

ダウンロードしたファイルを解凍し、中の`pak.nippon`フォルダ毎任意の場所に保存しておきましょう。
後ほど使います。

次に、Simutrans のソースコードを入手します。

https://github.com/simutrans/simutrans

今回もソースコードが github にあるので、git clone で入手します。
clone 先は、一個前の項目である vcpkg で作成した`build`内にします。

build フォルダ内に、`simutrans`のフォルダが新しく作られ、中に`GDI`や`SDI`とか`src`と書かれたフォルダがあるなら成功です。

# 構築を始める

必要なアプリやライブラリなどを用意できたので、これらを設定していき開発及びデバッグを行えるようにします。

## vcpkg から必要ライブラリをインストール

vcpkg から必要なライブラリをインストールするために、bat ファイルからライブラリのインストール処理を行います。
用意したソースコード内の`tools`フォルダ内にある、`install-building-libs-<PROCESS>.bat`を、`vcpkg`のフォルダへコピペします。

:::message
32 ビットか 64 ビットどちらでデバッグを行うかで、x84(32 ビット版)か x64(64 ビット版)を選んでください。
デバッグを行う時に、デバッグを行うプロセスに合うライブラリが導入されていないと失敗になります。
:::

コピペが完了したら、次はコマンドプロンプトでまずは`vcpkg`同梱の bat ファイルを起動します。

```
<PARH>/bootstrap-vcpkg.bat
```

![vcpkg Install](/images/simutrans-standard/1714f7c07537ad43d6fff1cc30e3ef00.png)

この同梱の bat ファイルの処理が終わったら、次は Simutrans の bat ファイルを起動します。
先に vcpkg のバッチファイルを処理していないと、この後の処理ができないので、必ずやっておきましょう。

```
<PARH>/install-building-libs-x64.bat
```

今回は 64 ビット版を導入するので x64 の方を使用します。
すると、長い処理が始まるまで終わるまで待機します。
最後に、`続行するには何かキーを押してください . . .`が表示されたら処理が完了です。
これで、必要なライブラリを導入することができました。

## Visual Studio でソースコードを確認する

git Clone しておいたソースコードを VisualStudio で展開します。
VisualStudio を起動したらこのようなウィンドウが表示されるはずなので、今回はローカルフォルダーを開くを選択します。
なお、一番上のリポジトリのクローンを選ぶと、git clone と同じ事をやってくれます。
便利ー！

![VisualStudioの開始画面](/images/simutrans-standard/1bb11555aab814d7780b9ecd93d40a9e.png)

選択するフォルダは`Simutrans.sln`のソリューション拡張子のファイルがあるフォルダを選択します。
すると、VisualStudio の画面が画像のようになります。

![ファイル一覧](/images/simutrans-standard/882c1db948a2dc7ad5b469812e9f3d78.png)

この状態でも、`src`フォルダの中に simutrans 本体のソースコードがあるので、コーディングを行うことができますが、
デバッグをするには今のフォルダのパス配下から、`Simutrans.sln`パス配下へ移る必要があります。
それには、`Simutrans.sln`をダブルクリックすることで移動することができます。

移動すると画像のようなツリーが表示されます。

![Simutrans.sln内のソリューション一覧](/images/simutrans-standard/c793df3b54e3ea4a2e74b828247362e6.png)

この状態で、`Simutrans-Main`を展開すると、ソースコードの一覧が表示されます。
その中で`vehicle.cc`というファイルをクリックし、ソースコードを確認してみましょう。
このファイルが主にゲーム中に乗り物がどのような動作をするかのコードが書かれているファイルになります。
試しに、645 行目へジャンプをしてみると`make_smoke`という関数がありますが、これがゲーム中で車両が出す排煙をどのように描画するかの処理が書かれた関数です。

## デバッグしてみよう

次は実際にデバッグで、Simutrans のゲームを確認してみましょう。
Simutrans-GDI を右クリックし、デバッグを選ぶとデバッグモードへ移行しますが、その前にソリューションプラットフォームで構成を設定しておきます。
`x64`と`x86`の 2 つが設定されていますが、これを`vcpkg`のインストールで使用した bat ファイルと同じ構成を設定します。
ここで違うの方の設定を選ぶと、構成が合わずにビルドが成功しませんので注意な点です。

![ソリューション構成](/images/simutrans-standard/fa451b4810857b1bf6a76f972e25a547.png)

`ローカルWindowsデバッガー`をクリックするとデバッグのためのビルドが始まります。
ただし、このデバックはこのような警告画面が表示され失敗になります。
これは設定などが間違えているというよりかは、ゲームを起動させるために必要なファイルが存在しないためエラーが発生するため表示されるものです。
なので、ゲームに必要なファイルなどを用意していきましょう。

`Simutrans.sln`が存在するフォルダに、`simutrans`フォルダが存在しその中に最新のゲームの基本設定ファイルが存在します。
ai,config,font,music,script,text,themes などを、`build`フォルダの`GDI`フォルダ内へコピペします。
それらのフォルダをコピペできたら、次はゲームの車両モデルなどが入っている`pak`を同じ`GDI`フォルダへコピペします。
今回は、前にダウンロードした`pak.nippon`の Pak をコピペします。
これでデバッグの用意が完了しました。

`ローカルWindowsデバッガー`を再度クリックし、ゲームが起動するか確認しましょう。

![起動成功](/images/simutrans-standard/badf00bbe6c4731d76c33715a6359809.png)
~~ﾃﾚｰ~~

お疲れ様でした！
