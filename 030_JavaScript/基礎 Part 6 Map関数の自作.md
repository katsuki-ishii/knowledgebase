---
base: "[[JavaScript.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-06-22T19:30:00
aliases: [Map関数の自作, Map関数, map関数, map, Map]
---
## 1. [[基礎 Part 6 Map関数の自作|map関数]]の基本的な考え方

- `.map()`は配列の各要素に対して関数を適用し、新しい配列を返すメソッド。
- 今回は**自力で同じ機能をforループで実装**するのが課題。
- `fn(arr[i], i)` のように、**値とインデックスの両方**を関数に渡すのがポイント。

---

## 2. 正しい実装例

```plain text
const map = function(arr, fn) {
    const newArray = [];
    for (let i = 0; i < arr.length; i++) {
        newArray.push(fn(arr[i], i));
    }
    return newArray;
};

```

- `const`で配列を[[基礎 Part 8 宣言・代入・再代入|宣言]]し、中身だけ変更するスタイルが推奨。

---

## 3. letとconstの使い分け

- **[[基礎 Part 8 宣言・代入・再代入|再代入]]しない変数**（配列の入っている箱そのものを変えない）：`const`
- **[[基礎 Part 8 宣言・代入・再代入|再代入]]する変数**（ループカウンタなど）：`let`
- `const`でも配列の中身（push, popなど）はガンガン変更OK！箱ごと差し替えたい時だけ`let`

---

## 4. JavaScriptの関数引数の仕様

- **[[基礎 Part 10 関数の定義方法3つ|関数定義]]の引数より多く渡してもOK→余分は無視される**
- **足りなければ***`undefined`***が自動で入る**
- そのため、「引数の数に合わせてifで分ける」必要なし！
- 例：
    - `function fn(a, b) { ... }` → `fn(1)` で b は undefined
    - `function fn(a) { ... }` → `fn(1, 2)` で2番目以降は無視

---

## 5. テスト例

```plain text
const plusOne = n => n + 1;
map([1,2,3], plusOne); // [2,3,4]

const plusI = (n, i) => n + i;
map([1,2,3], plusI); // [1,3,5]

const constant = () => 42;
map([10,20,30], constant); // [42,42,42]

```

---

## 6. よくある勘違い＆今回学んだ知識

- 関数に「必要な引数だけ」渡さなきゃいけないと思いがち → JSは柔軟！
- `const`で配列やオブジェクトを[[基礎 Part 8 宣言・代入・再代入|宣言]]しても中身は自由に変更できる
- [[基礎 Part 6 Map関数の自作|map]]の再実装は「コアな配列・関数の理解」に役立つ良問

---

