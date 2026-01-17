---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-07T18:38:00
aliases: [v-show, vshow, v show]
---
# [[基礎 Part 25 v-show|v-show]]まとめ

Vue.jsにおける[[基礎 Part 25 v-show|v-show]]の要点を整理する。

---

## 1. [[基礎 Part 25 v-show|v-show]]の基本

[[基礎 Part 25 v-show|v-show]]は要素の表示・非表示を[[ULCOD CSS|CSS]]の`display`プロパティで切り替える仕組み。

- 表示: `display: block`（通常表示）
- 非表示: `display: none`

DOMそのものは消えずに保持される。

### 例

```html
<p v-show="isVisible">表示／非表示が切り替わる</p>

```

---

## 2. [[基礎 Part 24 v-if|v-if]]との違い

### 図解

```plain text
v-if
┌──────────────┐
│ 条件がtrueのときだけDOMを生成・表示 │
└──────────────┘

v-show
┌──────────────┐
│ DOMは常に存在し displayのみ変更      │
└──────────────┘

```

| 特徴 | [[基礎 Part 24 v-if|v-if]] | [[基礎 Part 25 v-show|v-show]] |
| --- | --- | --- |
| DOMの扱い | 生成・削除 | 常に保持 |
| 切り替え速度 | 遅い（DOM操作） | 速い（[[ULCOD CSS|CSS]]切替） |
| 初期コスト | 条件がtrueの時だけ生成 | 常に生成 |
| 向く場面 | めったに表示しない要素 | 頻繁にON/OFFする要素 |

---

## 3. [[基礎 Part 25 v-show|v-show]]でつまずきやすいポイント

### (1) templateタグに使えない

[[基礎 Part 25 v-show|v-show]]はDOM要素専用。templateは実DOMにならないため無効。

### 例（誤り）

```html
<template v-show="isOpen">
  <p>A</p>
  <p>B</p>
</template>

```

### 正しい例

```html
<div v-show="isOpen">
  <p>A</p>
  <p>B</p>
</div>

```

### 図解

```plain text
<template>  → DOMに出ない → displayを付けられない
<div>       → DOMに出る   → displayを切り替え可能

```

---

### (2) DOMが消えず状態が保持される

非表示でもDOMが残るため、内部状態が維持される。

### 例

```html
<input v-show="show" v-model="text" />

```

showがfalse → trueに戻しても入力内容は残る。

---

### (3) 初期描画は必ず行われる

非表示状態でもDOMを最初に作成する。

重い[[基礎 Part 29 コンポーネント|コンポーネント]]に使うと非効率になる。

---

### (4) アニメーションが意図通り動かないことがある

[[基礎 Part 25 v-show|v-show]]はdisplay切替のため、transitionと相性が良くない場合がある。

opacityやheightを使った[[ULCOD CSS|CSS]]工夫が必要になることがある。

---

## 4. 使いどころの基準

```plain text
頻繁にON/OFF → v-show
まれに表示 → v-if

```

---

## 5. まとめ図

```plain text
           DOM生成コスト   切替速度   状態保持
v-if       高              低          ×（破棄）
v-show     低              高          ○（保持）

```

---

必要なら具体的な[[基礎 Part 29 コンポーネント|コンポーネント]]例や、[[基礎 Part 25 v-show|v-show]]と[[基礎 Part 24 v-if|v-if]]の選択基準の詳細も追記できる。

## 関連
- [[基礎 Part 24 v-if]]
- [[基礎 Part 26 v-for 配列]]
