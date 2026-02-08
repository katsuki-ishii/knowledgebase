---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-27T14:21:00
aliases: [defineProps, defineProps(), props, Props]
---
# Vue 3 [[基礎 Part 36 defineProps()|props]] と [[基礎 Part 36 defineProps()|defineProps]] 基礎まとめ

## 1. [[基礎 Part 36 defineProps()|props]]とは何か

[[基礎 Part 36 defineProps()|props]] は **properties（プロパティ）** の略で、

**親[[基礎 Part 30 コンポーネント|コンポーネント]]から子[[基礎 Part 30 コンポーネント|コンポーネント]]へ渡すためのデータ** を指す。

Vueでは以下の考え方を取る。

- [[基礎 Part 30 コンポーネント|コンポーネント]]は部品
- 親がデータを管理する
- 子は受け取って表示・利用する

[[基礎 Part 36 defineProps()|props]] は **子[[基礎 Part 30 コンポーネント|コンポーネント]]から見ると読み取り専用** である。

---

## 2. データの流れ（重要）

Vueの基本ルールは「一方向データフロー」。

```plain text
親コンポーネント（データを持つ）
        ↓ props
子コンポーネント（表示・利用）

```

子が [[基礎 Part 36 defineProps()|props]] を変更すると、

「どこでデータが変わったのか分からなくなる」ため、Vueはこれを禁止している。

---

## 3. 親から子へデータを渡す具体例

### 親[[基礎 Part 30 コンポーネント|コンポーネント]]（Parent.vue）

```plain text
<script setup>
const message = 'こんにちは'
</script>

<template>
  <ChildComponent :text="message" />
</template>

```

ここで起きていること：

- `message` は親が持つデータ
- `text` という名前で子に渡している
- `:` は JavaScript の値として渡すための Vue 特有の記法

---

## 4. 子[[基礎 Part 30 コンポーネント|コンポーネント]]での受け取り（[[基礎 Part 36 defineProps()|defineProps]]）

### 子[[基礎 Part 30 コンポーネント|コンポーネント]]（ChildComponent.vue）

```plain text
<script setup>
const props = defineProps({
  text: String
})
</script>

<template>
  <p>{{ props.text }}</p>
</template>

```

### [[基礎 Part 36 defineProps()|defineProps]] の役割

- この[[基礎 Part 30 コンポーネント|コンポーネント]]が「何を受け取るか」を[[基礎 Part 8 宣言・代入・再代入|宣言]]する
- [[基礎 Part 36 defineProps()|props]] の型を定義し、バグを防ぐ

---

## 5. [[基礎 Part 36 defineProps()|defineProps]] はなぜ [[基礎 Part 11 ESM　Import＆Export|import]] が不要なのか

`defineProps` は通常の JavaScript 関数ではない。

- Vue 3 + `<script setup>` 専用
- [[コンパイルとビルド|コンパイル]]時に Vue が処理する **[[基礎 Part 35 コンパイル時マクロ|マクロ]]**

そのため以下のように [[基礎 Part 11 ESM　Import＆Export|import]] は不要。

```javascript
const props = defineProps({
  text: String
})

```

これは JavaScript の文法ではなく、**Vue フレームワーク特有の仕組み**。

---

## 6. [[基礎 Part 36 defineProps()|props]] は変更してはいけない

### 間違った例

```javascript
props.text = '変更'

```

理由：

- [[基礎 Part 36 defineProps()|props]] の実体は親のデータ
- 子が勝手に変更すると状態が破綻する

### 正しい考え方

- 親がデータを変更する
- 子は表示や通知だけを担当する

---

## 7. 図で理解する [[基礎 Part 36 defineProps()|props]]

### 親 → 子 の関係

```plain text
[ Parent.vue ]
  message = 'こんにちは'
        |
        | props: text
        v
[ Child.vue ]
  props.text を表示

```

### HTML属性に近いイメージ

```plain text
<MyButton label="保存" disabled />

```

- label
- disabled

これらが [[基礎 Part 36 defineProps()|props]]

---

## 8. 実務でよくある [[基礎 Part 36 defineProps()|props]] の例

### ユーザー情報カード

```plain text
<UserCard :name="user.name" :age="user.age" />

```

### 共通ボタン

```plain text
<BaseButton :label="'送信'" :disabled="isLoading" />

```

### API結果の表示

```plain text
<ArticleItem :article="article" />

```

---

## 9. 重要ポイントまとめ

- [[基礎 Part 36 defineProps()|props]] は properties の略
- 親から子へ渡すデータ
- 子から見ると読み取り専用
- [[基礎 Part 36 defineProps()|defineProps]] は [[基礎 Part 11 ESM　Import＆Export|import]] 不要な Vue [[基礎 Part 35 コンパイル時マクロ|マクロ]]
- データの流れは常に「親 → 子」

この理解が Vue [[基礎 Part 30 コンポーネント|コンポーネント]]設計の土台になる。
