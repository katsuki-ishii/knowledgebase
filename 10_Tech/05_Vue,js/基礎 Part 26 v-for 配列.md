---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-07T18:59:00
aliases: [v-for 配列, v-for, vfor, v for, key]
---
# [[基礎 Part 26 v-for 配列|v-for]] と [[基礎 Part 26 v-for 配列|key]] の基礎

---

## 1. [[基礎 Part 26 v-for 配列|v-for]] の基本

[[基礎 Part 26 v-for 配列|v-for]] は配列やオブジェクトを繰り返し描画するための構文である。

```plain text
<li v-for="item in items">{{ item }}</li>

```

### 配列例

```javascript
const items = ["apple", "banana", "orange"]

```

描画結果:

- apple
- banana
- orange

---

## 2. [[基礎 Part 2 index.htmlとmain.js|index]] の取得

```plain text
<li v-for="(item, index) in items">{{ index }}: {{ item }}</li>

```

[[基礎 Part 2 index.htmlとmain.js|index]] は 0 始まりの番号。

---

## 3. オブジェクトを回す

```plain text
<div v-for="(value, key) in user">{{ key }}: {{ value }}</div>

```

例:

```javascript
const user = { name: "Katsuki", age: 25 }

```

描画結果:

name: Katsuki

age: 25

---

## 4. [[基礎 Part 26 v-for 配列|key]] が必要な理由

Vue は描画済み DOM を再利用して高速化する。その際、要素の識別に [[基礎 Part 26 v-for 配列|key]] を使う。[[基礎 Part 26 v-for 配列|key]] がないと Vue がどの DOM が何に対応するか判断できず、意図しない再利用が発生する。

### 図解

初回描画:

```plain text
items = [1, 2, 3]

DOM1: <div>1</div>
DOM2: <div>2</div>
DOM3: <div>3</div>

```

並び替え:

```plain text
items = [3, 1, 2]

```

[[基礎 Part 26 v-for 配列|key]] がない場合の Vue の推測動作:

```plain text
DOM1 に 3 を上書き
DOM2 に 1 を上書き
DOM3 に 2 を上書き

```

本来の対応関係を維持できないため、フォーム値の入れ替わりやアニメーションの乱れが発生する。

---

## 5. [[基礎 Part 26 v-for 配列|key]] を付けた場合

```plain text
<div v-for="item in items" :key="item">{{ item }}</div>

```

Vue は [[基礎 Part 26 v-for 配列|key]] を名札として扱い、要素を正しく追跡する。

### 図解

```plain text
key=3 の要素 → 新しい先頭位置へ移動
key=1 の要素 → 二番目へ移動
key=2 の要素 → 三番目へ移動

```

DOM の再利用が正しく行われ、バグを防げる。

---

## 6. 数字を直接回す

```plain text
<div v-for="n in 5" :key="n">{{ n }}</div>

```

Vue 独自仕様であり、1〜5 を自動生成する。

---

## 7. [[基礎 Part 24 v-if|v-if]] と併用しない

NG:

```plain text
<li v-for="item in items" v-if="item.active">{{ item }}</li>

```

OK:

```plain text
<li v-for="item in items.filter(i => i.active)" :key="item.id">{{ item.name }}</li>

```

---

## 8. Memento での利用例

Memento のライフグリッド描画では、寿命×52週の大量セルを [[基礎 Part 26 v-for 配列|v-for]] で生成する。この場合 [[基礎 Part 26 v-for 配列|key]] は必須となる。

```plain text
<div v-for="week in weeks" :key="week.id" class="cell">
  <!-- 現在週やイベント有無で色分け -->
</div>

```

week.[[HTMLのclass, id, style|id]] は誕生日からの経過週を数値化したものであり、一意性が保証されるため [[基礎 Part 26 v-for 配列|key]] に最適である。

---

## 9. 最低限覚えておくこと

[[基礎 Part 26 v-for 配列|v-for]] には必ず [[基礎 Part 26 v-for 配列|key]] を付ける。

```plain text
<div v-for="item in items" :key="item.id">{{ item }}</div>

```

理由の理解は後回しでも問題ないが、付ける習慣を身につけておくとバグを避けられる。

## 関連
- [[基礎 Part 27 v-for オブジェクト]]
- [[基礎 Part 28 v-for 数字]]
- [[基礎 Part 24 v-if]]
