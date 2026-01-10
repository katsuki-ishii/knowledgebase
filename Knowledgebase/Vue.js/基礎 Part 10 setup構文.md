---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-01T16:01:00
---
Vue.jsのsetupは、Composition APIをより簡潔かつ効率的に記述するための構文である。Vue 3.2で正式に導入され、従来のexport default構文やsetup関数の記述を簡略化することで、開発者の負担を減らし、コードの可読性と保守性を高める目的を持つ。

---

## 1. Composition APIの背景

Vue.jsには主に2つの記述スタイルが存在する。

### Options API

- Vue 2までの主流な書き方。
- data、methods、computedなどのオプションを使ってコンポーネントを定義する。
- コンポーネントのロジックが機能単位ではなく、オプション単位で分かれるため、複雑なコンポーネントではロジックの見通しが悪くなる。

```javascript
<script>
export default {
  data() {
    return { count: 0 }
  },
  methods: {
    increment() {
      this.count++
    }
  }
}
</script>

```

### Composition API

- Vue 3で導入された新しい書き方。
- setup関数の中で状態や関数を定義し、関連するロジックをまとめて扱うことができる。
- ReactのHooksのように、ロジックを再利用可能な関数（Composable）として切り出せる。

```javascript
<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const increment = () => count.value++
    return { count, increment }
  }
}
</script>

```

---

## 2. 構文とは

Composition APIのsetup関数を簡潔に記述するための構文。内部的にはVueがビルド時にsetup関数へ変換する仕組みを持つ。

```javascript
<script setup>
import { ref } from 'vue'

const count = ref(0)
const increment = () => count.value++
</script>

<template>
  <button @click="increment">{{ count }}</button>
</template>

```

このコードは内部的に次のように変換される。

```javascript
<script>
import { ref } from 'vue'
export default {
  setup() {
    const count = ref(0)
    const increment = () => count.value++
    return { count, increment }
  }
}
</script>

```

つまり、内で定義した変数や関数は自動的にテンプレートから参照可能となり、return文を書く必要がない。

---

## 3. 特徴と利点

| 特徴 | 内容 |
| --- | --- |
| 記述量の削減 | setup関数やreturnを省略でき、コードが短くなる。 |
| 自動公開 | 定義した変数や関数が自動でテンプレートに公開される。 |
| 型推論の強化 | TypeScriptと連携すると型補完が強化される。 |
| パフォーマンス最適化 | コンパイル時にsetup関数へ変換されるため、実行時オーバーヘッドが少ない。 |
| importのスコープ化 | モジュールスコープが明確で、不要なコードがグローバルに漏れない。 |

---

## 4. 関連APIとの組み合わせ

### definePropsとdefineEmits

- 

```javascript
<script setup>
const props = defineProps({
  title: String,
  count: Number
})

const emit = defineEmits(['update'])

function increment() {
  emit('update', props.count + 1)
}
</script>

```

### defineExpose

- 親コンポーネントから子コンポーネント内のメソッドを参照させたいときに使用する。

```javascript
<script setup>
import { ref } from 'vue'

const internalState = ref('ready')
function reset() {
  internalState.value = 'reset'
}

defineExpose({ reset })
</script>

```

---

## 5. Mementoアプリでの活用例

MementoアプリのようなVue 3 + Vite構成では、を用いたコンポーネント構成が主流である。例えば、LifeCalendar.vueコンポーネントは次のように書かれていると考えられる。

```javascript
<script setup>
import { useWeeks } from '@/composables/useWeeks'
import WeekGrid from '@/components/WeekGrid.vue'

const { weeks, markWeek } = useWeeks()
</script>

<template>
  <WeekGrid :weeks="weeks" @click-week="markWeek" />
</template>

```

このように、状態とUIロジックを短く明確に記述できる点がの大きな強みである。

---

## 6. まとめ

## 関連
- [[基礎 Part 3 App.vue]]
- [[基礎 Part 34 コンパイル時マクロ]]
- [[基礎 Part 35 defineProps()]]
