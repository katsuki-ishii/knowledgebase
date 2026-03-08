---
base: "[[JavaScript.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2026-02-15T12:00:00
tags:
  - コンストラクタ
  - constructor
  - new
  - prototype
aliases:
  - コンストラクタ関数
  - コンストラクタ
  - constructor
---

# [[基礎 Part 18 コンストラクタ関数|コンストラクタ関数]]

---

**[[基礎 Part 18 コンストラクタ関数|コンストラクタ関数]]**とは、

「同じ構造を持つ[[基礎 Part 15 オブジェクト|オブジェクト]]を効率よく量産するための仕組み」である。

目的は次の3つ。

1. 同じ形のデータを何度も作れるようにする
2. メソッドを共有してメモリ効率を良くする
3. [[基礎 Part 15 オブジェクト|オブジェクト]]に“型”という概念を持たせる

単なる便利関数ではなく、「設計図からインスタンスを作る」という発想を実現するための仕組みである。

---

## 1. 出発点：普通の関数で[[基礎 Part 15 オブジェクト|オブジェクト]]を作る

まずは最もシンプルな方法から。

```js
function createUser(name, age) {
  return {
    name: name,
    age: age
  };
}

const user1 = createUser("Taro", 25);
const user2 = createUser("Jiro", 30);
```

これは全く問題ない。

### 仕組み

```
createUser("Taro", 25)
        ↓
  新しいオブジェクトを作る
        ↓
  それを return する
```

### 特徴

- 毎回[[基礎 Part 15 オブジェクト|オブジェクト]]を返す
- シンプルでわかりやすい
- 小規模コードなら十分

---

## 2. 問題になるケース

メソッドを持たせる場合を考える。

```js
function createUser(name, age) {
  return {
    name,
    age,
    greet: function() {
      console.log("Hello " + name);
    }
  };
}
```

userを1000個作るとどうなるか。

```
user1 → greet(新しい関数)
user2 → greet(また新しい関数)
user3 → greet(また別の関数)
...
```

同じ処理なのに関数が毎回生成される。  
これがメモリ効率の問題である。

---

## 3. [[基礎 Part 18 コンストラクタ関数|コンストラクタ関数]]の登場

```js
function User(name, age) {
  this.name = name;
  this.age = age;
}

const user1 = new User("Taro", 25);
const user2 = new User("Jiro", 30);
```

### new がやっていること

```
1. 空のオブジェクトを作る
2. this をそのオブジェクトにする
3. 中身を代入する
4. 自動でそのオブジェクトを返す
```

図解：

```
new User("Taro", 25)
        ↓
   {}  ← 空オブジェクト生成
        ↓
   this が {} を指す
        ↓
   { name: "Taro", age: 25 }
        ↓
   それが返る
```

---

## 4. [[基礎 Part 17 プロトタイプ|prototype]]による共有

```js
User.prototype.greet = function() {
  console.log("Hello " + this.name);
};
```

### 仕組み

```
           User.prototype
                 ↑
                 |
user1  ──────────┘
user2  ──────────┘
```

全インスタンスが同じ greet 関数を参照する。

### 結果

- greet は1つしか存在しない
- メモリ効率が良い
- 処理が安定する

---

## 5. 型という概念

```js
user1 instanceof User // true
```

これは「この[[基礎 Part 15 オブジェクト|オブジェクト]]は User 型から作られた」と言えるということである。

単なる `{}` では判別できない。

---

## 6. 後から機能を追加できる

```js
User.prototype.sayAge = function() {
  console.log(this.age);
};
```

すでに作った user にも自動で追加される。

return パターンではできない。

---

## 7. [[基礎 Part 24 class指定|class]] との関係

```js
class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}
```

[[基礎 Part 24 class指定|class]] は内部的に

**[[基礎 Part 18 コンストラクタ関数|コンストラクタ関数]] + [[基礎 Part 17 プロトタイプ|prototype]]**

で動いている（[[基礎 Part 16 糖衣構文|糖衣構文]]である）。

---

## 8. まとめ

### return 関数

- シンプル
- 小規模なら十分
- 型の概念は弱い

### [[基礎 Part 18 コンストラクタ関数|コンストラクタ関数]]

- [[基礎 Part 17 プロトタイプ|prototype]] で共有可能
- メモリ効率が良い
- instanceof が使える
- 継承できる
- [[基礎 Part 24 class指定|class]] の土台
- 設計意図が明確

---

## 結論

小規模なら return でよい。

しかし、

- [[基礎 Part 15 オブジェクト|オブジェクト]]を大量に作る
- 型の概念を持たせたい
- 継承や拡張を考える
- [[基礎 Part 24 class指定|class]] の本質を理解したい

場合には[[基礎 Part 18 コンストラクタ関数|コンストラクタ関数]]の理解が不可欠である。
