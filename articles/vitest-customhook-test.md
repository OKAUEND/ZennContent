---
title: "VitestでCustom Hookのテストでやらかしたミス"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vitest, react]
published: false
---

# はじめに

Vitest と React と Custom Hooks のテストを行った時に、State の変更がテストで成功しなかったので、
その原因が基本的なミスだったので、自戒の念を込めて記事にしてみました。

## 実際のコード

```ts:Custom hook
export const useStateTEST = () => {
  const state = useState("");
  const setState = useCallback(({ set }) => (text: string) => {
    set(recoilAtom, text);
  });

  return [state, setState] as const;
};
```

```ts:Vitest
  describe("Custom hook TEST", () => {
    it("State TEST", () => {
      const { result } = renderHook(() => useStateTEST());
      expect(result.current[0]).toEqual("");
    });
    it("Atom Update TEST", async () => {
      const { result } = renderHook(() => useStateTEST());

      expect(result.current[0]).toEqual("");

      act(() => {
        result.current[1]("TEXT");
      });

      expect(result.current[0]).toEqual("TEXT");
    });
  });
```

### 勘違い

### ちゃんと確認しようね
