---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-06-22T20:10:00
---
## 概要

JavaScriptでは、条件式で使う値が「trueとみなされる値（truthy）」と「falseとみなされる値（falsey）」に分かれる。

- truthy: 条件式でtrueと判定される値
- falsey (falsy): 条件式でfalseと判定される値

---

## falseyになる値（7種類）

falseyになるのは次の7つだけ。

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

## truthyな値（それ以外すべて）

上記以外の値はすべてtruthy。

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

- `[]` （空配列）はtruthy
- `{}` （空オブジェクト）はtruthy
- `"0"`（0の文字列）はtruthy
- `NaN`はfalsey、`"NaN"`（文字列）はtruthy
- `false`（boolean）はfalsey、`"false"`（文字列）はtruthy

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

- falseyになるのは上記7種類だけ
- 配列やオブジェクトは中身がなくてもtruthy
- 文字列"0"や"false"もtruthy
- 型・値の違いに注意
