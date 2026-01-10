---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-13T22:59:00
---
### HMRとは

Hot Module Replacement（ホットモジュールリプレースメント）は、アプリケーションを再読み込みせずに、変更したコード部分だけをリアルタイムでブラウザに反映する仕組み。ページ全体をリロードせずに済むため、状態を保持したまま高速に開発できる。

### ViteにおけるHMRの仕組み

Viteは開発サーバーを立ち上げる際に、各モジュールをESM（ECMAScript Modules）形式で配信する。次のような流れでHMRが動作する。

1. ブラウザがViteサーバーからモジュールをimportして読み込む。
2. 開発中にファイルが変更されると、Viteがその変更を検知する。
3. サーバーがWebSocketを通じてブラウザに「どのモジュールが更新されたか」を通知する。
4. ブラウザは対象モジュールを再取得して差し替える。
5. VueやReactなどのフレームワークが差し替えに応じて再描画を行う。

これにより、ページ全体のリロードをせずに変更箇所だけが更新される。

### WebSocketの役割

HMRでは、Viteサーバーとブラウザの間でWebSocketが使われている。WebSocketはサーバーとクライアント間で常時接続を維持し、双方向通信を可能にする。これにより、サーバーが「ファイルが変更された」ときに、即座にブラウザへ通知を送ることができる。

### 実際の動作イメージ

ブラウザ側のHMRクライアント（擬似コード）

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

このように、Viteは内部でWebSocketサーバーを立て、ファイル変更をリアルタイムで通知する。

### なぜWebSocketなのか

| 通信方式 | 特徴 | HMRへの適性 |
| --- | --- | --- |
| HTTP (ポーリング) | 定期的に問い合わせが必要 | 遅延が大きく非効率 |
| SSE (Server-Sent Events) | 一方向通信のみ可能 | 制限あり |
| WebSocket | 双方向・リアルタイム通信が可能 | 最適 |

Viteは即時性と双方向性を重視してWebSocketを採用している。

### Vue + Viteの組み合わせ

Vue 3（Composition API）とViteを組み合わせた場合、`@vitejs/plugin-vue` がHMRを自動的に統合する。`.vue` ファイルの変更時に、templateやscript部分のみ差し替えが行われ、状態を保持したまま更新される。

### まとめ

| 項目 | 内容 |
| --- | --- |
| 名称 | Hot Module Replacement（HMR） |
| 目的 | ページを再読み込みせずに変更箇所のみ更新する |
| 通信方式 | WebSocket（ws://localhost:5173/__vite_ws） |
| 仕組み | サーバーがファイル変更を検知しWebSocketで通知、ブラウザが該当モジュールを差し替え |
| 利点 | 即時反映、状態保持、高速な開発体験 |
| 関連技術 | ESM（ECMAScript Modules）、WebSocket、Vue HMR runtime |

## 関連
- [[Web Socket]]
- [[コンパイルとビルド]]
