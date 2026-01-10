---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-06-24T00:40:00
---
## 1. 関数宣言（Function Declaration）

```javascript
function greet(name) {
  return "Hello, " + name;
}

console.log(greet("克樹")); // Hello, 克樹

```

**特徴**:

- 関数名が付く（`greet`）
- 「巻き上げ（hoisting）」される → 定義前でも呼び出せる
- よくグローバルやトップレベルで使う

---

## 2. 関数式（Function Expression）

```javascript
const greet = function(name) {
  return "Hello, " + name;
};

console.log(greet("克樹")); // Hello, 克樹

```

**特徴**:

- 無名関数（関数自体には名前がない）
- 変数に代入して使う
- 巻き上げされない（定義より前で呼ぶとエラー）
- 変数に関数を代入できるので、条件によって関数内容を切り替えたいときにも使える
例：

```javascript
let greet;
if (isMorning) {
  greet = function(name) {
    return "おはよう, " + name;
  };
} else {
  greet = function(name) {
    return "こんにちは, " + name;
  };
}
console.log(greet("克樹"));
// isMorningがtrueなら「おはよう, 克樹」
// falseなら「こんにちは, 克樹」
```

---

## 3. アロー関数（Arrow Function）

```javascript
const greet = name => "Hello, " + name;

console.log(greet("克樹")); // Hello, 克樹

```

**特徴**:

- より簡潔な記法（特に短い関数やコールバックに便利）
- `this` を持たない（外側の `this` を継承）
- ES6（2015）から追加された書き方
- 上記の場合だと、変数に代入されているので同じく関数式で、巻き上げはされない。すなわち定義前に呼ぶとエラーになる。

### アロー関数のバリエーション

```javascript
// 引数が1つだけなら () を省略可能
const square = x => x * x;

// 引数が複数の場合は () 必須
const sum = (a, b) => a + b;

// 処理が複数行なら { } と return が必要
const pow = (a, b) => {
  return a ** b;
};

```

---

## まとめ表

| 書き方 | 省略記法 | 巻き上げ | thisの扱い | 主な用途 |
| --- | --- | --- | --- | --- |
| `function f() {}` | ❌ | ✅ | 関数自身の `this` | 普通の処理 |
| `const f = function() {}` | ❌ | ❌ | 関数自身の `this` | 条件付き定義など |
| `const f = () => {}` | ✅ | ❌ | 外側の `this` を継承 | コールバック、合成など |

---

## ポイント

- **どの書き方も「関数」を定義できるが、細かい特徴が違うので場面によって使い分ける**
- **アロー関数はシンプルでよく使うが、thisの振る舞いに注意**
