---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-23T08:52:00
aliases: [グローバルCSS, グローバル, global CSS, global]
---
# [[基礎 Part 34 グローバルCSS|グローバルCSS]]設計まとめ

## 1. 前提：Vueの[[ULCOD CSS|CSS]]は基本ローカル

VueのSingle File [[基礎 Part 30 コンポーネント|Component]]（.vue）では、[[ULCOD CSS|CSS]]はデフォルトで**[[基礎 Part 30 コンポーネント|コンポーネント]]単位**に閉じられる設計になっている。

```plain text
<style scoped>
.button {
  color: red;
}
</style>

```

- `scoped` は Vue 独自の仕組み
- この[[ULCOD CSS|CSS]]は「この[[基礎 Part 30 コンポーネント|コンポーネント]]のHTMLにだけ」効く
- 大規模開発では[[ULCOD CSS|CSS]]の影響範囲を制御するため必須

---

## 2. [[基礎 Part 34 グローバルCSS|グローバルCSS]]を適用する2つの方法

### 方法① [[基礎 Part 2 index.htmlとmain.js|main]].ts / [[基礎 Part 2 index.htmlとmain.js|main.js]] で [[ULCOD CSS|CSS]] を [[基礎 Part 11 ESM　Import＆Export|import]] する（王道）

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
- **[[Vite HMR|Vite]]（[[コンパイルとビルド|ビルド]]ツール）の機能**

---

### 方法② [[基礎 Part 33 style scoped|scoped]] を付けない を使う

```plain text
<!-- App.vue -->
<style>
body {
  background: #f5f5f5;
}
</style>

```

### 仕組み

- `scoped` を付けないと、Vueは[[ULCOD CSS|CSS]]を加工しない
- 結果として普通の[[基礎 Part 34 グローバルCSS|グローバルCSS]]になる

```plain text
<style scoped>  → h1[data-v-xxx]
<style>         → h1

```

### 問題点

- [[ULCOD CSS|CSS]]の定義場所が分散する
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

- [[基礎 Part 34 グローバルCSS|グローバル]]に影響するものはここだけ見ればよい
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

- [[基礎 Part 33 style scoped|scoped]]なし[[ULCOD CSS|CSS]]は全アプリに影響
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

## 5. [[Vite HMR|Vite]]の役割まとめ

### [[基礎 Part 11 ESM　Import＆Export|import]] './global.[[ULCOD CSS|css]]' の正体

- JavaScriptは[[ULCOD CSS|CSS]]を[[基礎 Part 11 ESM　Import＆Export|import]]できない
- [[Vite HMR|Vite]]がそれを可能にしている

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

- Vueの[[ULCOD CSS|CSS]]設計は「ローカルが基本」
- [[基礎 Part 34 グローバルCSS|グローバルCSS]]はエントリーポイントで読む
- それを支えているのが[[Vite HMR|Vite]]

この構造を理解していると、

- [[ULCOD CSS|CSS]]が壊れたときの原因特定
- チーム設計
- 長期運用

すべてが楽になる。

---

## 7. 次の理解テーマ（予告）

- [[ULCOD CSS|CSS]]変数を使ったデザインシステム
- [[基礎 Part 33 style scoped|scoped]]でも外側に効かせる :deep
- [[Vite HMR|Vite]]が[[基礎 Part 33 style scoped|scoped CSS]]をどう変換しているか

ここまで理解できていれば、実務レベルの基礎は十分に固まっている。
