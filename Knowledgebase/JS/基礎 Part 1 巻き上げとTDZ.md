---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-06-17T23:00:00
aliases: [巻き上げとTDZ, 巻き上げ, TDZ, 巻き上げとtdz, 巻き上げとTdz]
---

## 1. 変数[[基礎 Part 8 宣言・代入・再代入|宣言]]・初期化・[[基礎 Part 1 巻き上げとTDZ|TDZ]]（Temporal Dead Zone）

- **[[基礎 Part 8 宣言・代入・再代入|宣言]]（declaration）**: 変数名をスコープに登録するだけ
    - 例: `let a;`
- **初期化（initialization）**: 変数に最初の値を入れる
    - 例: `let a = 1;`（[[基礎 Part 8 宣言・代入・再代入|宣言]]＋初期化）
    - 例: `let a; a = 1;`（[[基礎 Part 8 宣言・代入・再代入|宣言]]→初期化）
- **[[基礎 Part 1 巻き上げとTDZ|TDZ]]**: [[基礎 Part 8 宣言・代入・再代入|宣言]]はされているが、初期化前で使えない期間
    - `let`/`const`でのみ発生
    - アクセスすると `ReferenceError`
- **varとの違い**: `var`は[[基礎 Part 1 巻き上げとTDZ|巻き上げ]]＋初期化(`undefined`)されるので[[基礎 Part 1 巻き上げとTDZ|TDZ]]がない

---

## 2. Hoisting（[[基礎 Part 1 巻き上げとTDZ|巻き上げ]]）とスコープ解決

- **[[基礎 Part 1 巻き上げとTDZ|巻き上げ]]（hoisting）**:
    - 変数や関数の[[基礎 Part 8 宣言・代入・再代入|宣言]]がスコープの先頭に引っ越しされるように動作
- **function[[基礎 Part 8 宣言・代入・再代入|宣言]]**: [[基礎 Part 1 巻き上げとTDZ|巻き上げ]]＋初期化もされる → 定義前でも呼び出せる
- **[[基礎 Part 10 関数の定義方法3つ|関数式]]（const/letに[[基礎 Part 8 宣言・代入・再代入|代入]]）**: [[基礎 Part 1 巻き上げとTDZ|巻き上げ]]されるのは[[基礎 Part 8 宣言・代入・再代入|宣言]]だけ、初期化されるまで[[基礎 Part 1 巻き上げとTDZ|TDZ]] → 定義前に使うとエラー
- **スコープ解決**: どの変数・関数がどのスコープに属するかを事前に決定

---

## 3. [[基礎 Part 10 関数の定義方法3つ|関数の定義方法]]

- **function[[基礎 Part 8 宣言・代入・再代入|宣言]]**:
```javascript
function greet() {
  return "Hello";
}
greet();

```
- **[[基礎 Part 10 関数の定義方法3つ|関数式]]**（変数や定数に[[基礎 Part 8 宣言・代入・再代入|代入]]）:
```javascript
const greet = function() {
  return "Hello";
};
greet();

```
- **名前付き[[基礎 Part 10 関数の定義方法3つ|関数式]]**: 主に再帰やデバッグ用
```javascript
const fact = function factorial(n) {
  if(n === 0) return 1;
  return n * factorial(n - 1);
};

```
    - 外から`factorial`は使えず、中からだけ有効

---

## 4. [[基礎 Part 2クロージャ|クロージャ]]（[[基礎 Part 2クロージャ|Closure]]）

- 「関数が作られたときのスコープ（変数）を記憶し続ける仕組み」
```javascript
function createCounter() {
  let count = 0;
  return function() {
    count++;
    return count;
  };
}
const countUp = createCounter();
countUp(); // 1
countUp(); // 2

```
- 関数の外で[[基礎 Part 8 宣言・代入・再代入|宣言]]した変数は、呼び出しごとに初期化されるが、
[[基礎 Part 2クロージャ|クロージャ]]で包むと「記憶され続ける」

---

## 5. JavaScriptエンジンのコード実行プロセス

1. **Parsing（構文解析）**
    - コードを読み込んでエラーや構造を確認
2. **スコープ解決・Hoisting・[[基礎 Part 1 巻き上げとTDZ|TDZ]]**
    - 変数や関数のスコープ、[[基礎 Part 1 巻き上げとTDZ|TDZ]]の決定
3. **実行フェーズ**
    - 値の[[基礎 Part 8 宣言・代入・再代入|代入]]や関数の呼び出しを実際に処理

---

## 6. V8エンジンとは？

- Google製の超高速JavaScriptエンジン（Chrome/Node.jsなどで採用）
- ソースコードをバイトコード→機械語に変換して実行
- 構文解析→バイトコード（Ignition）→プロファイリング→JIT最適化（TurboFan）
- 他の有名エンジン：SpiderMonkey（Firefox）、JavaScriptCore（Safari）

---

## 7. JIT（Just-In-Time Compilation）

- コードを「実行しながら、必要な部分だけ機械語に最適化」する仕組み
- 最初はバイトコードで実行、よく使われる部分だけJITで爆速化
- 柔軟かつ高パフォーマンス（型が安定している箇所に特に強い）
- Deoptimization（型が崩れたら最適化解除）も行う

---

## 8. よく出る落とし穴・覚えておきたいポイント

- [[基礎 Part 1 巻き上げとTDZ|TDZ]]内での変数アクセスで `ReferenceError`
- [[基礎 Part 10 関数の定義方法3つ|関数宣言]]と[[基礎 Part 10 関数の定義方法3つ|関数式]]の[[基礎 Part 1 巻き上げとTDZ|巻き上げ]]挙動の違い
- [[基礎 Part 2クロージャ|クロージャ]]を使うことで変数の状態を「記憶」できる
- V8エンジンの仕組み・JITが高速化のカギ
- コードは「解析→準備→実行」という流れ

---

# 具体例・図解集（あとで見返しやすいサンプル）

## [[基礎 Part 1 巻き上げとTDZ|TDZ]]（Temporal Dead Zone）のイメージ

```javascript
console.log(x); // ReferenceError: Cannot access 'x' before initialization
let x = 10;

```

**図解イメージ：**

```plain text
┌───────┐
│ 名前: x│  ← 宣言はスコープにあるが、初期化前は使えない（TDZ）
└───────┘

```

---

## Hoistingの違い（function[[基礎 Part 8 宣言・代入・再代入|宣言]]と[[基礎 Part 10 関数の定義方法3つ|関数式]]）

```javascript
// function宣言：巻き上げあり
sayHello(); // OK！
function sayHello() { console.log('hello'); }

// 関数式（const/let）：巻き上げなし
sayHi(); // ReferenceError
const sayHi = function() { console.log('hi'); }

```

---

## [[基礎 Part 2クロージャ|クロージャ]]の典型例

```javascript
function createCounter() {
  let count = 0;
  return function() {
    count++;
    return count;
  }
}
const counter = createCounter();
counter(); // 1
counter(); // 2

```

**解説：**

- `createCounter`で作った関数は、count変数のスコープを「記憶」している
- だから毎回カウントがリセットされず増えていく

---

## よくあるエラー例

- [[基礎 Part 1 巻き上げとTDZ|TDZ]]: `ReferenceError: Cannot access 'x' before initialization`
- [[基礎 Part 1 巻き上げとTDZ|巻き上げ]]：[[基礎 Part 10 関数の定義方法3つ|関数式]]でのReferenceError
- [[基礎 Part 2クロージャ|クロージャ]]：新しいcounterを作るとカウントはリセットされる

---


