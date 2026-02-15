---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-06T17:51:00
aliases: [computed, Computed, 計算プロパティ]
---
### 1. [[基礎 Part 19 computed|computed]]とは何か

[[基礎 Part 19 computed|computed]]はVueの機能で、ある値から計算して得られる「派生データ」を自動的に最新の状態に保つための仕組みである。テンプレートの中に計算式を書く代わりに、あらかじめ計算結果を準備しておく役割を担う。

### 2. なぜ[[基礎 Part 19 computed|computed]]が必要なのか

テンプレート内で計算を直接書くと読みづらくなり、同じ計算を何度も書くことになる。[[基礎 Part 19 computed|computed]]に計算を集約することで、コードの意味が明確になり、保守しやすくなる。

```javascript
<p>{{ score > 3 ? 'Good' : 'Bad' }}</p>

```

このようにテンプレート内で条件式を書くと複雑化しやすい。

### 3. [[基礎 Part 4 ref関数|ref]]で同じことをするとどうなるのか

[[基礎 Part 4 ref関数|ref]]は値を保持するための仕組みで、式を自動で再評価するものではない。以下のように書いても自動更新は行われない。

```javascript
const score = ref(0)
const evaluation = ref(score.value > 3 ? 'Good' : 'Bad')

```

この場合、evaluationは最初の計算結果のままで、scoreが変わっても更新されない。

### 4. [[基礎 Part 19 computed|computed]]を使うとどう変わるか

```javascript
const score = ref(0)
const evaluation = computed(() => {
  return score.value > 3 ? 'Good' : 'Bad'
})

```

scoreが変わるたびにevaluationも自動で最新化される。これが[[基礎 Part 19 computed|computed]]の最も重要な利点である。

### 5. 具体的な画面の例

以下のコードではボタンを押すとscoreが増え、それに応じてevaluationが自動的に切り替わる。

```javascript
<script setup>
import { ref, computed } from 'vue'

const score = ref(0)
const evaluation = computed(() => {
  return score.value > 3 ? 'Good' : 'Bad'
})
</script>

<template>
  <p>評価: {{ evaluation }}</p>
  <p>スコア: {{ score }}</p>
  <button @click="score++">+1</button>
</template>

```

### 6. [[基礎 Part 19 computed|computed]]はキャッシュされる

[[基礎 Part 19 computed|computed]]の計算は必要になったときだけ実行され、値が変わらなければ再計算されない。これにより無駄な処理が減り、動作が軽くなる。

### 7. [[基礎 Part 19 computed|computed]]の別の書き方（getterとsetter）

setterを使うと[[基礎 Part 19 computed|computed]]に値を書き込むこともできる。

```javascript
const firstName = ref('Taro')
const lastName = ref('Yamada')

const fullName = computed({
  get() {
    return firstName.value + ' ' + lastName.value
  },
  set(newValue) {
    const [first, last] = newValue.split(' ')
    firstName.value = first
    lastName.value = last
  }
})

```

```javascript
fullName.value = 'Hanako Suzuki'

```

このように書くとfirstNameとlastNameも更新される。

### 8. [[基礎 Part 19 computed|computed]]が必要となる典型的な例

- 条件で表示内容を変える
- 数値の変換（例: メートル→キロメートル）
- 入力値から別の表示値を作る
- 配列のフィルタリングやソート結果を表示する

例: フィルタリング

```javascript
const items = ref(['apple', 'banana', 'melon'])
const filtered = computed(() => {
  return items.value.filter(item => item.includes('a'))
})

```

### まとめ

[[基礎 Part 19 computed|computed]]は、[[基礎 Part 4 ref関数|ref]]などの[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]値から計算される派生データを自動で最新の状態に保つための仕組みである。[[基礎 Part 4 ref関数|ref]]だけでは自動更新されず、テンプレートに直接計算を書くと可読性が下がるため、計算処理は[[基礎 Part 19 computed|computed]]に集約することでコードを明確に保てる。
