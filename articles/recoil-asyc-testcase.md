---
title: "非同期Selectorの初期生成時のテスト方法"
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
![](/images/recoil-asyc-testcase/result.png)

## 非同期の場合は待機が必要

### 公式ドキュメントより

## 解決方法
