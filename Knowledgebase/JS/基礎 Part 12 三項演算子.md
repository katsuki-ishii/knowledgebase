---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-11-01T16:25:00
aliases: [三項演算子, 三項, 三項演算, ternary operator, ternary]
---
[[基礎 Part 12 三項演算子|三項演算子]]（[[基礎 Part 12 三項演算子|ternary operator]]）は、if 文を簡潔に書くための構文である。

### 基本構文

```plain text
条件 ? 値1 : 値2

```

- 条件が true（真）なら 値1 が返る
- 条件が false（偽）なら 値2 が返る

---

### if 文との比較

### 通常の if 文

```javascript
let age = 20;
let message;

if (age >= 18) {
  message = "大人です";
} else {
  message = "子どもです";
}

```

### [[基礎 Part 12 三項演算子|三項演算子]]を使った場合

```javascript
let age = 20;
let message = age >= 18 ? "大人です" : "子どもです";

```

上記の2つは同じ意味になる。[[基礎 Part 12 三項演算子|三項演算子]]を使うことで、1行で簡潔に書ける。

---

### 主な使い方

### 条件によって表示を切り替える

```javascript
<p>{{ isLoggedIn ? "ログイン中" : "ログアウト中" }}</p>

```

### 関数内で値を条件的に返す

```javascript
function getColor(isActive) {
  return isActive ? "green" : "gray";
}

```

---

### 注意点

- [[基礎 Part 12 三項演算子|三項演算子]]を入れ子にすると可読性が下がる。
```javascript
// 読みにくい例
let result = score > 80 ? "優" : score > 60 ? "良" : "可";

```
複雑な条件分岐は通常の if-else 文を使うほうが読みやすい。

---

### 式と文の違い

[[基礎 Part 12 三項演算子|三項演算子]]は「式（expression）」であり、値を返す。

一方、if 文は「文（statement）」であり、値を返さない。

このため、[[基礎 Part 12 三項演算子|三項演算子]]は変数への[[基礎 Part 8 宣言・代入・再代入|代入]]や関数の戻り値に直接使える。

---

### まとめ

| 項目 | 内容 |
| --- | --- |
| 目的 | if-else を簡潔に書く |
| 構文 | `条件 ? 値1 : 値2` |
| 戻り値 | 条件が真なら値1、偽なら値2 |
| 注意点 | ネストして使うと読みづらくなる |