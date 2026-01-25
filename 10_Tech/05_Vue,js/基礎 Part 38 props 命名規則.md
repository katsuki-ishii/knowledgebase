---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-27T17:06:00
aliases: [props 命名規則, props命名規則, kebab-case, camelCase, props naming]
---
# Vue [[基礎 props 命名規則|props 命名規則 Part 37]]まとめ

## 1. [[基礎 defineProps()|props Part 35]]とは

[[基礎 defineProps()|props Part 35]]は、親[[基礎 コンポーネント|コンポーネント Part 29]]から子[[基礎 コンポーネント|コンポーネント Part 29]]へデータを渡すための仕組みである。

子[[基礎 コンポーネント|コンポーネント Part 29]]は、親から渡された値を読み取り専用のデータとして受け取る。

---

## 2. 命名規則の結論

| 場所 | 命名規則 | 理由 |
| --- | --- | --- |
| 親[[基礎 コンポーネント|コンポーネント Part 29]]のtemplate（属性） | [[基礎 props 命名規則|kebab-case Part 37]] | HTML属性は大文字小文字を区別しないため |
| 子[[基礎 コンポーネント|コンポーネント Part 29]]のscript | [[基礎 props 命名規則|camelCase Part 37]] | JavaScriptの変数として扱うため |
| 子[[基礎 コンポーネント|コンポーネント Part 29]]のtemplate内の式 | [[基礎 props 命名規則|camelCase Part 37]] | 中身はJavaScript式のため |

---

## 3. 図解：[[基礎 defineProps()|props Part 35]]が渡る流れ

```plain text
親 template（HTML）
  <UserCard user-name="tanaka" />
          │
          │ Vueが自動変換
          ▼
子 script（JavaScript）
  props.userName
          │
          ▼
子 template（JavaScript式）
  {{ userName }}

```

---

## 4. 親[[基礎 コンポーネント|コンポーネント Part 29]]側の具体例

### 親のtemplate

```plain text
<UserCard
  :user-name="user.name"
  :is-admin="user.isAdmin"
/>

```

ここではHTML属性として書くため、[[基礎 props 命名規則|kebab-case Part 37]]を使う。

---

## 5. 子[[基礎 コンポーネント|コンポーネント Part 29]]側の具体例

### script（Composition API）

```typescript
const props = defineProps<{
  userName: string
  isAdmin: boolean
}>()

```

### template

```plain text
<template>
  <p>{{ userName }}</p>
  <p v-if="isAdmin">管理者</p>
</template>

```

子[[基礎 コンポーネント|コンポーネント Part 29]]では、[[基礎 defineProps()|props Part 35]]はJavaScriptの変数として扱われるため、[[基礎 props 命名規則|camelCase Part 37]]で参照する。

---

## 6. なぜ子templateでも[[基礎 props 命名規則|camelCase Part 37]]なのか

子[[基礎 コンポーネント|コンポーネント Part 29]]のtemplateはHTMLに見えるが、以下はすべてJavaScript式である。

- {{ }} の中
- [[基礎 v-if|v-if Part 24]]、[[基礎 v-for 配列|v-for Part 26]]、[[基礎 v-bind|v-bind Part 12]] の値

そのため、変数名のルールはJavaScriptに従う。

```plain text
{{ user-name }}  // 誤り（引き算として解釈される）
{{ userName }}   // 正しい

```

---

## 7. HTML属性がcase-insensitiveであることの影響

HTMLでは次はすべて同じ意味になる。

```html
<input TYPE="text" VALUE="a">
<input type="text" value="a">

```

そのため、次のように書くと意図した[[基礎 defineProps()|props Part 35]]名が失われる。

```html
<UserCard userName="tanaka" />

```

ブラウザ内部では `username` として扱われてしまう。

---

## 8. [[基礎 props 命名規則|kebab-case Part 37]]を使う理由まとめ

- HTML仕様に安全
- すべて小文字で一貫性がある
- Vueが確実に[[基礎 props 命名規則|camelCase Part 37]]へ変換できる

---

## 9. 覚え方（実務向け）

- ハイフンが出てきたらHTMLの世界
- 変数として触るなら[[基礎 props 命名規則|camelCase Part 37]]
- [[基礎 defineProps()|props Part 35]]は属性として渡し、変数として使う

この整理を意識すると、Vueの[[基礎 defineProps()|props Part 35]]で迷わなくなる。

## 関連
- [[基礎 Part 39 defineProps() Part 35]]
- [[基礎 props バリデーション]]
