---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-07T17:05:00
aliases: [class指定, class, class binding, クラス指定]
---
# Vue における [[基礎 Part 23 class指定|class]] 指定

## 1. HTML の [[基礎 Part 23 class指定|class]] をどう扱うかという前提

Vue では UI を状態から自動生成するという思想を採用している。そのため DOM を直接操作せず、データを変えれば UI が変わる仕組みが用意されている。

```plain text
状態 → UI を自動で構築

```

文字列で [[基礎 Part 23 class指定|class]] を手動生成すると、条件が入るたびに煩雑になり破綻しやすい。Vue の [[基礎 Part 23 class指定|class]] バインディングはその問題を回避するためにある。

## 2. [[基礎 Part 23 class指定|class]] バインディングの3つの主要な書き方

### 2.1 文字列指定（最も単純）

固定の [[基礎 Part 23 class指定|class]] を書く場合はこれで十分。

```javascript
<div class="p-4 bg-blue-500 text-white">Hello</div>

```

### しかし条件が入ると急速に読みにくくなる例

```javascript
<div :class="isError ? 'text-red-500' : 'text-gray-900' + (isActive ? ' font-bold' : '')"></div>

```

```javascript
問題点
  ・条件式がネストする
  ・スペースの付け忘れなどで壊れやすい
  ・クラスの組み合わせが直感的に読めない

```

### 2.2 オブジェクト指定（条件付き ON/OFF）

特定の [[基礎 Part 23 class指定|class]] を状態に応じて付けたり外したりする場合に最適。

```javascript
<div :class="{ 'text-red-500': isError, 'font-bold': isActive }">
  Hello
</div>

```

```javascript
図解: オブジェクト構造
{
  'text-red-500': isError,  → true のときだけ class が付く
  'font-bold': isActive
}

```

複数条件が増えても水平に並べるだけで済むため、大規模でも可読性を保てる。

### 2.3 配列指定（[[基礎 Part 23 class指定|class]] 名そのものを動的に組み立てる）

変数として [[基礎 Part 23 class指定|class]] 名を渡したい場合や、複数クラスをまとめたい場合に使う。

```javascript
<script setup>
import { ref } from 'vue'
const className = ref('bg-blue-500')
</script>

<div :class="[className]">Hello</div>

```

className の値を書き換えるとそのまま [[基礎 Part 23 class指定|class]] が切り替わる。

```javascript
図解: 配列指定のイメージ
[className] → ['bg-blue-500'] → 実際の DOM では class="bg-blue-500"

```

Tailwind の色クラスやサイズクラスを変数化するときに特に便利。

```javascript
<div :class="[`bg-${color}-500`, `text-${size}`]"></div>

```

ただし Tailwind の動的生成は[[コンパイルとビルド|ビルド]]で削除されることがあるため safelist が必要。

```javascript
safelist: [
  'bg-blue-500',
  'bg-red-500',
  'bg-green-500'
]

```

## 3. 条件分岐が複雑化する前に [[基礎 Part 18 computed|computed]] に逃す

template に長いバインディングを書くと読みにくくなるため、[[基礎 Part 18 computed|computed]] にまとめる戦略が重要。

```javascript
<script setup>
import { ref, computed } from 'vue'

const isError = ref(false)
const isActive = ref(true)

const buttonClass = computed(() => [
  'px-4 py-2 rounded transition',
  {
    'bg-red-500 text-white': isError.value,
    'bg-blue-500 text-white': isActive.value && !isError.value,
    'bg-gray-300 text-black': !isActive.value
  }
])
</script>

<template>
  <button :class="buttonClass">Button</button>
</template>

```

## 4. Tailwind と Vue の相性の特徴

Tailwind はクラス名が長く、数が増えやすい。そのため Vue の [[基礎 Part 23 class指定|class]] バインディングが非常に有効。

```plain text
固定クラス → 文字列
状態 ON/OFF → オブジェクト
class 名を動的に → 配列 or computed
複雑になったら → UI コンポーネント化

```

## 4.5 [[基礎 Part 23 class指定|class]] 名を動的にするときの配列と [[基礎 Part 18 computed|computed]] の使い分け

[[基礎 Part 23 class指定|class]] 名そのものを切り替えたい場合、Vue では配列と [[基礎 Part 18 computed|computed]] のどちらでも実現できる。ただし用途によって適切な選択がある。

### 配列を選ぶべきケース

```plain text
・class 名が単純に 1 つだけ変わる
・テンプレート内で完結し、条件が少ない
・即座に見て理解できる程度の複雑さ

```

### 例: 色だけ簡単に切り替える場合

```javascript
<div :class="[`bg-${color}-500`]"></div>

```

color の値が少なく、ロジックも単純な場合に向いている。

### [[基礎 Part 18 computed|computed]] を選ぶべきケース

```plain text
・複数条件が絡む
・class の数が増える
・ロジックをテンプレートから隔離したい
・保守性を重視したい

```

### 例: 状態によって複数 [[基礎 Part 23 class指定|class]] が変わる場合

```javascript
const wrapperClass = computed(() => [
  baseClass,
  {
    'bg-blue-500': isActive.value,
    'bg-red-500': isError.value,
    'opacity-50': isLoading.value
  }
])

```

テンプレートを読みやすくし、ロジックの共通化ができる。

### 判断基準まとめ

```plain text
配列
  ・単純で短命な class の切替
  ・変数をそのまま渡したいだけ
  ・処理が 1〜2 行で収まる

computed
  ・クラスの組み合わせが複雑
  ・条件が 2 つ以上絡む
  ・テンプレートを見やすくしたい
  ・複数のコンポーネントで使い回したい

```

---

## 5. クラス爆発を防ぐための UI [[基礎 Part 29 コンポーネント|コンポーネント]]化

Tailwind を大量に並べると可読性が崩壊する。一定以上複雑になったら[[基礎 Part 29 コンポーネント|コンポーネント]]化するのが正解。

### 煩雑な例

```javascript
<button
  class="px-4 py-2 rounded-md font-bold transition-all shadow-md active:scale-95
         bg-blue-500 text-white hover:bg-blue-600 disabled:opacity-50
         disabled:bg-gray-400 dark:bg-blue-400 dark:text-black sm:px-6"
>
  Save
</button>

```

### [[基礎 Part 29 コンポーネント|コンポーネント]]化

```plain text
<BaseButton variant="primary" :disabled="isLoading">
  Save
</BaseButton>

```

内部で Tailwind の [[基礎 Part 23 class指定|class]] を集約すれば、template を常にシンプルに保てる。

```plain text
図解: コンポーネント化の効果
[ロジック] → BaseButton に集約
[template] → <BaseButton> のみ

```

## 関連
- [[基礎 Part 12 v-bind]]
- [[基礎 Part 32 style scoped]]
- [[基礎 Part 33 グローバルCSS]]
