---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-13T22:59:00
aliases: [Vite HMR, HMR, Vite, vite, hmr]
---
### [[Vite HMR|HMR]]とは

Hot Module Replacement（ホットモジュールリプレースメント）は、アプリケーションを再読み込みせずに、変更したコード部分だけをリアルタイムでブラウザに反映する仕組み。ページ全体をリロードせずに済むため、状態を保持したまま高速に開発できる。

### [[Vite HMR|Vite]]における[[Vite HMR|HMR]]の仕組み

[[Vite HMR|Vite]]は開発サーバーを立ち上げる際に、各モジュールを[[基礎 Part 11 ESM　Import＆Export|ESM]]（ECMAScript Modules）形式で配信する。次のような流れで[[Vite HMR|HMR]]が動作する。

1. ブラウザが[[Vite HMR|Vite]]サーバーからモジュールを[[基礎 Part 11 ESM　Import＆Export|import]]して読み込む。
2. 開発中にファイルが変更されると、[[Vite HMR|Vite]]がその変更を検知する。
3. サーバーが[[Web Socket|WebSocket]]を通じてブラウザに「どのモジュールが更新されたか」を通知する。
4. ブラウザは対象モジュールを再取得して差し替える。
5. VueやReactなどのフレームワークが差し替えに応じて再描画を行う。

これにより、ページ全体のリロードをせずに変更箇所だけが更新される。

### [[Web Socket|WebSocket]]の役割

[[Vite HMR|HMR]]では、[[Vite HMR|Vite]]サーバーとブラウザの間で[[Web Socket|WebSocket]]が使われている。[[Web Socket|WebSocket]]はサーバーとクライアント間で常時接続を維持し、双方向通信を可能にする。これにより、サーバーが「ファイルが変更された」ときに、即座にブラウザへ通知を送ることができる。

### 実際の動作イメージ

ブラウザ側の[[Vite HMR|HMR]]クライアント（擬似コード）

```javascript
const socket = new WebSocket("ws://localhost:5173/__vite_ws");

socket.addEventListener("message", (event) => {
  const payload = JSON.parse(event.data);
  if (payload.type === "update") {
    import(payload.path + "?t=" + Date.now()).then(newModule => {
      hotReplace(payload.path, newModule);
    });
  }
});

```

サーバー側の挙動（擬似コード）

```javascript
fileWatcher.on("change", (filePath) => {
  wss.send({
    type: "update",
    path: filePath,
  });
});

```

このように、[[Vite HMR|Vite]]は内部で[[Web Socket|WebSocket]]サーバーを立て、ファイル変更をリアルタイムで通知する。

### なぜ[[Web Socket|WebSocket]]なのか

| 通信方式 | 特徴 | [[Vite HMR|HMR]]への適性 |
| --- | --- | --- |
| HTTP (ポーリング) | 定期的に問い合わせが必要 | 遅延が大きく非効率 |
| SSE (Server-Sent Events) | 一方向通信のみ可能 | 制限あり |
| [[Web Socket|WebSocket]] | 双方向・リアルタイム通信が可能 | 最適 |

[[Vite HMR|Vite]]は即時性と双方向性を重視して[[Web Socket|WebSocket]]を採用している。

### Vue + [[Vite HMR|Vite]]の組み合わせ

Vue 3（Composition API）と[[Vite HMR|Vite]]を組み合わせた場合、`@vitejs/plugin-vue` が[[Vite HMR|HMR]]を自動的に統合する。`.vue` ファイルの変更時に、templateやscript部分のみ差し替えが行われ、状態を保持したまま更新される。

### まとめ

| 項目 | 内容 |
| --- | --- |
| 名称 | Hot Module Replacement（[[Vite HMR|HMR]]） |
| 目的 | ページを再読み込みせずに変更箇所のみ更新する |
| 通信方式 | [[Web Socket|WebSocket]]（ws://localhost:5173/__vite_ws） |
| 仕組み | サーバーがファイル変更を検知し[[Web Socket|WebSocket]]で通知、ブラウザが該当モジュールを差し替え |
| 利点 | 即時反映、状態保持、高速な開発体験 |
| 関連技術 | [[基礎 Part 11 ESM　Import＆Export|ESM]]（ECMAScript Modules）、[[Web Socket|WebSocket]]、Vue [[Vite HMR|HMR]] runtime |

## 関連
- [[Web Socket]]
- [[コンパイルとビルド]]
