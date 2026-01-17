---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-22T09:20:00
aliases: [style scoped, scoped, scoped CSS, style scoped CSS]
---
# Vue の [[基礎 Part 32 style scoped|scoped CSS]] と DOM・継承の整理

## 0. [[基礎 Part 32 style scoped|scoped]] がない世界と、なぜ [[基礎 Part 32 style scoped|scoped]] が生まれたか

### [[基礎 Part 32 style scoped|scoped]] がない場合の [[ULCOD CSS|CSS]]

Vue に限らず、[[ULCOD CSS|CSS]] は本来 **すべて[[基礎 Part 33 グローバルCSS|グローバル]]** に効く言語。

```css
.button {
  color: red;
}

```

この [[ULCOD CSS|CSS]] は「このファイル内」ではなく、**ページ内のすべての **`**.button**` に適用される。

### [[基礎 Part 29 コンポーネント|コンポーネント]][[基礎 Part 13 分割代入|分割]]が進むと起きる問題

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

- [[基礎 Part 29 コンポーネント|コンポーネント]]単位で見た目を完結できない
- クラス名の衝突が頻発する
- 修正が別[[基礎 Part 29 コンポーネント|コンポーネント]]に波及する

これは **大規模開発では致命的**。

---

### [[基礎 Part 32 style scoped|scoped]] が生まれた理由

目的は一つだけ。

> 「この[[基礎 Part 29 コンポーネント|コンポーネント]]用に書いた [[ULCOD CSS|CSS]] を、他に漏らさない」

- [[ULCOD CSS|CSS]] を JavaScript のスコープに近づけたい
- ただし [[ULCOD CSS|CSS]] の仕様自体は変えられない

そこで Vue が選んだ解決策が **属性セレクタによる限定**。

---

## 1. 前提：Vue [[基礎 Part 29 コンポーネント|コンポーネント]]と DOM

Vue の[[基礎 Part 29 コンポーネント|コンポーネント]]は設計上は分かれているが、**最終的にブラウザ上では 1 本の DOM ツリー**になる。

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

Vue の[[基礎 Part 29 コンポーネント|コンポーネント]]境界は **DOM の境界ではない**。

---

## 2. [[基礎 Part 32 style scoped|scoped]] がやっていること

```plain text
<style scoped>
.parent {
  color: red;
}
</style>

```

Vue は [[基礎 Part 32 style scoped|scoped]] がある場合、**HTML と [[ULCOD CSS|CSS]] を[[コンパイルとビルド|ビルド]]時に書き換える**。

### DOM 側

```html
<div class="parent" data-v-parent>

```

### [[ULCOD CSS|CSS]] 側

```css
.parent[data-v-parent] {
  color: red;
}

```

### 重要な事実

- `data-v-parent` は **Vue が自動で付ける内部識別子**
- 開発者が書いていない
- [[基礎 Part 32 style scoped|scoped]] ごとに一意

---

## 3. data-v-xxx が付く範囲

### 正しい範囲

> その[[基礎 Part 29 コンポーネント|コンポーネント]]自身が生成する DOM 要素すべて

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

これは **[[基礎 Part 32 style scoped|scoped]] の話ではない**。

### 正体

Vue 3 の **[[基礎 Part 31  属性フォークスルー|属性フォールスルー]]（fallthrough attributes）**。

---

## 5. [[基礎 Part 31  属性フォークスルー|属性フォールスルー]]とは

親[[基礎 Part 29 コンポーネント|コンポーネント]]で渡した属性のうち、

**[[基礎 Part 35 defineProps()|props]] として受け取られていないもの**を、

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

- [[基礎 Part 23 class指定|class]]
- [[HTMLのclass, id, style|id]]
- data-xxx
- aria-xxx

### フォールスルーされないもの

- data-v-parent（[[基礎 Part 32 style scoped|scoped]] 用識別子）
- Vue 内部管理用属性

---

## 6. なぜ「継承されたように見える」のか

原因は **[[ULCOD CSS|CSS]] の継承ルール**。

### 継承される [[ULCOD CSS|CSS]] プロパティの例

- color
- font-family
- font-size
- line-height

### 継承されない [[ULCOD CSS|CSS]] プロパティの例

- margin
- padding
- border
- background

---

## 7. [[基礎 Part 32 style scoped|scoped]] と [[ULCOD CSS|CSS]] 継承の関係

```css
.parent[data-v-parent] {
  color: red;
}

```

- [[ULCOD CSS|CSS]] が当たるのは **親要素だけ**
- `color` は継承される
- その結果、**子の文字が赤く見える**

これは [[基礎 Part 32 style scoped|scoped]] の仕様ではなく、**[[ULCOD CSS|CSS]] 自体の仕様**。

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

- [[基礎 Part 32 style scoped|scoped]] は「[[ULCOD CSS|CSS]] セレクタの衝突」を防ぐ仕組み
- [[基礎 Part 29 コンポーネント|コンポーネント]]境界で [[ULCOD CSS|CSS]] の適用先を限定する
- data-v-xxx は **絶対に他[[基礎 Part 29 コンポーネント|コンポーネント]]へ継承されない**
- 子に影響して見えるのは [[ULCOD CSS|CSS]] の継承
- 「1 要素なら data が付く」は [[基礎 Part 32 style scoped|scoped]] ではなく[[基礎 Part 31  属性フォークスルー|属性フォールスルー]]

---

## 10. 実務での安全な理解

- 親はトーン（色・フォント）を決める
- 子は構造（余白・枠・サイズ）を決める
- [[基礎 Part 32 style scoped|scoped]] を JS のスコープと同一視しない
- DOM と [[ULCOD CSS|CSS]] のルールを基準に考える

## 関連
- [[基礎 Part 33 グローバルCSS]]
- [[基礎 Part 23 class指定]]
