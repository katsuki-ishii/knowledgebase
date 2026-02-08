---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-13T14:21:00
aliases: [index.htmlとmain.js, index.html, main.js, index, main]
---
# 1. `index.html` の役割

Vue + [[Vite HMR|Vite]] プロジェクトでは、`index.html` はアプリの**エントリーポイント（入口）**である。

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <title>Memento</title>
  </head>
  <body>
    <!-- Vueアプリが描画される場所 -->
    <div id="app"></div>

    <!-- main.js がアプリの起動スクリプト -->
    <script type="module" src="/src/main.js"></script>
  </body>
</html>

```

###  ポイント

| 要素 | 説明 |
| --- | --- |
| `<div id="app"></div>` | Vueアプリが描画される「マウント先」 |
| `<script type="module" src="/src/main.js">` | Vueアプリの起動を担当するスクリプト |

##  2. `main.js` の役割

Vue 3 の標準的な `main.js` の内容：

```javascript
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import './assets/main.css'

const app = createApp(App)
app.use(router)
app.mount('#app')

```

###  解説

| 行 | 内容 | 補足 |
| --- | --- | --- |
| `import { createApp } from 'vue'` | Vueライブラリのアプリ生成関数を読み込む | Vue 3 の Composition API 用構文 |
| `import App from './App.vue'` | アプリ全体の[[基礎 Part 30 コンポーネント|ルートコンポーネント]] | UI とロジックの中心 |
| `import router from './router'` | ページ遷移（Vue Router）を読み込む | SPA のルーティングを管理 |
| `app.use(router)` | ルーターをアプリに登録する | プラグインとして組み込む |
| `app.mount('#app')` | [[基礎 Part 2 index.htmlとmain.js|index.html]] の `#app` に描画を開始する | ここでUIが表示される |

## 3. [[Vite HMR|Vite]] がしている裏側の仕事

[[Vite HMR|Vite]] は `index.html` を起点に依存関係を解析し、ブラウザで動くように最適化する。

```plain text
index.html
   │
   └── <script type="module" src="/src/main.js">
           ↓
           main.js を読み込む
           ↓
           import文をたどって依存ファイルを解決
           ↓
           ブラウザ実行用に変換・配信

```

[[Vite HMR|Vite]] はこれをリアルタイムで行い、ホットリロード（変更即時反映）も提供する。

## 4. ブラウザでの実行の流れ

1. **HTML解析**：`<div id="app">` を見つける（まだ空）
2. `**main.js**`** 読み込み**：`createApp(App)` が呼ばれる
3. **Vueアプリ生成**：`App.vue` が仮想DOMとして構築される
4. **マウント処理**：`#app` に Vue [[基礎 Part 30 コンポーネント|コンポーネント]]が描画される
5. **UI表示完了**

## 5. `/src/main.js` の意味

[[Vite HMR|Vite]] 環境では `/` がプロジェクトルートを表す。

- `/src/main.js` = プロジェクト直下の `src/main.js`
- 本番[[コンパイルとビルド|ビルド]]時には、`/assets/main-xxxx.js` のような最適化済みファイルに変換される

## 6. まとめ

| 要素 | 役割 |
| --- | --- |
| `<script type="module">` | ESモジュールとして読み込み、`import/export`が使用可能になる |
| `src="/src/main.js"` | Vueアプリのエントリーポイント |
| [[Vite HMR|Vite]] | モジュール依存関係を解決し、ブラウザで動作可能な形に変換する |
| `createApp(App).mount('#app')` | Vueアプリを実際のDOMに描画する |

---

> 💡 一言で言うと
> `index.html` は「Vueアプリが描画される舞台」であり、
> 
> `<script type="module" src="/src/main.js">` は「Vueアプリを起動するスイッチ」である。
