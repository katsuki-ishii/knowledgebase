---
base: "[[JavaScript.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2026-02-15T12:00:00
tags:
  - 糖衣構文
  - class
  - prototype
aliases:
  - 糖衣構文
  - Syntactic Sugar
  - シンタックスシュガー
---

# [[基礎 Part 16 糖衣構文|糖衣構文]]（[[基礎 Part 16 糖衣構文|Syntactic Sugar]]）

---

## 1. [[基礎 Part 16 糖衣構文|糖衣構文]]とは

**[[基礎 Part 16 糖衣構文|糖衣構文]]（[[基礎 Part 16 糖衣構文|Syntactic Sugar]]）**とは、本質的な仕組みは変えずに、より書きやすく・読みやすくした構文のことである。

重要なのは「**内部の動作は変わらない**」という点である。

---

## 2. [[HTMLのclass, id, style|class]] は[[基礎 Part 16 糖衣構文|糖衣構文]]である

### [[HTMLのclass, id, style|class]]構文

```js
class Person {
  constructor(name) {
    this.name = name
  }

  greet() {
    console.log("Hi " + this.name)
  }
}
```

### 実際の内部構造（prototypeベース）

```js
function Person(name) {
  this.name = name
}

Person.prototype.greet = function() {
  console.log("Hi " + this.name)
}
```

やっていることは同じであり、**[[HTMLのclass, id, style|class]]はprototype構造の[[基礎 Part 16 糖衣構文|糖衣構文]]**である。

---

## 3. new の内部動作

```js
const p = new Person("Taro")
```

内部では次の処理が行われている。

1. 空の[[基礎 Part 15 オブジェクト|オブジェクト]]を生成
2. その[[基礎 Part 15 オブジェクト|オブジェクト]]を `this` に設定
3. 関数を実行
4. その[[基礎 Part 15 オブジェクト|オブジェクト]]を返す

### 図解

```
new Person("Taro")
    ↓
{}  ← 空オブジェクト生成
    ↓
this = {}
    ↓
this.name = "Taro"
    ↓
{ name: "Taro" }
```

---

## 4. 関数は[[基礎 Part 15 オブジェクト|オブジェクト]]である

JavaScriptでは関数も[[基礎 Part 15 オブジェクト|オブジェクト]]である。

```js
function test() {}

test.x = 10
console.log(test.x) // 10
```

関数は次の特徴を持つ。

- 呼び出せる
- プロパティを持てる

---

## 5. 他の代表的な[[基礎 Part 16 糖衣構文|糖衣構文]]

### 5-1. [[基礎 Part 10 関数の定義方法3つ|アロー関数]]

```js
(x) => x * 2
```

↓

```js
function(x) {
  return x * 2
}
```

---

### 5-2. [[基礎 Part 13 分割代入|分割代入]]

```js
const { name } = user
```

↓

```js
const name = user.name
```

---

### 5-3. [[基礎 Part 14 スプレット構文|スプレッド構文]]

```js
const copy = { ...obj }
```

↓

```js
const copy = Object.assign({}, obj)
```

---

### 5-4. テンプレートリテラル

```js
`Hello ${name}`
```

↓

```js
"Hello " + name
```

---

### 5-5. async / await

```js
await fetchData()
```

↓

```js
fetchData().then(result => {
  // 処理
})
```

---

## 6. [[基礎 Part 16 糖衣構文|糖衣構文]]と生成[[AI激動期における「定数」思考|AI]]時代

生成[[AI激動期における「定数」思考|AI]]は、[[基礎 Part 16 糖衣構文|糖衣構文]]もプリミティブな構文もどちらも理解できる。

重要なのは「どの構文を暗記しているか」ではなく、

- 内部で何が起きているか説明できるか
- バグの原因を構造的に追えるか

である。

---

## 7. 本質の整理

[[基礎 Part 16 糖衣構文|糖衣構文]]とは

> **「抽象度を上げて、人間にとって扱いやすくした構文」**である。

しかし内部の仕組みは消えていない。

理解の階層は次のようになる。

| 層 | 例 |
|----|-----|
| **表層** | [[HTMLのclass, id, style|class]] / async / [[基礎 Part 14 スプレット構文|スプレッド]] など |
| **中層** | prototype / Promise / プロパティアクセス |
| **基礎** | [[基礎 Part 15 オブジェクト|オブジェクト]]生成 / 実行コンテキスト / this の束縛 |

上の層だけ知っていてもコードは書ける。  
しかし**下の層まで理解していると、設計とデバッグができる**。
