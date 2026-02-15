---
base: "[[Web.REST.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Web
作成日時: 2025-02-13T12:00:00
aliases: [OpenAPI基礎まとめ, OpenAPI, openapi]
---
## 1. [[OpenAPI]]とは何か

[[OpenAPI]]とは、[[REST 概要|REST]] APIの仕様を機械可読な形式（[[設定ファイル言語|YAML]] / JSON）で記述するための標準仕様である。

これは実装コードではない。APIがどのように振る舞うべきかを定義する「契約書」である。

[[OpenAPI]]自体はAPIを実行しない。データを返さない。ロジックも持たない。

定義するのは以下である。

* どの[[URI・URL・URN|URL]]が存在するか
* どの[[HTTPメソッド GET|メソッド]]が使えるか
* どのような入力を受け取るか
* どのような出力を返すか
* どのようなエラーが起こり得るか

---

## 2. [[OpenAPI]]の全体構造

[[OpenAPI]]ファイルは大きく以下の構造を持つ。

```
openapi
info
servers
paths
components
security
tags
externalDocs
```

最も重要なのは以下の2つである。

* paths
* components

---

## 3. 基本構造の例

```
openapi: 3.0.3

info:
  title: Sample API
  version: 1.0.0

paths:
  /users:
    get:
      summary: ユーザー一覧取得
      responses:
        '200':
          description: 成功
```

構造図で表すと以下の通りである。

```
OpenAPI
 ├── info
 ├── paths
 │    └── URL
 │         └── method
 │              ├── parameters
 │              ├── requestBody
 │              └── responses
 └── components
```

---

## 4. pathsの役割

pathsはAPI本体である。

ここでは以下を定義する。

* エンドポイント
* 利用可能なメソッド
* 入力
* 出力

例：

```
paths:
  /users/{id}:
    get:
      responses:
        '200':
          description: 成功
```

重要なのは、ここには実装は書かれないという点である。

ここに書かれるのは「このAPIはこう振る舞うべきである」という[[基礎 Part 8 宣言・代入・再代入|宣言]]のみである。

---

## 5. responsesの意味

responsesでは、APIが返すステータスコードとレスポンス構造を定義する。

例：

```
responses:
  '200':
    description: 成功
  '404':
    description: 見つからない
```

これは実際のレスポンスを生成するものではない。

あくまで契約である。

実際に200や404を返すのは実装側（例：Lambda）である。

---

## 6. componentsとは何か

componentsは再利用可能な定義の置き場である。

これは実行されない。
値を持たない。

型定義や共通部品をまとめておく場所である。

### 6.1 schemas

データ構造の定義を行う。

例：

```
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
```

これは「Userという型」を[[基礎 Part 8 宣言・代入・再代入|宣言]]しているだけである。

### 6.2 $[[基礎 Part 4 ref関数|ref]]による参照

定義したschemaは以下のように参照する。

```
schema:
  $ref: '#/components/schemas/User'
```

参照構造の図：

```
components
 └── schemas
      └── User

paths
 └── /users/{id}
      └── get
           └── responses
                └── $ref → User
```

componentsの定義を変更すれば、参照している全箇所に反映される。

---

## 7. componentsで定義できるもの

schemas以外にも以下が定義可能である。

* responses
* parameters
* requestBodies
* headers
* securitySchemes
* examples
* links
* callbacks

実務で特に重要なのは以下である。

* schemas
* responses
* securitySchemes

---

## 8. [[OpenAPI]]と実装の関係

[[OpenAPI]]は契約書である。

実装はその契約を守る責任を持つ。[[HTTPとRESTの関係|HTTPとREST]]の文脈では、[[REST 概要|REST]]的なAPIの「仕様」を[[OpenAPI]]が担う。

関係図は以下の通りである。

```
OpenAPI（契約）
      ↓
API Gateway（入口）
      ↓
Lambda（実装）
```

[[OpenAPI]]が自動的に実装を保証するわけではない。

実装が契約に従う必要がある。

---

## 9. [[OpenAPI]]の本質

[[OpenAPI]]の本質は以下である。

* APIのインターフェースを[[ITと不完全さに惹かれる理由|完全]]に明文化する
* 曖昧さを排除する
* 再利用可能な設計を可能にする
* 型生成や自動ドキュメント生成を可能にする

---

## 10. まとめ

[[OpenAPI]]は[[REST 概要|REST]] APIの仕様を定義するための標準形式である。

実装ではない。

中心は以下である。

* pathsで振る舞いを定義する
* componentsで共通部品を定義する
* $[[基礎 Part 4 ref関数|ref]]で参照する

[[OpenAPI]]はAPI設計の基盤であり、実装より一段抽象度の高いレイヤーに存在するものである。
