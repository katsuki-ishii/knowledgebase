---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-13T17:15:00
aliases: [App.vue, App, app.vue]
---
# [[基礎 App.vue|App.vue Part 3]] とは

Vue.js アプリ全体の「[[基礎 コンポーネント|ルートコンポーネント Part 29]]（root [[基礎 コンポーネント|component Part 29]]）」であり、アプリの土台となる部分。`main.js` から読み込まれ、アプリ全体をブラウザの `#app` にマウントする。

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App)
  .use(router)
  .mount('#app')

```

[[基礎 App.vue|App.vue Part 3]] は「全ページ共通の構造」を定義する場所。例えばヘッダー・フッター・ルーター表示領域（`<RouterView />`）を配置する。

---

## [[基礎 App.vue|App.vue Part 3]] の基本構成

```plain text
<template>
  <div id="app" class="min-h-screen bg-gray-50">
    <Navbar />
    <RouterView />
    <Footer />
  </div>
</template>

<script setup>
import Navbar from './components/Navbar.vue'
import Footer from './components/Footer.vue'
</script>

<style scoped>
#app {
  min-height: 100vh;
}
</style>

```

### 構成のポイント

| 部分 | 内容 |
| --- | --- |
| `<template>` | アプリ全体のレイアウト構造を定義する |
| `<script setup>` | Composition API の簡潔な構文でロジックを記述する |
| `<style scoped>` | [[基礎 コンポーネント|コンポーネント Part 29]]固有のスタイルを適用する |

---

## [[基礎 App.vue|App.vue Part 3]] の責務

- アプリ全体のレイアウト（Header/Footer/RouterView）を定義する
- ページ遷移領域（`<RouterView>`）を提供する
- [[基礎 グローバルCSS|グローバル Part 33]]状態やテーマを監視する
- 全体スタイル（背景やレイアウト）を設定する

[[基礎 App.vue|App.vue Part 3]] 自体は「アプリの枠組み」を担当し、ビジネスロジック（API 呼び出し・状態管理）は持たない。

---

## [[基礎 App.vue|App.vue Part 3]] に書いてはいけないこと（アンチパターン）

### 1. [[基礎 グローバルCSS|グローバル Part 33]] [[ULCOD CSS|CSS]] を直接書く

[[基礎 App.vue|App.vue Part 3]] に `scoped` なしで [[ULCOD CSS|CSS]] を書くと、全[[基礎 コンポーネント|コンポーネント Part 29]]に影響してしまう。

```plain text
<style>
button {
  background: red; /* 全てのボタンが赤くなる */
}
</style>

```

**代わりにすること：**

- [[基礎 グローバルCSS|グローバル Part 33]] [[ULCOD CSS|CSS]] は `src/assets/main.css` にまとめる
- Tailwind クラスで完結させる

```javascript
// main.js
import './assets/main.css'

```

### 2. API 呼び出しを直接書く

[[基礎 App.vue|App.vue Part 3]] は構造を管理する場所であり、データ取得は composable に分離すべき。

```javascript
// 悪い例: App.vue に直接 axios 呼び出し
onMounted(async () => {
  const res = await axios.get('/api/user')
})

// 良い例: useUser.js に分離
export function useUser() {
  const user = ref(null)
  onMounted(async () => {
    user.value = await axios.get('/api/user')
  })
  return { user }
}

```

### 3. ロジックを詰め込みすぎる

[[基礎 App.vue|App.vue Part 3]] に認証処理・設定管理・状態共有などをすべて書くと、肥大化して可読性と保守性が低下する。

**解決策：** ロジックは `composables` に切り出し、[[基礎 App.vue|App.vue Part 3]] ではそれを呼び出すだけにする。

### 4. ページごとに共通要素を再レンダリングする

`App.vue` 内でヘッダーやフッターをルーター外側に配置せず、各ページごとに再定義すると、不要な再描画が起きる。

**解決策：** 共通要素は `App.vue` 内で一度だけレンダリングする。

### 5. [[基礎 グローバルCSS|グローバル Part 33]]な[[基礎 副作用と純粋関数|副作用 Part 20]]をここで実行する

ログイン検証や[[基礎 イベントオブジェクト|イベント Part 15]]リスナー登録などの[[基礎 副作用と純粋関数|副作用 Part 20]]を [[基礎 App.vue|App.vue Part 3]] に書くと、他の部分に影響する。

**解決策：** `onMounted` などの[[基礎 副作用と純粋関数|副作用 Part 20]]は専用 composable（例: `useAuth.js`）にまとめる。

### 6. [[基礎 グローバルCSS|グローバル Part 33]]スタイルをハードコードする

`App.vue` に直接 `style` タグでカラーテーマやフォントを定義すると、テーマ変更やカスタマイズが困難になる。

**解決策：** Tailwind や [[ULCOD CSS|CSS]] 変数を用いてテーマを柔軟に管理する。

---

## ベストプラクティスまとめ

| 項目 | 内容 |
| --- | --- |
| Composition API (`<script setup>`) | 最新の記法でシンプルかつ型安全 |
| composable の活用 | ロジック・状態管理は `useXxx.js` に分離する |
| スコープ付き [[ULCOD CSS|CSS]] (`scoped`) | 他[[基礎 コンポーネント|コンポーネント Part 29]]への影響を防ぐ |
| Tailwind の利用 | [[基礎 グローバルCSS|グローバル Part 33]] [[ULCOD CSS|CSS]] を最小限に抑え、統一デザインを保つ |
| [[基礎 App.vue|App.vue Part 3]] は構造に専念 | ロジックや[[基礎 副作用と純粋関数|副作用 Part 20]]を持たない |
| 共通[[基礎 コンポーネント|コンポーネント Part 29]]を明確に分離 | 再利用性を高める |

---

## まとめ

- [[基礎 App.vue|App.vue Part 3]] はアプリ全体の骨格を担う
- 機能ロジック（API 通信・状態管理）は composable に切り出す
- スタイルは `scoped` または Tailwind によってローカル化する
- [[基礎 App.vue|App.vue Part 3]] に[[基礎 グローバルCSS|グローバル Part 33]] [[ULCOD CSS|CSS]] やビジネスロジックを入れるのは責務分離違反
- 共通 UI [[基礎 コンポーネント|コンポーネント Part 29]]は [[基礎 App.vue|App.vue Part 3]] に、動的ロジックは composable に分離する

[[基礎 App.vue|App.vue Part 3]] は「家の骨組み」であり、ロジックや API 通信は「部屋の中の動作」である。

## 関連
- [[基礎 Part 39 index.htmlとmain.js Part 2]]
- [[基礎 setup構文 Part 10]]
- [[基礎 コンポーネント]]
