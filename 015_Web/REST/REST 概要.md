---
base: "[[Web.REST.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Web
作成日時: 2025-08-03T15:12:00
aliases: [REST 概要, REST, rest]
---
## 1. そもそも [[REST 概要|REST]] って何？

- [[REST 概要|REST]]（Representational State Transfer）は、

> Webのように
> - 多数のクライアント
> - ネットワーク越し
> - スケールする
> 
> システムを**壊れにくく・拡張しやすく作る**ための設計原則

HTTP・[[URI・URL・URN|URL]]・[[設定ファイル言語|JSON]] を使うのは **手段**であって **目的ではない**。

- **Representational State Transfer** の略。
- 2000 年、Roy Fielding 博士論文で命名。
- Web 全体を成功させた“設計思想”を抽象化した **アーキテクチャスタイル**。

### 1.1 命名と起源

- HTTP/1.0→1.1 仕様策定にも携わった Fielding が、実装経験を分析して体系化。
- "リソースの状態 (State)" を "表現 (Representation)" として "転送 (Transfer)" する、という発想。

### 1.2 アーキテクチャスタイルとは

- MVC や Pipe&Filter と同じく、具体的製品より 1 段抽象度が高い設計テンプレート。
- **パターン (設計図)** より粒度が大きく、実装 (ブラウザ・サーバ・Apache など) より抽象。

---

## 2. [[REST 概要|REST]] を構成する 6 つのスタイル（[[ULCOD CSS|ULCOD]] [[ULCOD CSS|CSS]]）

| 頭文字 | スタイル | 要点 |
| --- | --- | --- |
| **U** | **Uniform Interface** | [[URI・URL・URN|URI]] + HTTP メソッド + 表現形式 を共通手順化 |
| **L** | **Layered System** | プロキシやロードバランサを多段に挟んでも OK |
| **C** | **Client / Server** | UI とデータ保管を分離。マルチプラットフォーム対応◎ |
| **C** | **Cache** | リソースの鮮度に応じてクライアント側で再利用し帯域削減 |
| **S** | **Stateless Server** | サーバはクライアント状態を保持しない → 水平スケール容易 |
| **C** | **Code on Demand** (任意) | JS などコード配布でクライアント機能を拡張 |

> 覚え方: [[ULCOD CSS|ULCOD]] [[ULCOD CSS|CSS]]（ウルコド・シス）

---

## 3. リソース指向設計

1. **名付ける** – [[URI・URL・URN|URI]] を設計し Web 上で一意に。
2. **表現する** – HTML / [[設定ファイル言語|JSON]] / [[設定ファイル言語|XML]] / 画像など多様な Representation。
3. **状態を移送する** – HTTP メソッドで取得・更新・削除。

### 用語

- **Resource**: Web 上の情報の抽象。（例）東京の天気、論文 PDF、写真。
- **[[URI・URL・URN|URI]]**: Resource の名前札。
- **Representation**: ある時点の状態を表すデータ。より具体的には、HTML や [[設定ファイル言語|JSON]]、[[設定ファイル言語|XML]]、画像、PDF などの**ファイル形式やデータフォーマット**のことを指し、リソースの内容をどのように表現・転送するかを決定するもの。

---

## 4. [[REST 概要|REST]] と分散システム

| RPC 型 | [[REST 概要|REST]] 型 |
| --- | --- |
| メソッド呼び出し主体 → 細粒度 | リソースにリンク → 粗粒度 |
| 新メソッド追加で互換性問題 | [[URI・URL・URN|URI]] 追加/表現変更で済む |
| スタブ更新の運用コスト大 | クライアント汎用 HTTP で接続 |

---

## 5. HATEOAS（Hypermedia As The Engine Of Application State）

- HATEOAS は [[REST 概要|REST]] の重要な制約の一つで、クライアントがサーバの提供するハイパーメディア（リンク）を通じて、アプリケーションの状態遷移を行うことを意味します。
- これにより、クライアントはサーバの [[URI・URL・URN|URI]] や仕様を事前に知る必要がなく、返されるリンクを辿ることで動作を継続できるため、変更に強い設計が可能になります。

### 具体例：

たとえば、`/orders/123` を GET したときに次のようなレスポンスが返る場合：

```json
{
  "orderId": 123,
  "status": "processing",
  "links": [
    { "rel": "cancel", "href": "/orders/123/cancel", "method": "POST" },
    { "rel": "track", "href": "/orders/123/track", "method": "GET" }
  ]
}

```

このように、次にできる操作（キャンセルや追跡など）をリンクとして提供することで、クライアントは仕様を知らずともナビゲート可能になります。

---

## 6. [[REST 概要|REST]] のメリット

- HTTP インフラをそのまま利用 → ファイアウォール越えやすい。
- ステートレス + キャッシュで高スケーラビリティ。
- シンプル API → 開発者学習コスト低い。

---