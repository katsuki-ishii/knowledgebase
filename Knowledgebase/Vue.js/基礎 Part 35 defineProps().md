---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-27T14:21:00
---
# Vue 3 props と defineProps 基礎まとめ

## 1. propsとは何か

props は **properties（プロパティ）** の略で、

**親コンポーネントから子コンポーネントへ渡すためのデータ** を指す。

Vueでは以下の考え方を取る。

- コンポーネントは部品
- 親がデータを管理する
- 子は受け取って表示・利用する

props は **子コンポーネントから見ると読み取り専用** である。

---

## 2. データの流れ（重要）

Vueの基本ルールは「一方向データフロー」。

```plain text
親コンポーネント（データを持つ）
        ↓ props
子コンポーネント（表示・利用）

```

子が props を変更すると、

「どこでデータが変わったのか分からなくなる」ため、Vueはこれを禁止している。

---

## 3. 親から子へデータを渡す具体例

### 親コンポーネント（Parent.vue）

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

## 4. 子コンポーネントでの受け取り（defineProps）

### 子コンポーネント（ChildComponent.vue）

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

### defineProps の役割

- このコンポーネントが「何を受け取るか」を宣言する
- props の型を定義し、バグを防ぐ

---

## 5. defineProps はなぜ import が不要なのか

`defineProps` は通常の JavaScript 関数ではない。

- Vue 3 + `<script setup>` 専用
- コンパイル時に Vue が処理する **マクロ**

そのため以下のように import は不要。

```javascript
const props = defineProps({
  text: String
})

```

これは JavaScript の文法ではなく、**Vue フレームワーク特有の仕組み**。

---

## 6. props は変更してはいけない

### 間違った例

```javascript
props.text = '変更'

```

理由：

- props の実体は親のデータ
- 子が勝手に変更すると状態が破綻する

### 正しい考え方

- 親がデータを変更する
- 子は表示や通知だけを担当する

---

## 7. 図で理解する props

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

これらが props

---

## 8. 実務でよくある props の例

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

- props は properties の略
- 親から子へ渡すデータ
- 子から見ると読み取り専用
- defineProps は import 不要な Vue マクロ
- データの流れは常に「親 → 子」

この理解が Vue コンポーネント設計の土台になる。

## 関連
- [[基礎 Part 36 props バリデーション]]
- [[基礎 Part 37 props 命名規則]]
- [[基礎 Part 29 コンポーネント]]
