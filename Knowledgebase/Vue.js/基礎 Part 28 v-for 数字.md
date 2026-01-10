---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-07T23:41:00
---
# v-for で数字を扱う基礎まとめ

## 1. v-for は Vue の文法

`v-for` は Vue が提供する繰り返し処理の文法。JavaScript ではなく Vue 独自の仕組み。

```plain text
<li v-for="item in items">{{ item }}</li>

```

---

## 2. 数字を使った v-for の基本動作

Vue では、次のように数字を直接渡すと、その回数分だけ 1 から順番に値を生成する。

```plain text
<div v-for="n in 5">{{ n }}</div>

```

出力

```plain text
1
2
3
4
5

```

---

## 3. 値 n と インデックス i の関係

`v-for="(n, i) in 5"` とすると、Vue は内部で次のようなイメージの配列を作る。

図解

```plain text
インデックス:  0   1   2   3   4
値 n:          1   2   3   4   5
---------------------------------
v-for は (n, i) を順に渡していく

```

- n は Vue が生成する値（1 からスタート）
- i は配列と同じく 0 から始まるインデックス（JavaScript の仕様準拠）

例

```plain text
<div v-for="(n, i) in 5" :key="i">
  n = {{ n }}, i = {{ i }}
</div>

```

---

## 4. 0 から始めたい場合

n をそのまま使うと 1 から始まるため、0 始まりにしたい場合は調整する。

```plain text
<div v-for="n in 5" :key="n">{{ n - 1 }}</div>

```

---

## 5. 実務でよく使うパターン

### 5-1. ローディング用のダミーカード

```plain text
<div v-for="n in 5" :key="n" class="skeleton-card"></div>

```

### 5-2. 星評価表示

```plain text
<span v-for="n in 5" :key="n">★</span>

```

評価値を反映させる例

```plain text
<span v-for="n in 5" :key="n" :class="{ active: n <= rating }">★</span>

```

### 5-3. ページネーション（1〜n の番号）

```plain text
<button v-for="n in totalPages" :key="n" @click="go(n)">
  {{ n }}
</button>

```

### 5-4. 帳票などの空行作成

```plain text
<tr v-for="n in 10" :key="n">
  <td></td><td></td><td></td>
</tr>

```

### 5-5. 配列を作らずループだけしたいケース

配列を生成する必要はなく、数字だけでよい。

```plain text
<div v-for="n in 5" :key="n"></div>

```

---

## 6. key の指定

繰り返しでは key を付けるのが Vue の基本ルール。

```plain text
<div v-for="n in 5" :key="n">{{ n }}</div>

```

---

## 7. Vue の仕様と JavaScript の仕様の整理

| 内容 | Vue の仕様 | JavaScript の仕様 |
| --- | --- | --- |
| `v-for` | ○ | × |
| 数字を直接渡すと 1〜n を生成 | ○ | × |
| インデックスが 0 から始まる | △（JS に合わせている） | ○ |
| `(n, i)` の形式 | ○ | × |

---

## まとめ

- `v-for="n in 数字"` は Vue が提供する便利な繰り返し機能
- 数字を渡すと 1 からその数字までの値が自動生成される
- `(n, i)` の i は 0 スタートで、JavaScript の配列の考え方に基づく
- 実務ではローディング、星評価、ページネーションなどで頻出

必要があれば、次は数字を使う v-for のアンチパターンや、より複雑な応用例もまとめる

## 関連
- [[基礎 Part 26 v-for 配列]]
- [[基礎 Part 27 v-for オブジェクト]]
