---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-21T23:30:00
---
# Vueにおける属性とルート要素の学習まとめ

## 1. 前提：Vueコンポーネントとは何か

**VueコンポーネントはHTMLタグではない。**

DOMを生成するための設計図（関数）であり、`<ChildComponent />` というタグ自体は最終的なDOMには存在しない。

```plain text
親テンプレート
<ChildComponent class="red" />

↓ Vueが展開

<div class="red">...</div>

```

属性（class / id など）は、生成されたDOM要素に付与される。

---

## 2. ルート要素とは

### 定義

コンポーネントが最終的に返すDOM構造の入口となる要素。

### ルート要素が1つの場合

```plain text
<template>
  <div>内容</div>
</template>

```

- 親から渡された属性は自動でこの要素に付く

```plain text
<ChildComponent class="box" />

↓

<div class="box">内容</div>

```

---

## 3. Vue 3 の変更点：ルート要素が複数書ける

### Fragment（フラグメント）

```plain text
<template>
  <p>A</p>
  <p>B</p>
</template>

```

- 余計なラッパーDOMを作らない
- DOM上には p 要素が2つ並ぶ

```plain text
<p>A</p>
<p>B</p>

```

---

## 4. 属性の自動付与（Attribute Fallthrough）

### 名前

**Attribute Fallthrough（属性フォールスルー）**

Vueでは、親コンポーネントで指定した属性が、

子コンポーネントのルート要素に自動的に渡される挙動をこう呼ぶ。

意味：

- 親で指定した属性が
- propsとして受け取られなかった場合に
- 子のルート要素へそのまま「落ちてくる」

---

## 5. ルート要素が複数ある場合の属性の挙動

```plain text
<ChildComponent class="red" />

```

```plain text
<template>
  <p>A</p>
  <p>B</p>
</template>

```

結果：

- class はどこにも付かない
- Vueは自動で判断しない

理由：

- どの要素に付けるべきか決められないため

---

## 5. $attrs とは何か

### 定義

親から渡された属性のうち、propsとして定義されなかったものの集合。

```plain text
<ChildComponent class="red" id="main" />

```

```plain text
<script setup>
defineProps({ title: String })
</script>

```

この場合：

```plain text
$attrs = {
  class: "red",
  id: "main"
}

```

---

## 6. v-bind="$attrs" の意味

```plain text
<p v-bind="$attrs">A</p>

```

意味：

- $attrs の中身をすべてこの要素の属性として付与する

結果：

```plain text
<p class="red" id="main">A</p>

```

---

## 7. 図解：属性の流れ

```plain text
親
<ChildComponent class="red" id="x" />
        |
        v
$attrs = { class: "red", id: "x" }
        |
        v
<p v-bind="$attrs">

```

---

## 8. $attrs を複数の要素に使った場合

```plain text
<template>
  <div v-bind="$attrs">A</div>
  <div v-bind="$attrs">B</div>
</template>

```

生成されるDOM：

```html
<div class="red" id="x">A</div>
<div class="red" id="x">B</div>

```

問題点：

- id が重複する
- テストやアクセシビリティが壊れる

---

## 9. 実務での原則

- $attrs は原則1箇所だけに使う
- 複数に付けたくなったら分解する

### 分解例

```plain text
<script setup>
const attrs = useAttrs()
</script>

<template>
  <div :class="attrs.class">A</div>
  <div :class="attrs.class">B</div>
</template>

```

---

## 10. 判断基準まとめ

- コンポーネントは「箱」か？
    - Yes → ルート1つ、属性を集約
    - No → Fragmentを使う
- 親から class / id を付けたいか？
    - Yes → $attrs を明示的に配置

---

## 11. 全体像（要点）

- コンポーネントのタグ自体はDOMに出ない
- 属性は生成されたDOMに付く
- ルート1つなら自動
- Fragmentでは自動付与されない
- $attrs は属性の転送箱
- 責任を持って付け先を決める

## 関連
- [[基礎 Part 29 コンポーネント]]
- [[基礎 Part 35 defineProps()]]
