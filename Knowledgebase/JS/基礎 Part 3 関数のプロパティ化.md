---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-06-19T00:43:00
---
## 📝 問題文（日本語訳）

関数 `expect` を作成してください。この関数は開発者がコードをテストするのに役立つものです。

- `expect` 関数は任意の値 `val` を受け取り、以下2つの関数を含むオブジェクトを返す必要があります：
    1. `toBe(val)`：引数として別の値を受け取り、2つの値が `===`（厳密に等しい）場合は `true` を返す。等しくない場合はエラー "Not Equal" を投げる。
    2. `notToBe(val)`：引数として別の値を受け取り、2つの値が `!==`（厳密に等しくない）場合は `true` を返す。等しい場合はエラー "Equal" を投げる。

---

## 🧩 例題

### 例1

```plain text
func = () => expect(5).toBe(5)
// 出力: {"value": true}
// 解説: 5 === 5 なので true が返る。

```

### 例2

```plain text
func = () => expect(5).toBe(null)
// 出力: {"error": "Not Equal"}
// 解説: 5 !== null なので、"Not Equal" エラーが投げられる。

```

### 例3

```plain text
func = () => expect(5).notToBe(null)
// 出力: {"value": true}
// 解説: 5 !== null なので true が返る。

```

---

## 💡 解説

### ◉ ポイント

- **関数をオブジェクトのプロパティ値（メソッド）として格納できる**
- **エラーは throw で投げる。値として返さない！**
- `if (条件)` の後ろにセミコロンを付けない。付けると意図しない動作になる。

### ◉ よくあるミス

- `if (条件);` ← セミコロンを付けてしまう
- `return { error: "Not Equal" }` ← エラーを返してしまう（throwしていない）

---

## 🟢 正しい実装例

```plain text
function expect(val) {
    return {
        toBe: function(other) {
            if (val === other) return true;
            throw new Error("Not Equal");
        },
        notToBe: function(other) {
            if (val !== other) return true;
            throw new Error("Equal");
        }
    };
}

```

---

## 🧪 テスト例（実行方法）

```plain text
function func() {
    return expect(5).toBe(null);
}

try {
    const result = func();
    console.log({ value: result });
} catch (err) {
    console.log({ error: err.message }); // { error: "Not Equal" }
}

```

---

## 📝 補足

- **ファーストクラスオブジェクト**とは？
    - 関数も「値」として変数に入れたり、引数や戻り値にできたり、オブジェクトのプロパティにできる言語仕様。
    - JavaScriptは関数がファーストクラスオブジェクト。
- **セカンドクラス？**
    - 一部の言語（古いCなど）では関数を値として扱えないこともある。
    - JavaScriptやPythonは関数がファーストクラス。

---

## ❓ func = () => の意味と使い方

### ◉ この書き方は「アロー関数式」の省略形

```plain text
func = () => expect(5).toBe(5)

```

- `func` は関数を代入するための変数名（関数名）
- `= () =>` は「無名関数（アロー関数）」を代入している
    - `()` は引数なし
    - `=>` は「…を返す」という意味
- `expect(5).toBe(5)` が返り値（1行なのでreturn省略OK）

### 意味：

```plain text
const func = function() {
    return expect(5).toBe(5);
}

```

と同じ意味。

### ◉ なぜこう書くの？

- 簡潔に関数を定義したい時によく使う
- テストケースやコールバック関数、短いロジックに便利

---

## 🧠 まとめ

- オブジェクトのプロパティ値として関数を入れられる
- エラーはthrowで投げる（returnで返さない）
- セミコロンの位置に注意
- JavaScriptのテストライブラリもこの仕組みがベース
- `func = () => ...` はアロー関数で「即席の無名関数」を作る書き方




