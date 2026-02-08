---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-21T23:30:00
aliases: [属性フォークスルー, 属性フォールスルー, $attrs, attribute fallthrough, attrs]
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

属性（[[基礎 Part 24 class指定|class]] / [[HTMLのclass, id, style|id]] など）は、生成されたDOM要素に付与される。

---

## 2. ルート要素とは

### 定義

[[基礎 Part 30 コンポーネント|コンポーネント]]が最終的に返すDOM構造の入口となる要素。

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

## 4. 属性の自動付与（[[基礎 Part 32 属性フォークスルー|Attribute Fallthrough]]）

### 名前

**[[基礎 Part 32 属性フォークスルー|Attribute Fallthrough]]（[[基礎 Part 32 属性フォークスルー|属性フォールスルー]]）**

Vueでは、親[[基礎 Part 30 コンポーネント|コンポーネント]]で指定した属性が、

子[[基礎 Part 30 コンポーネント|コンポーネント]]のルート要素に自動的に渡される挙動をこう呼ぶ。

意味：

- 親で指定した属性が
- [[基礎 Part 36 defineProps()|props]]として受け取られなかった場合に
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

- [[基礎 Part 24 class指定|class]] はどこにも付かない
- Vueは自動で判断しない

理由：

- どの要素に付けるべきか決められないため

---

## 5. [[基礎 Part 32 属性フォークスルー|$attrs]] とは何か

### 定義

親から渡された属性のうち、[[基礎 Part 36 defineProps()|props]]として定義されなかったものの集合。

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

## 6. [[基礎 Part 12 v-bind|v-bind]]="[[基礎 Part 32 属性フォークスルー|$attrs]]" の意味

```plain text
<p v-bind="$attrs">A</p>

```

意味：

- [[基礎 Part 32 属性フォークスルー|$attrs]] の中身をすべてこの要素の属性として付与する

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

## 8. [[基礎 Part 32 属性フォークスルー|$attrs]] を複数の要素に使った場合

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

- [[HTMLのclass, id, style|id]] が重複する
- テストやアクセシビリティが壊れる

---

## 9. 実務での原則

- [[基礎 Part 32 属性フォークスルー|$attrs]] は原則1箇所だけに使う
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

- [[基礎 Part 30 コンポーネント|コンポーネント]]は「箱」か？
    - Yes → ルート1つ、属性を集約
    - No → Fragmentを使う
- 親から [[基礎 Part 24 class指定|class]] / [[HTMLのclass, id, style|id]] を付けたいか？
    - Yes → [[基礎 Part 32 属性フォークスルー|$attrs]] を明示的に配置

---

## 11. 全体像（要点）

- [[基礎 Part 30 コンポーネント|コンポーネント]]のタグ自体はDOMに出ない
- 属性は生成されたDOMに付く
- ルート1つなら自動
- Fragmentでは自動付与されない
- [[基礎 Part 32 属性フォークスルー|$attrs]] は属性の転送箱
- 責任を持って付け先を決める
