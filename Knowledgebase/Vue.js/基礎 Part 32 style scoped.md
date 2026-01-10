---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-22T09:20:00
---
# Vue の scoped CSS と DOM・継承の整理

## 0. scoped がない世界と、なぜ scoped が生まれたか

### scoped がない場合の CSS

Vue に限らず、CSS は本来 **すべてグローバル** に効く言語。

```css
.button {
  color: red;
}

```

この CSS は「このファイル内」ではなく、**ページ内のすべての **`**.button**` に適用される。

### コンポーネント分割が進むと起きる問題

```plain text
<!-- A.vue -->
<style>
.button {
  color: red;
}
</style>

```

```plain text
<!-- B.vue -->
<style>
.button {
  color: blue;
}
</style>

```

結果：

- 読み込み順によって
- どちらか一方が全体に勝つ

```plain text
最後に読み込まれた CSS が勝つ

```

### 何が問題か

- コンポーネント単位で見た目を完結できない
- クラス名の衝突が頻発する
- 修正が別コンポーネントに波及する

これは **大規模開発では致命的**。

---

### scoped が生まれた理由

目的は一つだけ。

> 「このコンポーネント用に書いた CSS を、他に漏らさない」

- CSS を JavaScript のスコープに近づけたい
- ただし CSS の仕様自体は変えられない

そこで Vue が選んだ解決策が **属性セレクタによる限定**。

---

## 1. 前提：Vue コンポーネントと DOM

Vue のコンポーネントは設計上は分かれているが、**最終的にブラウザ上では 1 本の DOM ツリー**になる。

```plain text
<Parent>
  <Child />
</Parent>

```

実際の DOM 構造（概念図）：

```plain text
Parent の DOM
└─ Child の DOM

```

Vue のコンポーネント境界は **DOM の境界ではない**。

---

## 2. scoped がやっていること

```plain text
<style scoped>
.parent {
  color: red;
}
</style>

```

Vue は scoped がある場合、**HTML と CSS をビルド時に書き換える**。

### DOM 側

```html
<div class="parent" data-v-parent>

```

### CSS 側

```css
.parent[data-v-parent] {
  color: red;
}

```

### 重要な事実

- `data-v-parent` は **Vue が自動で付ける内部識別子**
- 開発者が書いていない
- scoped ごとに一意

---

## 3. data-v-xxx が付く範囲

### 正しい範囲

> そのコンポーネント自身が生成する DOM 要素すべて

### 具体例

```plain text
<!-- Parent.vue -->
<template>
  <div class="parent">
    <Child />
  </div>
</template>

```

```plain text
<!-- Child.vue -->
<template>
  <div class="child">
    <p>text</p>
  </div>
</template>

```

### 実際の DOM（概念）

```html
<div class="parent" data-v-parent>
  <div class="child" data-v-child>
    <p data-v-child>text</p>
  </div>
</div>

```

### ポイント

- 親の `data-v-parent` は **子には付かない**
- 子には必ず **子専用の **`**data-v-child**` が付く
- 子が div 1 個でも同じ

---

## 4. 「1要素なら data が継承される」という誤解

これは **scoped の話ではない**。

### 正体

Vue 3 の **属性フォールスルー（fallthrough attributes）**。

---

## 5. 属性フォールスルーとは

親コンポーネントで渡した属性のうち、

**props として受け取られていないもの**を、

子のルート要素が 1 つのときに自動で付与する仕組み。

### 例

```plain text
<!-- Parent.vue -->
<Child class="box" data-test="x" />

```

```plain text
<!-- Child.vue -->
<template>
  <div>hello</div>
</template>

```

### 結果の DOM

```html
<div class="box" data-test="x" data-v-child>hello</div>

```

### フォールスルーされるもの

- class
- id
- data-xxx
- aria-xxx

### フォールスルーされないもの

- data-v-parent（scoped 用識別子）
- Vue 内部管理用属性

---

## 6. なぜ「継承されたように見える」のか

原因は **CSS の継承ルール**。

### 継承される CSS プロパティの例

- color
- font-family
- font-size
- line-height

### 継承されない CSS プロパティの例

- margin
- padding
- border
- background

---

## 7. scoped と CSS 継承の関係

```css
.parent[data-v-parent] {
  color: red;
}

```

- CSS が当たるのは **親要素だけ**
- `color` は継承される
- その結果、**子の文字が赤く見える**

これは scoped の仕様ではなく、**CSS 自体の仕様**。

---

## 8. 図解まとめ

```plain text
DOM ツリー

parent (data-v-parent)
└─ child (data-v-child)
   └─ p (data-v-child)

CSS 適用

.parent[data-v-parent] { color: red }
        │
        └─ color が子に継承される

```

---

## 9. 重要な結論

- scoped は「CSS セレクタの衝突」を防ぐ仕組み
- コンポーネント境界で CSS の適用先を限定する
- data-v-xxx は **絶対に他コンポーネントへ継承されない**
- 子に影響して見えるのは CSS の継承
- 「1 要素なら data が付く」は scoped ではなく属性フォールスルー

---

## 10. 実務での安全な理解

- 親はトーン（色・フォント）を決める
- 子は構造（余白・枠・サイズ）を決める
- scoped を JS のスコープと同一視しない
- DOM と CSS のルールを基準に考える

## 関連
- [[基礎 Part 33 グローバルCSS]]
- [[基礎 Part 23 class指定]]
