---
base: "[[JavaScript.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2026-02-15T12:00:00
tags:
  - オブジェクト
  - プリミティブ
  - 型
aliases:
  - オブジェクト
  - JavaScriptのオブジェクト
  - object
---

# [[基礎 Part 15 オブジェクト|JavaScriptのオブジェクト]]

---

## [[基礎 Part 15 オブジェクト|オブジェクト]]とは

[[基礎 Part 15 オブジェクト|JavaScriptのオブジェクト]]は、**プロパティ（キー）とバリュー（値）のペアを持つデータ構造**である。

```javascript
const person = {
  name: 'かつき',    // プロパティ: バリュー
  age: 26,
  city: 'Musashi-Kosugi'
};
```

### [[基礎 Part 15 オブジェクト|オブジェクト]]の本質的な特徴

1. バリューには何でも入る（プリミティブ、関数、[[基礎 Part 15 オブジェクト|オブジェクト]]、配列）
2. 動的に変更できる（プロパティの追加・削除が可能）
3. 参照型である（値渡しではなく参照渡し）

---

## バリューに入るもの

[[基礎 Part 15 オブジェクト|オブジェクト]]のバリューには、JavaScriptのあらゆる型を格納できる。

```javascript
const obj = {
  // プリミティブ型
  str: 'hello',
  num: 42,
  bool: true,
  nothing: null,
  undef: undefined,

  // 関数
  func: function() { return 'hello'; },
  arrow: () => 'world',

  // オブジェクト（ネスト）
  nested: {
    inner: 'value'
  },

  // 配列（これもオブジェクト）
  arr: [1, 2, 3]
};
```

---

## JavaScriptの型システム

JavaScriptの全てのデータは、プリミティブ型と[[基礎 Part 15 オブジェクト|オブジェクト]]型の2つに分類される。

### プリミティブ型（7種類）

1. `string` - 文字列
2. `number` - 数値
3. `bigint` - 大きな整数
4. `boolean` - 真偽値
5. `undefined` - 未定義
6. `null` - 空（※typeofでは[[基礎 Part 15 オブジェクト|object]]と表示されるバグ）
7. `symbol` - 一意の識別子

### [[基礎 Part 15 オブジェクト|オブジェクト]]型

プリミティブ型以外の全て。

- `Object` - 通常の[[基礎 Part 15 オブジェクト|オブジェクト]]
- `Array` - 配列
- `Function` - 関数
- `Date` - 日付
- `RegExp` - 正規表現
- その他

```javascript
// 型の確認
typeof 'hello';       // "string"
typeof 42;            // "number"
typeof true;          // "boolean"
typeof undefined;     // "undefined"
typeof null;          // "object" （JavaScriptのバグ）
typeof {};            // "object"
typeof [];            // "object"
typeof function(){}; // "function"
```

---

## プリミティブと[[基礎 Part 15 オブジェクト|オブジェクト]]の違い

### 1. 値渡し vs 参照渡し

**プリミティブ型は値渡し**

```javascript
let a = 10;
let b = a;  // 値がコピーされる
b = 20;

console.log(a);  // 10（変わらない）
console.log(b);  // 20
```

メモリイメージ：

```
a → [10]  // 独立した値
b → [20]  // 独立した値
```

**[[基礎 Part 15 オブジェクト|オブジェクト]]型は参照渡し**

```javascript
let obj1 = { value: 10 };
let obj2 = obj1;  // 参照がコピーされる（同じオブジェクトを指す）
obj2.value = 20;

console.log(obj1.value);  // 20（変わっている）
console.log(obj2.value);  // 20
```

メモリイメージ：

```
obj1 → [0x001] ┐
               ├→ { value: 20 }  // 同じオブジェクト
obj2 → [0x001] ┘
```

---

### 2. ミュータブル vs イミュータブル

**プリミティブはイミュータブル（不変）**

```javascript
let str = 'hello';
str[0] = 'H';  // 変更できない
console.log(str);  // "hello"（変わらない）

// 新しい値を作ることはできる
str = 'Hello';  // これはOK（新しい値を代入）
```

**[[基礎 Part 15 オブジェクト|オブジェクト]]はミュータブル（可変）**

```javascript
let arr = ['h', 'e', 'l', 'l', 'o'];
arr[0] = 'H';  // 変更できる
console.log(arr);  // ['H', 'e', 'l', 'l', 'o']
```

---

### 3. メモリの格納場所

```
スタック領域（Stack）
├─ プリミティブ値を直接格納
└─ オブジェクトへの参照（アドレス）を格納

ヒープ領域（Heap）
└─ オブジェクトの実体を格納
```

---

## プリミティブと[[基礎 Part 15 オブジェクト|オブジェクト]]の関係

### プリミティブは「[[基礎 Part 15 オブジェクト|オブジェクト]]になりうる」

プリミティブ値は、必要なときだけ一時的に[[基礎 Part 15 オブジェクト|オブジェクト]]化される。これを**ボクシング**と呼ぶ。

```javascript
const age = 26;  // プリミティブとして保存される

// プロパティやメソッドにアクセスすると...
age.toString();  // この瞬間だけ new Number(26) に変換される
                 // toString() 実行後、すぐ破棄される

console.log(typeof age);  // "number"（プリミティブのまま）
```

実際の動作イメージ：

```javascript
// 書いたコード
age.toString();

// 内部で起きていること（疑似コード）
const temp = new Number(age);  // 一時的にラップ
const result = temp.toString(); // メソッド実行
// temp は即座に破棄される
return result;
```

---

### プリミティブにプロパティを追加できない理由

```javascript
const age = 26;

// プリミティブにプロパティを追加しようとすると...
age.customProp = 'test';

console.log(age.customProp);  // undefined
// なぜ？一時オブジェクトに追加 → 即破棄されるから
```

```javascript
// 明示的にオブジェクト化すると保持できる
const ageObj = new Number(26);
ageObj.customProp = 'test';

console.log(ageObj.customProp);  // 'test'
console.log(typeof ageObj);      // "object"
```

---

### なぜこのような仕様なのか

**メモリ効率のため**である。プリミティブ値は軽量（数バイト）だが、[[基礎 Part 15 オブジェクト|オブジェクト]]は重い（プロパティ管理のオーバーヘッドがある）。

```javascript
// プリミティブなら軽い
const numbers = [];
for (let i = 0; i < 100000; i++) {
  numbers.push(i);  // 各要素が数バイト
}

// 全てオブジェクトだと重い
const numberObjects = [];
for (let i = 0; i < 100000; i++) {
  numberObjects.push(new Number(i));  // 各要素が数十バイト以上
}
```

---

## 関数は[[基礎 Part 15 オブジェクト|オブジェクト]]である

JavaScriptでは、**関数は最初から[[基礎 Part 15 オブジェクト|オブジェクト]]**として扱われる。プリミティブのように「必要なときだけ」ではなく、定義した瞬間から[[基礎 Part 15 オブジェクト|オブジェクト]]である。

### 関数は[[基礎 Part 15 オブジェクト|オブジェクト]]の証明

```javascript
function greet() {
  console.log('Hello');
}

// 関数にプロパティを追加できる
greet.description = '挨拶用の関数';
greet.version = 1.0;

console.log(greet.description);  // '挨拶用の関数'
console.log(typeof greet);       // "function"（内部的にはオブジェクト）
```

プリミティブと違って、**追加したプロパティが保持される**。

---

### 関数の標準プロパティ

関数を定義すると、自動的にいくつかのプロパティが付与される。

```javascript
function add(a, b) {
  return a + b;
}

console.log(add.name);       // "add" （関数名）
console.log(add.length);     // 2 （引数の個数）
console.log(add.toString()); // "function add(a, b) { return a + b; }"
```

#### 1. name プロパティ

関数の名前を文字列で返す。

```javascript
function greet() {}
console.log(greet.name);  // "greet"

// 無名関数（変数に代入）
const func = function() {};
console.log(func.name);  // "func"（変数名が入る）

// アロー関数
const arrow = () => {};
console.log(arrow.name);  // "arrow"

// オブジェクトのメソッド
const obj = {
  method() {}
};
console.log(obj.method.name);  // "method"
```

#### 2. length プロパティ

引数の個数を返す。ただし、デフォルト値やrestパラメータは含まれない。

```javascript
function func1(a, b, c) {}
console.log(func1.length);  // 3

function func2(a, b = 10, c) {}
console.log(func2.length);  // 1（bのデフォルト値以降はカウントされない）

function func3(a, ...rest) {}
console.log(func3.length);  // 1（restパラメータはカウントされない）

function func4() {}
console.log(func4.length);  // 0
```

#### 3. prototype プロパティ

関数[[基礎 Part 15 オブジェクト|オブジェクト]]には`prototype`プロパティが存在する。これは[[基礎 Part 15 オブジェクト|オブジェクト]]指向プログラミングの継承に使われる（詳細は別ノート参照）。

注意：[[基礎 Part 10 関数の定義方法3つ|アロー関数]]には`prototype`プロパティが存在しない。

```javascript
function normalFunc() {}
console.log(normalFunc.prototype);  // {}

const arrowFunc = () => {};
console.log(arrowFunc.prototype);  // undefined
```

---

### 関数は「第一級[[基礎 Part 15 オブジェクト|オブジェクト]]」

関数が[[基礎 Part 15 オブジェクト|オブジェクト]]であるため、以下のことが可能になる。

```javascript
// 1. 変数に代入できる
const myFunc = function() { console.log('Hi'); };

// 2. 引数として渡せる
function execute(func) {
  func();
}
execute(myFunc);

// 3. 戻り値として返せる
function createGreeter() {
  return function() { console.log('Hello'); };
}
const greeter = createGreeter();

// 4. 配列に入れられる
const functions = [
  function() { console.log('A'); },
  function() { console.log('B'); }
];
functions[0]();  // "A"

// 5. オブジェクトのプロパティとして格納できる
const obj = {
  method: function() { console.log('method'); }
};
obj.method();
```

---

### console.log vs console.dir

関数をconsole.logで表示すると、[[基礎 Part 15 オブジェクト|オブジェクト]]としてではなく関数として表示される。

```javascript
function greet() {
  console.log('Hello');
}

greet.customProp = 'カスタムプロパティ';

console.log(greet);
// [Function: greet]
// ↑ 追加したプロパティは見えない

console.dir(greet);
// [Function: greet] { customProp: 'カスタムプロパティ' }
// ↑ オブジェクトとして表示され、プロパティが見える
```

これは`console.log()`が内部で`.toString()`を呼び、関数の場合はソースコードを文字列として返すため。[[基礎 Part 15 オブジェクト|オブジェクト]]としての内部構造を見たい場合は`console.dir()`を使う。

---

## [[基礎 Part 15 オブジェクト|オブジェクト]]の作り方

JavaScriptでは複数の方法で[[基礎 Part 15 オブジェクト|オブジェクト]]を作成できる。

```javascript
// 1. オブジェクトリテラル（最も一般的）
const obj1 = { name: 'test', age: 26 };

// 2. new Object()
const obj2 = new Object();
obj2.name = 'test';
obj2.age = 26;

// 3. Object.create()
const proto = { greet: function() { console.log('Hello'); } };
const obj3 = Object.create(proto);
obj3.name = 'test';
```

### 動的にプロパティを操作する

```javascript
const user = {};

// プロパティの追加
user.name = 'かつき';
user.age = 26;

// プロパティの削除
delete user.age;

// プロパティの存在確認
console.log('name' in user);  // true
console.log('age' in user);   // false

console.log(user);  // { name: 'かつき' }
```

---

## まとめ：[[基礎 Part 15 オブジェクト|オブジェクト]]の全体像

```
JavaScriptの世界
├─ プリミティブ型（7種類）
│  ├─ 値渡し
│  ├─ イミュータブル
│  ├─ スタック領域に格納
│  └─ 必要時のみ一時的にオブジェクト化
│
└─ オブジェクト型
   ├─ 参照渡し
   ├─ ミュータブル
   ├─ ヒープ領域に格納
   └─ 種類
      ├─ Object（通常のオブジェクト）
      ├─ Array（配列）
      ├─ Function（関数）※最初からオブジェクト
      ├─ Date
      ├─ RegExp
      └─ その他
```

### プリミティブと[[基礎 Part 15 オブジェクト|オブジェクト]]の比較表

| 特徴 | プリミティブ | [[基礎 Part 15 オブジェクト|オブジェクト]] |
|------|------------|------------|
| 渡し方 | 値渡し | 参照渡し |
| 変更可否 | イミュータブル（不変） | ミュータブル（可変） |
| プロパティ | 一時的にラップされる | 最初から持っている |
| メモリ | スタック領域 | ヒープ領域 |
| 等価比較 | 値で比較 | 参照（アドレス）で比較 |

### 関数の特徴

- 関数は**最初から[[基礎 Part 15 オブジェクト|オブジェクト]]**である（プリミティブではない）
- 呼び出し可能 + プロパティ保持が可能
- デフォルトで`name`、`length`、`prototype`プロパティを持つ
- 第一級[[基礎 Part 15 オブジェクト|オブジェクト]]として、[[AI激動期における「定数」思考|変数]]への[[基礎 Part 8 宣言・代入・再代入|代入]]・引数への渡し・戻り値として使用可能
- [[基礎 Part 10 関数の定義方法3つ|アロー関数]]は`prototype`を持たない

---

## 参考：便利なconsoleメソッド

```javascript
// console.dir() - オブジェクトの内部構造を表示
console.dir(object, { depth: null });

// console.table() - 配列やオブジェクトを表形式で表示
console.table([{ name: 'かつき', age: 26 }]);

// console.time() / console.timeEnd() - 処理時間計測
console.time('処理');
// ...処理...
console.timeEnd('処理');

// console.trace() - スタックトレース表示
console.trace('ここに至るまでの呼び出し');

// console.group() / console.groupEnd() - ログのグループ化
console.group('グループ名');
console.log('内容1');
console.log('内容2');
console.groupEnd();
```
