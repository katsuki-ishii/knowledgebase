---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-23T08:52:00
---
# グローバルCSS設計まとめ

## 1. 前提：VueのCSSは基本ローカル

VueのSingle File Component（.vue）では、CSSはデフォルトで**コンポーネント単位**に閉じられる設計になっている。

```plain text
<style scoped>
.button {
  color: red;
}
</style>

```

- `scoped` は Vue 独自の仕組み
- このCSSは「このコンポーネントのHTMLにだけ」効く
- 大規模開発ではCSSの影響範囲を制御するため必須

---

## 2. グローバルCSSを適用する2つの方法

### 方法① main.ts / main.js で CSS を import する（王道）

```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'

import './assets/reset.css'
import './assets/variables.css'
import './assets/global.css'

createApp(App).mount('#app')

```

### 何が起きているか（図解）

```plain text
main.ts
  |
  |-- import reset.css
  |-- import variables.css
  |-- import global.css
  v
Vite が解析
  v
HTML に <style> / <link> として注入
  v
ページ全体にCSSが適用される

```

- Vueの機能ではない
- JavaScriptの標準でもない
- **Vite（ビルドツール）の機能**

---

### 方法② scoped を付けない を使う

```plain text
<!-- App.vue -->
<style>
body {
  background: #f5f5f5;
}
</style>

```

### 仕組み

- `scoped` を付けないと、VueはCSSを加工しない
- 結果として普通のグローバルCSSになる

```plain text
<style scoped>  → h1[data-v-xxx]
<style>         → h1

```

### 問題点

- CSSの定義場所が分散する
- 影響範囲がコードから分かりにくい
- 大規模開発では保守コストが高い

---

## 3. なぜ方法①が大規模開発に向いているのか

### 理由① 影響範囲の入口が一か所

```plain text
main.ts
  ├─ reset.css   （全体初期化）
  ├─ variables.css（デザイン定数）
  └─ global.css  （共通ルール）

```

- グローバルに影響するものはここだけ見ればよい
- 影響調査が早い

---

### 理由② チーム開発で事故が起きにくい

```plain text
<style>
button {
  background: red;
}
</style>

```

- scopedなしCSSは全アプリに影響
- 新人・外注が書くと致命的事故になりやすい

一方でルールをこう分けると安全になる。

```plain text
assets/        → グローバルCSSのみ
components/    → scoped CSSのみ

```

---

## 4. 実務での典型的な構成

```plain text
src/
├─ assets/
│  ├─ reset.css
│  ├─ variables.css
│  └─ global.css
├─ components/
│  └─ Button.vue（scoped）
├─ App.vue
└─ main.ts

```

### 役割分担

- assets/: デザインルール・全体方針
- components/: 見た目の実装

---

## 5. Viteの役割まとめ

### import './global.css' の正体

- JavaScriptはCSSをimportできない
- Viteがそれを可能にしている

```plain text
JavaScriptコード
  ↓
Viteが解析
  ↓
HTML/CSSに変換
  ↓
ブラウザで有効化

```

Vueはここではほぼ関与しない。

---

## 6. 本質的な理解

- VueのCSS設計は「ローカルが基本」
- グローバルCSSはエントリーポイントで読む
- それを支えているのがVite

この構造を理解していると、

- CSSが壊れたときの原因特定
- チーム設計
- 長期運用

すべてが楽になる。

---

## 7. 次の理解テーマ（予告）

- CSS変数を使ったデザインシステム
- scopedでも外側に効かせる :deep
- Viteがscoped CSSをどう変換しているか

ここまで理解できていれば、実務レベルの基礎は十分に固まっている。

## 関連
- [[基礎 Part 32 style scoped]]
- [[基礎 Part 23 class指定]]
