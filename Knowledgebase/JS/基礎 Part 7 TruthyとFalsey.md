---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-06-22T20:10:00
aliases: [TruthyとFalsey, Truthy, Falsey, truthy, falsey, falsy, Falsy]
---
## 概要

JavaScriptでは、条件式で使う値が「trueとみなされる値（[[基礎 Part 7 TruthyとFalsey|truthy]]）」と「falseとみなされる値（[[基礎 Part 7 TruthyとFalsey|falsey]]）」に分かれる。

- [[基礎 Part 7 TruthyとFalsey|truthy]]: 条件式でtrueと判定される値
- [[基礎 Part 7 TruthyとFalsey|falsey]] ([[基礎 Part 7 TruthyとFalsey|falsy]]): 条件式でfalseと判定される値

---

## [[基礎 Part 7 TruthyとFalsey|falsey]]になる値（7種類）

[[基礎 Part 7 TruthyとFalsey|falsey]]になるのは次の7つだけ。

| 値 | 実体例 | コメント |
| --- | --- | --- |
| `false` | `false` | falseリテラル |
| `0` | `0`, `-0`, `0n` | 数値のゼロ（BigInt含む） |
| `""` | `""` | 空文字列 |
| `null` | `null` | null値 |
| `undefined` | `undefined` | 未定義 |
| `NaN` | `NaN` | Not a Number（数値でない値） |
| `document.all` | `document.all` | 特殊（ほぼ使わない） |

```plain text
if (false)      // falsey
if (0)          // falsey
if (-0)         // falsey
if (0n)         // falsey
if ("")        // falsey
if (null)       // falsey
if (undefined)  // falsey
if (NaN)        // falsey

```

---

## [[基礎 Part 7 TruthyとFalsey|truthy]]な値（それ以外すべて）

上記以外の値はすべて[[基礎 Part 7 TruthyとFalsey|truthy]]。

| 値のタイプ | 実体例 |
| --- | --- |
| 数値 | `1`, `-5`, `3.14` |
| 文字列 | `"a"`, `"0"`, `"false"` |
| 配列 | `[]`, `[1,2,3]` |
| オブジェクト | `{}`, `{a:1}` |
| 関数 | `function() {}` |
| trueリテラル | `true` |
| BigInt（0n以外） | `1n`, `-123n` |
| Symbol | `Symbol("a")` |

```plain text
if (1)           // truthy
if ("a")         // truthy
if ([])          // truthy
if ({})          // truthy
if (function(){})// truthy
if ("0")         // truthy

```

---

## 注意が必要な具体例

- `[]` （空配列）は[[基礎 Part 7 TruthyとFalsey|truthy]]
- `{}` （空オブジェクト）は[[基礎 Part 7 TruthyとFalsey|truthy]]
- `"0"`（0の文字列）は[[基礎 Part 7 TruthyとFalsey|truthy]]
- `NaN`は[[基礎 Part 7 TruthyとFalsey|falsey]]、`"NaN"`（文字列）は[[基礎 Part 7 TruthyとFalsey|truthy]]
- `false`（boolean）は[[基礎 Part 7 TruthyとFalsey|falsey]]、`"false"`（文字列）は[[基礎 Part 7 TruthyとFalsey|truthy]]

---

## サンプルコード

```plain text
// 空文字列はfalsey
if ("") {
  console.log("truthy");
} else {
  console.log("falsey"); // "falsey"
}

// 配列やオブジェクトは空でもtruthy
if ([]) {
  console.log("配列はtruthy"); // 実行される
}
if ({}) {
  console.log("オブジェクトもtruthy"); // 実行される
}

// 0はfalseyだが"0"はtruthy
if (0) {
  console.log("0 is truthy");
} else {
  console.log("0 is falsey"); // 実行される
}
if ("0") {
  console.log('"0" is truthy'); // 実行される
}

```

---

## まとめ

- [[基礎 Part 7 TruthyとFalsey|falsey]]になるのは上記7種類だけ
- 配列やオブジェクトは中身がなくても[[基礎 Part 7 TruthyとFalsey|truthy]]
- 文字列"0"や"false"も[[基礎 Part 7 TruthyとFalsey|truthy]]
- 型・値の違いに注意
