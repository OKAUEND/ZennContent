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
    it("Atom Update TEST", async () => {
      const { result } = renderHook(() => useStateTEST());
      const [text , setText] = result.current;

      expect(text).toEqual("");

      act(() => {
        setText("TEXT");
      });

      expect(text).toEqual("TEXT");
    });
  });
```

### 勘違い

これで状態の変更をテストしようとしたら、反映されなくて 1 日ぐらい悩みました。
原因は、CustomHook の戻値が tuple であり、使用側でそれを分割代入しているため。
CustomHook の状態が変更されると、戻り値にも反映され、それが分割代入部分にも…と考えた。

### ちゃんと確認しようね

```ts:Vitest
  describe("Custom hook TEST", () => {
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

テストで参照する値を、CustomHook の戻り値に指定するようにするだけ。
戻り値が tuple なので、配列のインデックスを指定するのは少し見にくいなと思い、分割代入をしたので NG だった。
