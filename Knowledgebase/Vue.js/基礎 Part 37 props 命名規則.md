---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-27T17:06:00
---
# Vue props 命名規則まとめ

## 1. propsとは

propsは、親コンポーネントから子コンポーネントへデータを渡すための仕組みである。

子コンポーネントは、親から渡された値を読み取り専用のデータとして受け取る。

---

## 2. 命名規則の結論

| 場所 | 命名規則 | 理由 |
| --- | --- | --- |
| 親コンポーネントのtemplate（属性） | kebab-case | HTML属性は大文字小文字を区別しないため |
| 子コンポーネントのscript | camelCase | JavaScriptの変数として扱うため |
| 子コンポーネントのtemplate内の式 | camelCase | 中身はJavaScript式のため |

---

## 3. 図解：propsが渡る流れ

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

## 4. 親コンポーネント側の具体例

### 親のtemplate

```plain text
<UserCard
  :user-name="user.name"
  :is-admin="user.isAdmin"
/>

```

ここではHTML属性として書くため、kebab-caseを使う。

---

## 5. 子コンポーネント側の具体例

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

子コンポーネントでは、propsはJavaScriptの変数として扱われるため、camelCaseで参照する。

---

## 6. なぜ子templateでもcamelCaseなのか

子コンポーネントのtemplateはHTMLに見えるが、以下はすべてJavaScript式である。

- {{ }} の中
- v-if、v-for、v-bind の値

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

そのため、次のように書くと意図したprops名が失われる。

```html
<UserCard userName="tanaka" />

```

ブラウザ内部では `username` として扱われてしまう。

---

## 8. kebab-caseを使う理由まとめ

- HTML仕様に安全
- すべて小文字で一貫性がある
- Vueが確実にcamelCaseへ変換できる

---

## 9. 覚え方（実務向け）

- ハイフンが出てきたらHTMLの世界
- 変数として触るならcamelCase
- propsは属性として渡し、変数として使う

この整理を意識すると、Vueのpropsで迷わなくなる。

## 関連
- [[基礎 Part 35 defineProps()]]
- [[基礎 Part 36 props バリデーション]]
