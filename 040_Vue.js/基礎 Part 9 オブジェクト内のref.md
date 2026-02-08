---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-30T08:27:00
aliases: [オブジェクト内のref, オブジェクト内ref, toRefs]
---
## おさらい: [[基礎 Part 4 ref関数|ref()]]とは

`ref()`はVue 3のComposition APIで使われる関数で、値を[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]（反応的）にするために使用する。内部的には`{ value: ... }`というオブジェクトを返し、Vueは`.value`プロパティの変更を検知してUIを自動更新する。

```javascript
import { ref } from 'vue'
const count = ref(0)

```

---

### 普通のオブジェクトに[[基礎 Part 4 ref関数|ref]]が含まれる場合

```javascript
const state = {
  count: ref(0),
  message: ref('Hello')
}

```

この場合、`state`自体は[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]ではない。つまり、`state.count`は「[[基礎 Part 4 ref関数|ref]]オブジェクトそのもの」であり、Vueの[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]追跡の対象にはならない。

- 値の更新は可能：`state.count.value++`
- しかしUIは自動で更新されない：`state`が[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]ではないため

---

### [[基礎 Part 5 reactive関数|reactive()]]を使う場合

```javascript
import { reactive, ref } from 'vue'

const state = reactive({
  count: ref(0),
  message: ref('Hello')
})

```

`reactive()`で包むと、Vueは[[基礎 Part 4 ref関数|ref]]を自動的にアンラップする（`.value`を自動的に外す）。

テンプレートでは次のように書ける：

```plain text
<template>
  <div>{{ state.count }}</div>
</template>

```

---

### [[基礎 Part 9 オブジェクト内のref|toRefs]]()を使う場合

`reactive`なオブジェクトを個別の[[基礎 Part 4 ref関数|ref]]に分解したい場合に使用する。

```javascript
import { reactive, toRefs } from 'vue'

const state = reactive({ count: 0, message: 'Hello' })
const { count, message } = toRefs(state)

```

これで`count`と`message`が[[基礎 Part 4 ref関数|ref]]として扱えるようになる。

---

### 最初から[[基礎 Part 4 ref関数|ref]]で全体を包む方法

オブジェクト全体を[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]にしたい場合は、`ref()`でオブジェクトを直接包むのが最もシンプル。

```javascript
import { ref } from 'vue'

const state = ref({
  count: 0,
  message: 'Hello'
})

```

- `state.value`全体が[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]になる
- 内部プロパティ（`state.value.count`など）も自動的に追跡される
- UIも正しく再描画される

Vueは`ref({ ... })`にオブジェクトが渡された場合、内部的に`reactive()`を適用している

---

### 各パターンの比較

| パターン | [[基礎 Part 20 リアクティブエフェクト|リアクティブ]]になる範囲 | 書きやすさ | 推奨度 |
| --- | --- | --- | --- |
| `{ count: ref(0) }` | 各[[基礎 Part 4 ref関数|ref]]の中身のみ | 複雑 (`.value`多い) | ✗ |
| `reactive({ count: ref(0) })` | 全体＋自動アンラップ | やや冗長 | ○ |
| `ref({ count: 0 })` | 全体 | シンプルで安全 | ◎ |

---

### 実践的な使い分け

- シンプルな状態管理（フォーム、モーダルなど）：`ref({ ... })`
- 複雑な状態管理（[[基礎 Part 16 Vueコード規約|Pinia]]やVuexのようなストア構造）：`reactive()`

---

### まとめ

- 普通のオブジェクトに`ref()`を入れても、オブジェクト自体は[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]にならない
- `reactive()`を使うと、内部の`ref`は自動アンラップされる
- オブジェクト全体を[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]にしたいなら、`ref({})`で包むのが最もシンプルで確実
