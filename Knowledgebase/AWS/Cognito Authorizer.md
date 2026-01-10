---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
作成日時: 2026-01-08T01:13:00
---
# API Gateway Cognito Authorizer まとめ

## Cognito Authorizerとは？

API Gateway Cognito Authorizer とは、Cognito が発行した JWT（ログイン証明）を API Gateway がリクエスト受付時に検証する仕組みである。

目的は、

- 未ログイン・不正ユーザーを API の入口で遮断すること
- Lambda を「認証済み前提」でシンプルに保つこと

である。

---

## 登場人物

### Cognito

- AWS の認証専門サービス
- ユーザー登録、ログイン、パスワード管理を担当
- ログイン成功時に JWT を発行する

### JWT（JSON Web Token）

- ログイン済みであることを示す改ざん不可能なトークン
- 有効期限、ユーザーIDなどが含まれる

### API Gateway

- API の入口
- リクエストを受け取り、問題なければ Lambda に渡す

### Lambda

- 実際のビジネス処理を書く場所
- データ取得・保存などを担当

---

## 全体構成（図解）

```plain text
[ユーザー]
    |
    | ログイン
    v
[Cognito]
    |
    | JWT発行
    v
[フロントエンド]
    |
    | Authorization: Bearer JWT
    v
[API Gateway]
    |  Cognito Authorizer
    |  ・JWT検証
    |  ・有効期限チェック
    v
[Lambda]
    |
    | ビジネス処理
    v
[レスポンス]

```

---

## Cognito Authorizer がやっていること

API Gateway が Lambda を呼ぶ前に以下を行う。

1. Authorization ヘッダから JWT を取得
2. JWT が Cognito 発行かを検証
3. 有効期限が切れていないかを確認
4. OKなら Lambda を実行
5. NGなら 401 Unauthorized を返す

Lambda は一切 JWT 検証をしない。

---

## Lambda 側のコードの変化

### Lambda 内検証の場合

```javascript
if (!verifyJwt(event.headers.authorization)) {
  return { statusCode: 401 }
}

```

### Cognito Authorizer を使う場合

```javascript
const claims = event.requestContext.authorizer.claims
const userId = claims.sub

```

認証コードが消え、ユーザー情報を使うだけになる。

---

## claims とは何か

claims は JWT の中身で、API Gateway が展開して Lambda に渡す。

代表例：

- sub: ユーザーID
- email: メールアドレス
- cognito:groups: 所属グループ

---

## 認証と認可の役割分担

- 認証（誰か）: Cognito Authorizer
- 認可（何をしていいか）: Lambda

具体例：

```javascript
if (!claims['cognito:groups']?.includes('admin')) {
  return { statusCode: 403 }
}

```

---

## コストについて

- Cognito Authorizer 自体は無料
- API Gateway の料金は変わらない
- Cognito User Pool は 5万 MAU まで無料
- 不正リクエストで Lambda が起動しないため、むしろ安くなる場合がある

---

## Lambda 内検証との比較

| 観点 | Cognito Authorizer | Lambda内検証 |
| --- | --- | --- |
| 認証場所 | API入口 | Lambda内部 |
| セキュリティ | 高い | 低い |
| Lambdaの責務 | 純粋 | 混在 |
| 実務での採用 | 王道 | 限定的 |

---

## 使うべき場面

- 本番API
- 外部公開API
- チーム開発
- 長期運用プロダクト

---

## 一文まとめ

API Gateway Cognito Authorizer は、ログイン確認を API の入口で完結させ、Lambda を安全かつシンプルに保つための標準的な設計である。