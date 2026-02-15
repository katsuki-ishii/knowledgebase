---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-27T17:06:00
aliases: [props 命名規則, props命名規則, kebab-case, camelCase, props naming]
---
# Vue [[基礎 Part 38 props 命名規則|props 命名規則]]まとめ

## 1. [[基礎 Part 36 defineProps()|props]]とは

[[基礎 Part 36 defineProps()|props]]は、親[[基礎 Part 30 コンポーネント|コンポーネント]]から子[[基礎 Part 30 コンポーネント|コンポーネント]]へデータを渡すための仕組みである。

子[[基礎 Part 30 コンポーネント|コンポーネント]]は、親から渡された値を読み取り専用のデータとして受け取る。

---

## 2. 命名規則の結論

| 場所 | 命名規則 | 理由 |
| --- | --- | --- |
| 親[[基礎 Part 30 コンポーネント|コンポーネント]]のtemplate（属性） | [[基礎 Part 38 props 命名規則|kebab-case]] | HTML属性は大文字小文字を区別しないため |
| 子[[基礎 Part 30 コンポーネント|コンポーネント]]のscript | [[基礎 Part 38 props 命名規則|camelCase]] | JavaScriptの変数として扱うため |
| 子[[基礎 Part 30 コンポーネント|コンポーネント]]のtemplate内の式 | [[基礎 Part 38 props 命名規則|camelCase]] | 中身はJavaScript式のため |

---

## 3. 図解：[[基礎 Part 36 defineProps()|props]]が渡る流れ

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

## 4. 親[[基礎 Part 30 コンポーネント|コンポーネント]]側の具体例

### 親のtemplate

```plain text
<UserCard
  :user-name="user.name"
  :is-admin="user.isAdmin"
/>

```

ここではHTML属性として書くため、[[基礎 Part 38 props 命名規則|kebab-case]]を使う。

---

## 5. 子[[基礎 Part 30 コンポーネント|コンポーネント]]側の具体例

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

子[[基礎 Part 30 コンポーネント|コンポーネント]]では、[[基礎 Part 36 defineProps()|props]]はJavaScriptの変数として扱われるため、[[基礎 Part 38 props 命名規則|camelCase]]で参照する。

---

## 6. なぜ子templateでも[[基礎 Part 38 props 命名規則|camelCase]]なのか

子[[基礎 Part 30 コンポーネント|コンポーネント]]のtemplateはHTMLに見えるが、以下はすべてJavaScript式である。

- {{ }} の中
- [[基礎 Part 25 v-if|v-if]]、[[基礎 Part 27 v-for 配列|v-for]]、[[基礎 Part 12 v-bind|v-bind]] の値

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

そのため、次のように書くと意図した[[基礎 Part 36 defineProps()|props]]名が失われる。

```html
<UserCard userName="tanaka" />

```

ブラウザ内部では `username` として扱われてしまう。

---

## 8. [[基礎 Part 38 props 命名規則|kebab-case]]を使う理由まとめ

- HTML仕様に安全
- すべて小文字で一貫性がある
- Vueが確実に[[基礎 Part 38 props 命名規則|camelCase]]へ変換できる

---

## 9. 覚え方（実務向け）

- ハイフンが出てきたらHTMLの世界
- 変数として触るなら[[基礎 Part 38 props 命名規則|camelCase]]
- [[基礎 Part 36 defineProps()|props]]は属性として渡し、変数として使う

この整理を意識すると、Vueの[[基礎 Part 36 defineProps()|props]]で迷わなくなる。
