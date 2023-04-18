---
title: "[React]Suspense + ErrorBoundaryでエラー処理を行ってみよう"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react]
published: true
---

## はじめに

正常値の処理は簡単だけど、エラー系の処理は結構複雑になりがちだと思われます（たぶん。
特に Recoil の Selector を複数ネストした場合で、データ取得が奥の方だと大変であったり…。
今回は個人開発で躓いたエラー系の処理を、 React の Suspense と ErrorBoundary を使い、
コンポーネントを宣言的に使ってエラー処理を行ってみる内容です。
(でも記事の内容的に Recoil はあんまり関係ないかも…)

### 問題...

Hook と Component で分かれていた場合、エラー処理をする場合は主にこういう風になるかと思います。
(~~Hook 名は適当~~)

```ts
const Component = () => {
  const [data, error] = useZenn();

  if (error) {
    return <div>エラー</div>;
  }

  return <div>{data}:正常だよ！</div>;
};
```

このように、Hook からの値の状態を見てエラー画面を表示するかどうかを自前で実装するか、
なにかしらのメンテナンスされているライブラリを使うかになるかの 2 択になるはず。

`useZenn`の Hook が単純な処理ならこれでも簡単に実装できますが、問題は Hook の中でデータ取得するのが複雑な場合。
その場合、データ取得の部分から Hook で値を出す部分まで順通りに正常値かエラー値かを判別しながら関数を戻っていくか、
`throw`されたエラーをさらにどこかで Catch してエラーの値に整形して…の処理がおそらく必要になるかなと。

また、正常値 or エラー値で判別したり、正常値 and エラー値のプロパティを持つオブジェクトを作って戻したりすると、
今度は 型が複雑になり TypeScript 君がオコになりやすいため、型がめんどーーになるって思われるかと。

### ErrorBoundary! ...がだめ！

https://17.reactjs.org/docs/error-boundaries.html

ところで、React には子コンポーネントでエラーが発生した場合には、そのエラーをキャッチしてエラー画面を出してくれる便利な機能があります。
これを使えばいいじゃん！って思いますが、実は対応してない部分があるのは有名な話。

:::message
ErrorBoundary は、以下のエラーをキャッチしません。
・イベントハンドラー
・**非同期コード**
・サーバー側のレンダリング
・ErrorBoundary で throw されたエラー
:::

非同期処理で発生したエラーは全く対応してないってことになります。
そして、ほとんどがデータ取得が非同期処理で行われている現代じゃ、ほぼこれは使えない機能ということになります。
そうなると、正常値とエラー値を持つ状態で返し使用するコンポーネント側で状態を見て判別するしかなくなってしまいます。
しかも、宣言的じゃないのとテストの要件も複雑になりがちなので、個人開発でもロジックの設計が複雑になりがちにでした。(たぶん)

### ErrorBundary と Suspense

Suspense は React18 で正式採用された機能の一つです。
実際には React16.6 で実験的に実装されてましたので、実際に使えるようになったのは少し前から。

https://react.dev/reference/react/Suspense

Suspense は子コンポーネントで非同期処理中の読み込みを感知し、代わりにフォールバックコンテンツをレンダリングする機能です。
(実際には Promise が発生させている throw をレンダリングエラーとしてキャッチし、代わりのコンテンツを描画しているという流れです)
この機能を ErrorBoundary と組み合わせて使う事で、非同期処理のエラーを ErrorBoundary がキャッチし、エラー画面を表示できます。

```ts
interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
}

class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props | Readonly<Props>) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(_: any) {
    return { hasError: true };
  }

  render(): React.ReactNode {
    if (this.state.hasError) {
      return this.props.fallback;
    }

    return this.props.children;
  }
}
```

`ErrorBoundary`の構造はこのようにしておきます。
公式の例は JavaScript でしたので、TypeScript で型をつけておきます。
といっても、公式のに型を与えるぐらいですが。
この`ErrorBoundary`と`Suspense`を、上の方に書いた`Component`と組み合わせるとこのようになります。

```jsx
<ErrorBoundary fallback={<h2>Error!!!!</h2>}>
  <Suspense fallback={<h1>Loading...</h1>}>
    <Component />
  </Suspense>
</ErrorBoundary>
```

このようになり、`Component`でエラーが発生すると`ErrorBoundary`の Error!!!!され、エラーがキャッチされます。
これで、自身でエラーの状態を管理する必要がなくなるので、`Component`はエラー処理を自分でしなくてよくなるので、
責務が限定されてすっきりしますね。

```ts
const Component = () => {
  const [data] = useZenn();

  return <div>{data}:正常だよ！</div>;
};
```

実際にエラーを throw する場所も、FetchAPI や Axios を使用した箇所のよくある status チェックの場所で throw するだけでよいので、
各ロジックや状態も責務が限定されるため、処理も正常系だけ扱えばよくなるのでシンプルになると思います。

#### Recoil でも同じ

Recoil でも扱いは同じで、Recoil の Selector 内で FetchAPI などを使いデータ処理を行った箇所で throw を行えばよいだけになります。

```ts
const ZennQuery = selectorFamily<SampleData, QueryInput>({
  key: "data-flow/ZennQuery",
  get:
    ({ query }) =>
    async () => {
      const result = await fetch<ApiResult>(query);
      if (result.error) {
        throw result.error;
      }

      if (result.data === undefined) {
        throw new Error("No Data");
      }

      return result.data;
    },
});
```

このようにエラーを throw しておけば、後は Suspense と ErrorBoundary がキャッチをしエラーレンダリングを行ってくれます。

## 終わりに

Suspense と ErrorBoundary の組み合わせにより、ErrorBoundary が抱えていた非同期エラーはキャッチ出来ない問題を解消しています。
そして、2 つを組み合わせることで、自らエラーの状態を管理せずともよくなり、2 つを宣言的に組み込む事でエラー処理を行えるようになりました。
これを行う事で、ロジック部分やコンポーネント部分もより責務を限定しシンプルな構成に出来ると思うので、これからもどんどん使っていきたいと思います。
