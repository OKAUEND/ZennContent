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

![VisualStudioのインストール時にC++のみを選択する](/images/simutrans-standard/77e21aaf93efd499a8ac3520cec879e2.png)
:::
他に試したい機能があれば選ぶのもありですが、正直かなり容量を食います。
:::

インストール場所などは、各自の任意の場所へ。

### Git

https://git-scm.com/
ライブラリやソースコードを入手するために、github にあるのをクローンで入手する必要があるため、
Git を導入するため公式よりダウンロードをします。
公式サイトの Download for Windows より実行ファイルをダウンロードして、インストールを行ってください。

### vcpkg(※要 Git)

https://github.com/microsoft/vcpkg

以前ですと、ライブラリを入手するために別途 DL してビルドしていましたが、最近のは Microsoft が提供してる
クロスプラットフォームパッケージより入手します。

Code の部分より直接ダウンロードをし任意のフォルダに解凍するか、git の clone で入手するかの 2 つの手段があります。
今回は、git の clone を使ってやってみましょう。

まずは`コマンドプロンプト`を起動します。
次にパッケージを保存したいフォルダの位置へ、`cd`のコマンドを使い階層を移動します。
:::
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

### Simutrans の Pak とソースコード を入手

Simutrans は実行ファイルなどだけではプレイすることができず、モデルやゲーム内容などの数値が設定されている`Pak`が必要となります。
:::
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

## 構築を始める

必要なアプリやライブラリなどを用意できたので、これらを設定していき開発及びデバッグを行えるようにします。

### vcpkg から必要ライブラリをインストール

vcpkg から必要なライブラリをインストールするために、bat ファイルからライブラリのインストール処理を行います。
用意したソースコード内の`tools`フォルダ内にある、`install-building-libs-<PROCESS>.bat`を、`vcpkg`のフォルダへコピペします。

:::
32 ビットか 64 ビットどちらでデバッグを行うかで、x84(32 ビット版)か x64(64 ビット版)を選んでください。
デバッグを行う時に、デバッグを行うプロセスに合うライブラリが導入されていないと失敗になります。
:::

コピペが完了したら、次はコマンドプロンプトでまずは`vcpkg`同梱の bat ファイルを起動します。

```
<PARH>/bootstrap-vcpkg.bat
```

この同梱の bat ファイルの処理が終わったら、次は Simutrans の bat ファイルを起動します。
先に vcpkg のバッチファイルを処理していないと、この後の処理ができないので、必ずやっておきましょう。

```
<PARH>/install-building-libs-x64.bat
```

今回は 64 ビット版を導入するので x64 の方を使用します。
すると、長い処理が始まるまで終わるまで待機します。
最後に、`続行するには何かキーを押してください . . .`が表示されたら処理が完了です。
