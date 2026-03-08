---
base: "[[JavaScript.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2026-02-15T12:00:00
tags:
  - class
  - クラス
  - インスタンス
  - prototype
aliases:
  - Class
  - class
  - クラス
---

# [[基礎 Part 19 Class|Class]]

---

## [[基礎 Part 19 Class|Class]]の目的

**[[基礎 Part 19 Class|クラス]]**の目的は、同じ構造と振る舞いを持つ[[基礎 Part 15 オブジェクト|オブジェクト]]を効率的かつ安全に生成することである。

具体的には以下を実現するために存在する。

- データとその操作をひとまとめにする
- 同じ概念の実体を複数生成できるようにする
- コードの再利用性を高める
- 責務を分離し、設計を整理する

[[基礎 Part 19 Class|クラス]]は単なる文法機能ではなく、「状態を持つ概念」を表現するための設計手段である。  
（内部では[[基礎 Part 18 コンストラクタ関数|コンストラクタ関数]]と[[基礎 Part 17 プロトタイプ|prototype]]で動いている＝[[基礎 Part 16 糖衣構文|糖衣構文]]である。）

---

## 1. [[基礎 Part 15 オブジェクト|オブジェクト]]の基本構造

```js
const user1 = {
  name: "Taro",
  greet() {
    console.log("Hello, I'm " + this.name);
  }
};
```

### 構造

- `name` → データ（プロパティ）
- `greet` → 関数（メソッド）

JavaScriptでは関数も値であるため、[[基礎 Part 15 オブジェクト|オブジェクト]]の中に格納できる。

### 図解

```
user1
├─ name: "Taro"
└─ greet: function
```

同じ構造を複数作ると、関数もその数だけ生成される。

---

## 2. [[基礎 Part 19 Class|クラス]]とは何か

[[基礎 Part 19 Class|クラス]]は「**[[基礎 Part 15 オブジェクト|オブジェクト]]を作るための設計図**」である。

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log("Hello, I'm " + this.name);
  }
}
```

### インスタンス生成

```js
const user1 = new User("Taro");
const user2 = new User("Hanako");
```

---

## 3. new がしていること

```js
const user1 = new User("Taro");
```

内部処理の流れ：

1. 空の[[基礎 Part 15 オブジェクト|オブジェクト]]を作る
2. その[[基礎 Part 15 オブジェクト|オブジェクト]]の [[Prototype]] を `User.prototype` に設定
3. [[基礎 Part 18 コンストラクタ関数|constructor]] を実行
4. [[基礎 Part 15 オブジェクト|オブジェクト]]を返す

---

## 4. [[基礎 Part 17 プロトタイプ|prototype]] 構造

[[基礎 Part 19 Class|クラス]]で定義したメソッドは各インスタンスに複製されない。  
**[[基礎 Part 17 プロトタイプ|prototype]] に1つだけ存在する**。

### 図解

```
User
├─ constructor
└─ prototype
    └─ greet()

user1
├─ name: "Taro"
└─ [[Prototype]] → User.prototype

user2
├─ name: "Hanako"
└─ [[Prototype]] → User.prototype
```

user1 と user2 は同じ greet を共有する。

---

## 5. [[基礎 Part 18 コンストラクタ関数|constructor]] の役割

```js
constructor(name) {
  this.name = name;
}
```

- 左側 → インスタンスのプロパティ
- 右側 → 引数

`this.name = name` は「渡された値をインスタンスに保存する」という意味である。

引数名とプロパティ名は一致させるのが一般的である。理由は可読性のためである。

---

## 6. 具体例：Product [[基礎 Part 19 Class|クラス]]

```js
class Product {
  constructor(name, price, stock) {
    this.name = name;
    this.price = price;
    this.stock = stock;
  }

  buy(quantity) {
    if (quantity > this.stock) {
      console.log("在庫が足りない");
      return;
    }
    this.stock -= quantity;
  }

  applyDiscount(percent) {
    this.price = this.price * (1 - percent / 100);
  }
}
```

### 使用例

```js
const apple = new Product("Apple", 100, 10);
apple.buy(2);
apple.applyDiscount(20);
```

この[[基礎 Part 19 Class|クラス]]は

- **状態**（name, price, stock）
- **振る舞い**（buy, applyDiscount）

をセットで持っている。

---

## 7. 何を[[基礎 Part 19 Class|クラス]]にするべきか

### [[基礎 Part 19 Class|クラス]]に向いているもの

- 状態を持つ
- 状態を操作する振る舞いがある
- 複数インスタンスが存在する可能性がある
- 概念として「モノ」として扱える

例：User, Product, Cart, Order, TokenManager

### [[基礎 Part 19 Class|クラス]]にしなくてよいもの

- 単純な計算処理
- 状態を持たない関数
- 単発のロジック

例：

```js
function login(email, password) {
  // 認証処理
}
```

ログイン処理自体は「手続き」であり、通常は[[基礎 Part 19 Class|クラス]]にする必要はない。

ただし、トークン管理やセッション状態を保持するなら、AuthClient や TokenManager のような[[基礎 Part 19 Class|クラス]]にする意味がある。

---

## 8. まとめ

- [[基礎 Part 19 Class|クラス]]は設計図
- new はインスタンスを生成する
- メソッドは [[基礎 Part 17 プロトタイプ|prototype]] に保存され共有される
- 状態と振る舞いをまとめるのが[[基礎 Part 19 Class|クラス]]の本質
- すべてを[[基礎 Part 19 Class|クラス]]にする必要はない
- 「状態を持つ概念」を切り出すのが設計のポイント
