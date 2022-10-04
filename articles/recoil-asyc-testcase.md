---
title: "Vitest環境にて、Recoilの非同期Selectorの詰まったポイント"
emoji: "🤗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [recoil, react, vitest]
published: false
---

# はじめに

## コード

```ts:Recoil
const asycRecoilSelector = selector<Promise<string>>({
  key: "asyc-seletor",
  get: async ({ get }) => {
    return await (
      await axios.get<string>("/testing")
    ).data;
  },
});
```

```ts:Custom hook
const useAsycRecoil = () => {
  const asycSelectorRecoil = useRecoilValue(asycRecoilSelector);

  const setCurrentID = useRecoilCallback(({ set }) => (text: string) => {
    set(currentIDAtom, text);
  });

  return [asycSelectorRecoil, setCurrentID] as const;
};
```

```ts:Vitest
  describe("Recoil Asyc Custom hook TEST", () => {
    it("デフォルトで最初から値がはいっているか", async () => {
      const { result } = renderHook(() => useAsycRecoil(), {
        wrapper: RecoilRoot,
      });

      expect(result.current[0]).toEqual("Hello!!");
    });
  });
```

通信部分は、`MSW`で mock 化しており、返答は単純な文字列の`Hello!!`にしておくこととする。

ただ、結果的にこのテストは失敗判定になってしまう。
理由としては、CustomHook の戻り値が`null`になってしまうため。

![失敗時の画像](/images/recoil-asyc-testcase/result.png)

### 非同期の場合は待機が必要

Selector の非同期処理が終わる前に、テストの処理が先に進んでしまうため、まだ値が生成されておらずに null になると思われる。
なので、公式ドキュメントにあるヘルパー関数を使い、テストの処理を一時中断だせて非同期処理を終わらせる必要がある。

## 解決方法

### 公式ドキュメントより

[Recoil Testing](https://recoiljs.org/docs/guides/testing/#testing-recoil-state-with-asynchronous-queries-inside-of-a-react-component)

```ts:ヘルパー関数
function flushPromisesAndTimers(): Promise<void> {
  return act(
    () =>
      new Promise(resolve => {
        setTimeout(resolve, 100);
        jest.runAllTimers();
      }),
  );
}
```

ただし、ドキュメントにあるヘルパー関数は Jest 用のため、Vitest で使用するためには変更が必要となる。
~~jest を参照してる時点で、Vitest 環境じゃ動かないしね…~~

```ts:Vitest Ver ヘルパー関数
function flushPromisesAndTimers(): Promise<void> {
  return act(
    () =>
      new Promise(resolve => {
        setTimeout(resolve, 100);
        //Mock timerを動作させるために、useFakeTimers()のメソッドを使用する
        vi.useFakeTimers()
        vi.runAllTimers();
      }),
  );
}
```

Vitest の環境では、Timer を動作させる時には useFakeTimers()を呼び出すのを追加させて、
Jest の参照を、Vitest の Utility を参照するように変更する。

これでテストが意図通りに動作する。
