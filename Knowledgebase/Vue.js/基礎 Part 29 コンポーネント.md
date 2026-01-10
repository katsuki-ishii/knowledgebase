---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-21T23:14:00
---
# Vueコンポーネント基礎まとめ

## 1. コンポーネントとは

コンポーネントとは、画面を構成する部品をひとまとめにしたもの。

見た目・データ・動きをセットで持つ。

例：カウンターボタン、ヘッダー、フッターなど。

---

## 2. ルートコンポーネントと main.js

### 役割分担

- main.js：アプリを起動するファイル
- ルートコンポーネント：画面構造の一番上にあるコンポーネント

### 図解

```plain text
main.js（起動係）
   ↓
App.vue（ルートコンポーネント）
   ↓
子コンポーネント群

```

### 具体例（Vue）

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')

```

この場合、App.vue がルートコンポーネント。

---

## 3. 親コンポーネントと子コンポーネント

### 定義

- 親：template 内で子を呼び出している側
- 子：親に使われている側

### 図解

```plain text
App（親）
 ├─ CountUp（子）
 ├─ CountUp（子）
 └─ CountUp（子）

```

---

## 4. import と template での利用

### import は普通の JavaScript

```javascript
import CountUp from './CountUp.vue'

```

script setup を使っている場合、import した時点で template から使える。

### template では HTML タグのように書く

```plain text
<template>
  <CountUp />
  <CountUp />
  <CountUp />
</template>

```

---

## 5. 同じ子コンポーネントを複数並べた場合

### ポイント

- 同じコンポーネントでも置いた数だけインスタンスが作られる
- 子コンポーネント内で持つ値は、それぞれ独立する

### 具体例

```javascript
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

```

```javascript
<CountUp />
<CountUp />
<CountUp />

```

それぞれが別の count を持つため、連動しない。

---

## 6. 値をどこに持たせるかの違い

### 子が値を持つ場合

```plain text
App
 ├─ CountUp（count = 0）
 ├─ CountUp（count = 0）
 └─ CountUp（count = 0）

```

- 独立する
- 親からまとめて制御できない

### 親が値を持つ場合

```plain text
App（count = 0）
 ├─ CountUp（表示のみ）
 ├─ CountUp（表示のみ）
 └─ CountUp（表示のみ）

```

- 連動できる
- 状態管理が一元化される

---

## 7. コンポーネントタグの書き方は2通り

### PascalCase

```plain text
<CountUp />

```

### kebab-case

```plain text
<count-up></count-up>

```

機能は同じ。

.vue ファイル内の template では PascalCase が推奨される。

---

## 8. PascalCase と2単語以上が推奨される理由

### PascalCase の理由

- HTML タグと見分けがつきやすい
- コンポーネントであることが一目で分かる

```plain text
<div></div>      // HTML
<CountUp />      // コンポーネント

```

### 2単語以上の理由

- HTML の予約語や将来追加されるタグと衝突しにくい
- 人間が読んで混乱しにくい

```plain text
<AppHeader />
<UserProfile />
<BaseButton />

```

---

## 9. 重要な理解のまとめ

- コンポーネントは画面の部品
- main.js は起動係であり、コンポーネントではない
- ルートコンポーネントは App.vue
- 親は子を使う
- 同じ子コンポーネントでも状態は独立する
- PascalCase・2単語以上が設計上安全

## 関連
- [[基礎 Part 30 コンポーネント登録]]
- [[基礎 Part 31  属性フォークスルー]]
- [[基礎 Part 35 defineProps()]]
