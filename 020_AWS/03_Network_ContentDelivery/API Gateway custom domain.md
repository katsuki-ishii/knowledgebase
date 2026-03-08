---
base: "[[AWS.Network_ContentDelivery.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
作成日時: 2026-02-13T00:00:00
aliases: [API Gateway custom domain, API Gatewayカスタムドメイン, カスタムドメイン, Custom Domain, custom domain]
---
## 1. API Gatewayのデフォルト[[URI・URL・URN|URL]]とは何か

API Gatewayを作成すると、以下のような[[URI・URL・URN|URL]]が自動で発行される。

```
https://abc123.execute-api.ap-northeast-1.amazonaws.com/prod
```

このドメインはAWSが管理するパブリックDNSゾーン
`execute-api.amazonaws.com` に属している。

名前解決の流れは以下の通り。

```
クライアント
   ↓
ISP DNS
   ↓
ルートDNS
   ↓
amazonaws.com
   ↓
execute-api.amazonaws.com
   ↓
AWS管理DNS
   ↓
IPアドレス返却
```

つまり、execute-apiの[[URI・URL・URN|URL]]はAWSが管理するDNS基盤で名前解決される。

---

## 2. [[API Gateway custom domain|カスタムドメイン]]とは何か

[[API Gateway custom domain|カスタムドメイン]]とは、自分が管理するドメイン名をAPI Gatewayに割り当てる機能である。

例：

```
api.prod.katsu-boy.com
```

これをAPI Gatewayに紐付けることで、execute-apiの[[URI・URL・URN|URL]]を隠蔽できる。

内部構造は以下のようになる。

```
api.prod.katsu-boy.com
        ↓
Route53（Aliasレコード）
        ↓
API Gateway カスタムドメイン
        ↓
Lambda / ECS
```

---

## 3. なぜ[[API Gateway custom domain|カスタムドメイン]]が必要か

主な理由はブランディングではなく、設計の抽象化である。

### 3.1 抽象化

フロントエンドは以下の[[URI・URL・URN|URL]]だけを知っていればよい。

```
https://api.prod.katsu-boy.com
```

内部が

* API Gateway
* ALB
* ECS
* 別リージョン

に変わっても影響を受けない。

### 3.2 環境分離

```
api.dev.katsu-boy.com
api.stg.katsu-boy.com
api.prod.katsu-boy.com
```

環境ごとの分離が明確になる。

### 3.3 セキュリティ管理

* ACM証明書管理
* WAF連携
* CORS管理

が整理しやすくなる。

---

## 4. DNSの基本挙動

DNSは「[[ITと不完全さに惹かれる理由|完全]]一致」を優先する。

例：

```
katsu-boy.com              → フロント
api.prod.katsu-boy.com     → APIGW
```

この場合、以下のように正しく分岐する。

```
https://katsu-boy.com
  → フロント

https://api.prod.katsu-boy.com
  → API Gateway
```

両者は[[ITと不完全さに惹かれる理由|完全]]に別ホストである。

---

## 5. ワイルドカードレコードの影響

以下のようなレコードがある場合は注意が必要。

```
*.katsu-boy.com → フロント
```

この場合、api.prod.katsu-boy.com もフロントに向かう。

DNSの優先順位は以下。

1. [[ITと不完全さに惹かれる理由|完全]]一致
2. ワイルドカード
3. 存在しなければNXDOMAIN

---

## 6. Route53で必要な設定

### 6.1 APIGW側

* [[API Gateway custom domain|カスタムドメイン]]作成
* ACM証明書紐付け
* エンドポイントタイプ選択（Regional推奨）

### 6.2 Route53側

作成するレコード：

* 種類：Aレコード
* エイリアス：Yes
* ターゲット：API Gateway[[API Gateway custom domain|カスタムドメイン]]

例：

```
api.prod.katsu-boy.com
  → Alias → d-abc123.execute-api.ap-northeast-1.amazonaws.com
```

---

## 7. SPA構成との関係

典型的な構成は以下。

```
app.katsu-boy.com   → CloudFront + S3
api.katsu-boy.com   → API Gateway
```

ブラウザから見ると以下の通信が発生する。

```
GET https://api.katsu-boy.com/users
```

一般ユーザーはこれを意識しないが、技術的には公開HTTPエンドポイントである。

---

## 8. CORSとオリジン

以下は別オリジンである。

```
prod.katsu-boy.com
api.prod.katsu-boy.com
```

そのため、CORS設定が必要になる。

特に[[Cookie]]認証を使う場合は以下に注意。

* SameSite属性
* Domain属性
* credentials: include

---

## 9. 設計としての整理例

将来拡張を考慮した構成例。

```
katsu-boy.com            → LP
app.katsu-boy.com        → SPA
api.katsu-boy.com        → API
admin.katsu-boy.com      → 管理画面
```

環境分離。

```
app.dev.katsu-boy.com
api.dev.katsu-boy.com

app.prod.katsu-boy.com
api.prod.katsu-boy.com
```

---

## 10. 本質的な理解

* execute-apiドメインも[[API Gateway custom domain|カスタムドメイン]]も本質的にはDNS解決である
* 違いは「誰がそのゾーンを管理しているか」
* [[API Gateway custom domain|カスタムドメイン]]は抽象化レイヤーである
* [[API Gateway custom domain|DNS設計]]はアーキテクチャ設計そのものである

API Gatewayの[[API Gateway custom domain|カスタムドメイン]]は、見た目のための機能ではなく、将来変更耐性と構成抽象化のための設計要素である。
