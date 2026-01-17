---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-01T16:01:00
aliases: [setup構文, setup, <script setup>, script setup]
---
Vue.jsの[[基礎 Part 10 setup構文|setup]]は、Composition APIをより簡潔かつ効率的に記述するための構文である。Vue 3.2で正式に導入され、従来の[[基礎 Part 11 ESM　Import＆Export|export]] [[基礎 Part 36 props バリデーション|default]]構文や[[基礎 Part 10 setup構文|setup]]関数の記述を簡略化することで、開発者の負担を減らし、コードの可読性と保守性を高める目的を持つ。

---

## 1. Composition APIの背景

Vue.jsには主に2つの記述スタイルが存在する。

### Options API

- Vue 2までの主流な書き方。
- data、methods、[[基礎 Part 18 computed|computed]]などのオプションを使って[[基礎 Part 29 コンポーネント|コンポーネント]]を定義する。
- [[基礎 Part 29 コンポーネント|コンポーネント]]のロジックが機能単位ではなく、オプション単位で分かれるため、複雑な[[基礎 Part 29 コンポーネント|コンポーネント]]ではロジックの見通しが悪くなる。

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
- [[基礎 Part 10 setup構文|setup]]関数の中で状態や関数を定義し、関連するロジックをまとめて扱うことができる。
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

Composition APIの[[基礎 Part 10 setup構文|setup]]関数を簡潔に記述するための構文。内部的にはVueが[[コンパイルとビルド|ビルド]]時に[[基礎 Part 10 setup構文|setup]]関数へ変換する仕組みを持つ。

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
| 記述量の削減 | [[基礎 Part 10 setup構文|setup]]関数やreturnを省略でき、コードが短くなる。 |
| 自動公開 | 定義した変数や関数が自動でテンプレートに公開される。 |
| 型推論の強化 | TypeScriptと連携すると型補完が強化される。 |
| パフォーマンス最適化 | [[コンパイルとビルド|コンパイル]]時に[[基礎 Part 10 setup構文|setup]]関数へ変換されるため、実行時オーバーヘッドが少ない。 |
| [[基礎 Part 11 ESM　Import＆Export|import]]のスコープ化 | モジュールスコープが明確で、不要なコードが[[基礎 Part 33 グローバルCSS|グローバル]]に漏れない。 |

---

## 4. 関連APIとの組み合わせ

### [[基礎 Part 35 defineProps()|defineProps]]と[[基礎 Part 34 コンパイル時マクロ|defineEmits]]

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

- 親[[基礎 Part 29 コンポーネント|コンポーネント]]から子[[基礎 Part 29 コンポーネント|コンポーネント]]内のメソッドを参照させたいときに使用する。

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

MementoアプリのようなVue 3 + [[Vite HMR|Vite]]構成では、を用いた[[基礎 Part 29 コンポーネント|コンポーネント]]構成が主流である。例えば、LifeCalendar.vueコンポーネントは次のように書かれていると考えられる。

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
