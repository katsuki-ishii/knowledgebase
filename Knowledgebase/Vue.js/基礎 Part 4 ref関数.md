---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-14T00:29:00
---
# 1. ref() の基本

`ref()` はリアクティブな値を作る関数で、内部的にオブジェクトを返す。

```javascript
import { ref } from 'vue'

const price = ref(9.99)
console.log(price) // => { value: 9.99 }

price.value += 1
console.log(price.value) // => 10.99

```

Vue は `price.value` の変更を検知し、自動的にUIを再描画する。

### 特徴

- プリミティブ値（数値・文字列・真偽値など）をリアクティブ化できる
- `.value` プロパティ経由でアクセスする
- `<template>` 内では `.value` を省略できる（テンプレートアンラップ）

---

## 2. ref() のリアクティビティ確認

```javascript
<script setup>
import { ref } from 'vue'

const message = ref('Hello')

function updateMessage() {
  message.value = 'Updated!'
}
</script>

<template>
  <h1>{{ message }}</h1>
  <button @click="updateMessage">Change</button>
</template>

```

ボタンをクリックすると `message.value` が変更され、Vue が自動的に `<h1>` の表示を更新する。

---

## 3. プリミティブ型とは

プリミティブ型は JavaScript の基本的な値で、直接参照を持たない。

```javascript
let a = 1
let b = a

a = 2
console.log(b) // => 1 (bは変わらない)

```

| 種類 | 例 | 特徴 |
| --- | --- | --- |
| number | `42` | 値そのものを保持 |
| string | `'hello'` | 値がコピーされる |
| boolean | `true` | 参照ではなく値として扱われる |

プリミティブは「値渡し」であり、変更を検知できないため、そのままではリアクティブにできない。

---

## 4. オブジェクト型と監視の仕組み

オブジェクトは参照型であり、Vue は `Proxy` を使ってそのプロパティへのアクセスを監視する。

```javascript
const obj = { value: 1 }

const proxy = new Proxy(obj, {
  get(target, key) {
    console.log('GET', key)
    return target[key]
  },
  set(target, key, val) {
    console.log('SET', key, val)
    target[key] = val
    return true
  }
})

console.log(proxy.value) // GET value
proxy.value = 2           // SET value 2

```

Vue のリアクティブシステムは、このようにプロパティの読み書きを横取りして、変更を検知している。

`ref()` はプリミティブ値をオブジェクト（Proxyで監視可能）で包むことで、リアクティブ化を実現している。

---

## 5. テンプレート内の `.value` 省略

`<template>` 内では、Vue が自動的に `.value` を展開してくれる。

```plain text
<script setup>
import { ref } from 'vue'

const count = ref(0)
function increment() {
  count.value++
}
</script>

<template>
  <p>Count: {{ count }}</p> <!-- 内部的には count.value に変換される -->
  <button @click="increment">+1</button>
</template>

```

### 省略できないケース

- `<script>` 内（JavaScript コード内）
- `reactive()` の中にある `ref`

```javascript
import { reactive, ref } from 'vue'

const state = reactive({
  price: ref(100)
})

```

```plain text
{{ state.price.value }} <!-- reactive内のrefは展開されない -->

```

---

## 6. まとめ

| 概念 | 説明 |
| --- | --- |
| `ref()` | プリミティブ値をオブジェクトに包んでリアクティブ化する関数 |
| `.value` | リアクティブな値の実体を持つプロパティ |
| プリミティブ型 | 値そのもの。変更を検知できない |
| オブジェクト型 | 参照を持つ。Proxyで監視可能 |
| テンプレートアンラップ | `<template>` 内で `.value` を省略できる仕組み |

## 関連
- [[基礎 Part 5 reactive関数]]
- [[基礎 Part 8 reactiveとrefの併用]]
- [[基礎 Part 9 オブジェクト内のref]]
