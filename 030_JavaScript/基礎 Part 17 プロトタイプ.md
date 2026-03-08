---
base: "[[JavaScript.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2026-02-15T12:00:00
tags:
  - プロトタイプ
  - prototype
  - 継承
aliases:
  - プロトタイプ
  - prototype
---

# JavaScript [[基礎 Part 17 プロトタイプ|プロトタイプ]]基礎まとめ

---

## 1. [[基礎 Part 17 プロトタイプ|プロトタイプ]]とは何か

**[[基礎 Part 17 プロトタイプ|プロトタイプ]]**とは、[[基礎 Part 15 オブジェクト|オブジェクト]]が持つ「参照リンク」である。

すべての[[基礎 Part 15 オブジェクト|オブジェクト]]は内部に **`[[Prototype]]`** という隠しスロットを持っている。

**`[[Prototype]]` は別の何かではなく**、上で言った「参照リンク」を格納している**場所の名前**である。ECMAScript仕様で使われる `[[...]]` は「内部スロット」を表す記法で、JavaScriptのコードからは直接触れない。つまり「プロトタイプ」という概念を、エンジン内部では `[[Prototype]]` という名前のスロットで実装している、という関係になる。

**このスロットに入っているもの**は、**「次にたどるオブジェクト」への参照**（もう1つのオブジェクト、またはチェーンの終端を示す `null`）だけである。プロパティ名や値の一覧が入っているわけではなく、「ないプロパティを探しに行く先」のアドレスが1つ入っている、というイメージである。

重要なのは、**これはコピーではなく参照である**という点である。

- **参照**：子[[基礎 Part 15 オブジェクト|オブジェクト]]は親[[基礎 Part 15 オブジェクト|オブジェクト]]のプロパティを「コピーして持っている」のではなく、親[[基礎 Part 15 オブジェクト|オブジェクト]]そのものへの**リンク（アドレス）**だけを持っている。プロパティにアクセスするたびに、そのリンクをたどって親の実体を見に行く。
- **コピーだった場合**：親のプロパティを子に複写して持たせる方式だと、親を後から変更しても子には反映されない。また、同じ内容を複数[[基礎 Part 15 オブジェクト|オブジェクト]]が持つことになりメモリも重くなる。
- **参照である利点**：親が1つあるだけなのでメモリ効率がよく、親を変更すればそれを参照している子すべてに即座に反映される（詳しくは「3. 参照であることの重要性」で扱う）。

### 図解

```
child
  ├─ 自分のプロパティ
  └─ [[Prototype]] → parent
```

child に存在しないプロパティは parent を探しに行く。

---

## 2. プロパティ探索の本質

JavaScriptでプロパティにアクセスするとき、エンジンは次の順番で探索する。

```
1. 自分自身にあるか確認する
2. なければ [[Prototype]] を見る
3. null になるまで繰り返す
```

これは単純な再帰探索である。

### 図解

```
child
  ↓
parent
  ↓
Object.prototype
  ↓
null
```

### 具体例

```js
const parent = {
  value: 10
};

const child = Object.create(parent);

child.value; // 10
```

child 自身には value は存在しない。  
しかし parent を参照しているため取得できる。

---

## 3. 参照であることの重要性

[[基礎 Part 17 プロトタイプ|プロトタイプ]]は**コピーではない**。  
親[[基礎 Part 15 オブジェクト|オブジェクト]]が変更されれば、子にも影響する。

### 具体例

```js
const parent = { count: 1 };
const child = Object.create(parent);

parent.count = 999;

child.count; // 999
```

child は常に parent を参照しているため、最新の値が反映される。

---

## 4. [[基礎 Part 17 プロトタイプ|プロトタイプ]]チェーン

[[基礎 Part 17 プロトタイプ|プロトタイプ]]は1段階だけではない。  
親にもさらに親が存在する。

これが連鎖し、最終的に `null` に到達する。

### 図解

```
child
  ↓
parent
  ↓
grandParent
  ↓
Object.prototype
  ↓
null
```

### 多段階の具体例

```js
const grandParent = { a: 1 };
const parent = Object.create(grandParent);
const child = Object.create(parent);

child.a; // 1
```

探索の流れは以下の通りである。

```
child に a はない
  ↓
parent にもない
  ↓
grandParent にある
  ↓
返す
```

---

## 5. [[基礎 Part 15 オブジェクト|Object]].[[基礎 Part 17 プロトタイプ|prototype]] の意味

通常の[[基礎 Part 15 オブジェクト|オブジェクト]]は最終的に **[[基礎 Part 15 オブジェクト|Object]].[[基礎 Part 17 プロトタイプ|prototype]]** を継承する。

ここには標準メソッドが定義されている。

例：

- `toString`
- `hasOwnProperty`
- `valueOf`

### 図解

```
{}
  ↓
Object.prototype
  ↓
null
```

### 具体例

```js
const obj = {};

obj.toString();
obj.hasOwnProperty("x");
```

これらは obj に直接定義されているわけではない。  
[[基礎 Part 15 オブジェクト|Object]].[[基礎 Part 17 プロトタイプ|prototype]] に存在する。

---

## 6. [[基礎 Part 17 プロトタイプ|prototype]] プロパティの意味

関数は特別に **`prototype`** というプロパティを持つ。

これは「その関数から生成される[[基礎 Part 15 オブジェクト|オブジェクト]]の親になる[[基礎 Part 15 オブジェクト|オブジェクト]]」である。

### 図解

```
function Person() {}

Person.prototype
      ↓
(インスタンスの [[Prototype]] になる)
```

### 具体例

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = function() {
  return "hi";
};

const taro = new Person("Taro");
```

構造は以下のようになる。

```
taro
  ↓
Person.prototype
  ↓
Object.prototype
  ↓
null
```

---

## 7. [[Prototype]] と [[基礎 Part 17 プロトタイプ|prototype]] の違い

| 名称 | 意味 |
|------|------|
| **[[Prototype]]** | すべての[[基礎 Part 15 オブジェクト|オブジェクト]]が持つ内部リンク |
| **[[基礎 Part 17 プロトタイプ|prototype]]** | 関数が持つ、インスタンスの親になる[[基礎 Part 15 オブジェクト|オブジェクト]] |

関係は以下である。

```
インスタンスの [[Prototype]] === コンストラクタ.prototype
```

---

## 8. 本質整理

1. すべての[[基礎 Part 15 オブジェクト|オブジェクト]]は [[Prototype]] を持つ
2. プロパティ探索はチェーンをたどる
3. 標準メソッドは [[基礎 Part 15 オブジェクト|Object]].[[基礎 Part 17 プロトタイプ|prototype]] にある
4. [[基礎 Part 17 プロトタイプ|prototype]] はインスタンスの親を定義するための仕組み
5. JavaScriptは[[基礎 Part 15 オブジェクト|オブジェクト]]同士の参照構造で動いている

以上が、[[基礎 Part 17 プロトタイプ|prototype]]そのものの構造と動作原理である。
