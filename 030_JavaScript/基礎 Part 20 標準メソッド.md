---
base: "[[JavaScript.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2026-02-13T12:00:00
aliases: [基礎 Part 20 標準メソッド, 標準メソッド, 標準メソッド JS, 組み込みメソッド, built-in methods]
---

#  [[基礎 Part 20 標準メソッド|標準メソッド]]

---

## 0. [[基礎 Part 20 標準メソッド|標準メソッド]]とは何か

[[基礎 Part 20 標準メソッド|標準メソッド]]とは、JavaScriptの実行環境にあらかじめ用意されているメソッドのことを指す。
開発者が自分で定義しなくても使用できる。

ただし、それらは特別な構文上の魔法ではない。
多くは各種[[基礎 Part 15 オブジェクト|オブジェクト]]の [[基礎 Part 17 プロトタイプ|prototype]] に定義された通常の関数である。

例えば次のようなものが[[基礎 Part 20 標準メソッド|標準メソッド]]にあたる。

* [[基礎 Part 15 オブジェクト|Object]]系: toString, hasOwnProperty, keys, assign
* Array系: push, pop, [[基礎 Part 6 Map関数の自作|map]], filter, reduce, forEach
* String系: includes, slice, replace, toUpperCase
* Function系: call, apply, bind
* Promise系: then, catch, finally

---

## 0-1. 実務で特によく使う[[基礎 Part 20 標準メソッド|標準メソッド]]

実務で頻繁に使用されるものを分類すると次の通り。

### 配列操作

```js
const arr = [1, 2, 3];

arr.push(4);
arr.map(x => x * 2);
arr.filter(x => x > 1);
arr.reduce((sum, x) => sum + x, 0);
```

フロントエンド・バックエンド問わず最重要。

### [[基礎 Part 15 オブジェクト|オブジェクト]]操作

```js
const obj = { a: 1, b: 2 };

Object.keys(obj);
Object.assign({}, obj);
obj.hasOwnProperty("a");
```

APIレスポンス処理やデータ変換で多用される。

### 文字列操作

```js
const str = "hello world";

str.includes("world");
str.slice(0, 5);
str.replace("world", "JS");
```

入力値処理やログ整形などで頻出。

### 関数制御

```js
function greet(name) {
  console.log("Hello " + name);
}

greet.call(null, "Taro");
```

フレームワーク内部や高度な設計で利用される。

## 1. 結論

[[基礎 Part 20 標準メソッド|標準メソッド]]とは、JavaScriptエンジンに特別に組み込まれた魔法の機能ではない。
多くは[[基礎 Part 15 オブジェクト|オブジェクト]]の [[基礎 Part 17 プロトタイプ|prototype]] に定義されている通常のメソッドである。

JavaScriptでは、プロパティ探索時に[[基礎 Part 17 プロトタイプ|プロトタイプチェーン]]をたどるため、
自分自身に存在しないメソッドでも利用できる。

---

## 2. [[基礎 Part 15 オブジェクト|Object]].[[基礎 Part 17 プロトタイプ|prototype]] にある[[基礎 Part 20 標準メソッド|標準メソッド]]

空[[基礎 Part 15 オブジェクト|オブジェクト]]を作る。

```js
const obj = {};
```

内部構造は次のようになっている。

```
obj
  ↓
Object.prototype
  ↓
null
```

[[基礎 Part 15 オブジェクト|Object]].[[基礎 Part 17 プロトタイプ|prototype]] には次のようなメソッドが定義されている。

* toString
* hasOwnProperty
* valueOf
* isPrototypeOf
* propertyIsEnumerable

### 例

```js
const obj = { name: "Taro" };

console.log(obj.toString());
console.log(obj.hasOwnProperty("name"));
```

これらは obj 自身に定義されているわけではない。
[[基礎 Part 15 オブジェクト|Object]].[[基礎 Part 17 プロトタイプ|prototype]] にあるメソッドが[[基礎 Part 17 プロトタイプ|プロトタイプチェーン]]経由で見つかっている。

---

## 3. Array の[[基礎 Part 20 標準メソッド|標準メソッド]]

[[基礎 Part 15 オブジェクト|配列]]も[[基礎 Part 15 オブジェクト|オブジェクト]]である。

```js
const arr = [1, 2, 3];
arr.push(4);
```

内部構造は次の通り。

```
arr
  ↓
Array.prototype
  ↓
Object.prototype
  ↓
null
```

push, pop, [[基礎 Part 6 Map関数の自作|map]], filter などは Array.[[基礎 Part 17 プロトタイプ|prototype]] に定義されている。

### 確認例

```js
console.log(Array.prototype.hasOwnProperty("push"));
```

---

## 4. Function の[[基礎 Part 20 標準メソッド|標準メソッド]]

関数も[[基礎 Part 15 オブジェクト|オブジェクト]]である。

```js
function test() {}

test.call(null);
```

内部構造は次の通り。

```
test
  ↓
Function.prototype
  ↓
Object.prototype
  ↓
null
```

call, apply, bind などは Function.[[基礎 Part 17 プロトタイプ|prototype]] に定義されている。

---

## 5. [[基礎 Part 17 プロトタイプ|プロトタイプチェーン]]による探索

プロパティ参照時、JavaScriptは次の順番で探索する。

1. 自分自身にあるか確認
2. なければ [[Prototype]] を参照
3. null に到達するまで繰り返す
4. 見つからなければ undefined

擬似コードで表すと次のようになる。

```
Get(object, property):
  if object が property を持つ
    return value
  else if object.[[Prototype]] が null ではない
    return Get(object.[[Prototype]], property)
  else
    return undefined
```

---

## 6. 例外: [[基礎 Part 17 プロトタイプ|prototype]] を持たない[[基礎 Part 15 オブジェクト|オブジェクト]]

```js
const obj = Object.create(null);
```

この場合は次の構造になる。

```
obj
  ↓
null
```

そのため次はエラーになる。

```js
obj.toString(); // エラー
```

[[基礎 Part 15 オブジェクト|Object]].[[基礎 Part 17 プロトタイプ|prototype]] を継承していないためである。

---

## 7. まとめ

* [[基礎 Part 20 標準メソッド|標準メソッド]]は [[基礎 Part 17 プロトタイプ|prototype]] に定義された通常のメソッドである
* ほとんどの[[基礎 Part 15 オブジェクト|オブジェクト]]は最終的に [[基礎 Part 15 オブジェクト|Object]].[[基礎 Part 17 プロトタイプ|prototype]] を継承する
* メソッドはコピーされているのではなく、チェーンをたどって参照されている
* JavaScriptは[[基礎 Part 15 オブジェクト|オブジェクト]]の連鎖構造で設計されている言語である
