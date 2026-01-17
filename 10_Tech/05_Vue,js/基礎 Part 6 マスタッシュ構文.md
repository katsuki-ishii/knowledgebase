---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-15T01:55:00
aliases: [マスタッシュ構文, マスタッシュ, Mustache, mustache, {{}}]
---
### 概要

Vueの`{{}}`は「[[基礎 Part 6 マスタッシュ構文|マスタッシュ構文]]（[[基礎 Part 6 マスタッシュ構文|Mustache]] Syntax）」と呼ばれ、テンプレート内でJavaScriptの式をHTMLに埋め込むために使用する。主に変数や式の評価結果を表示するために使われる。

```javascript
<template>
  <p>{{ message }}</p>
</template>

<script setup>
const message = 'こんにちは、Vue'
</script>

```

上記では`message`の値がHTML上に出力される。

---

### 仕組み

テンプレートはVueによってレンダリング関数に変換される。`{{ message }}`は内部的に`_toDisplayString(message)`のように変換され、[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]データが変化すると自動的に再描画される。

---

### 使用できる内容

`{{}}`の中では**JavaScriptの式**を記述できる。文（if文やfor文）は使用できない。

```plain text
<p>{{ user.name.toUpperCase() }}</p>
<p>{{ items.length > 0 ? 'アイテムあり' : '空' }}</p>
<p>{{ new Date().toLocaleDateString() }}</p>

```

- 有効: 変数、関数呼び出し、[[基礎 Part 12 三項演算子|三項演算子]]など
- 無効: if文、for文、[[基礎 Part 8 宣言・代入・再代入|代入]]式

---

### HTMLの扱い

`{{}}`はデフォルトでHTMLをエスケープ（タグを文字列として扱う）する。

```plain text
<p>{{ htmlText }}</p>

```

`htmlText`が`'<strong>太字</strong>'`の場合、タグはそのまま文字列として表示される。

HTMLをそのまま描画したい場合は`v-html`[[基礎 Part 16 ディレクティブ|ディレクティブ]]を使用する。

```plain text
<p v-html="htmlText"></p>

```

---

### [[基礎 Part 19 リアクティブエフェクト|リアクティブ]]データとの連動

`{{}}`は[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]データと連動し、値が変わると自動でUIが更新される。

```plain text
<template>
  <p>{{ count }}</p>
  <button @click="count++">＋</button>
</template>

<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

```

`count`が変化するたびに表示が再描画される。

---

### まとめ

| 項目 | 内容 |
| --- | --- |
| 構文 | `{{ expression }}` |
| 名称 | [[基礎 Part 6 マスタッシュ構文|マスタッシュ構文]]（[[基礎 Part 6 マスタッシュ構文|Mustache]] Syntax） |
| 内容 | JavaScriptの式を評価して出力 |
| 自動更新 | [[基礎 Part 19 リアクティブエフェクト|リアクティブ]]データと連動して更新される |
| HTMLの扱い | 自動エスケープされる |
| HTMLを有効化 | `v-html`を使用 |

## 関連
- [[基礎 Part 12 v-bind]]
- [[基礎 Part 7 v-html]]
