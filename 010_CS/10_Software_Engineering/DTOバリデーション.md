---
base: "[[CS.Software_Engineering.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - CS
作成日時: 2026-01-08T00:58:00
aliases: [DTOバリデーション, DTO, バリデーション, validation, DTO validation]
---
# [[DTOバリデーション]]まとめ

---

## 1. [[DTOバリデーション]]とは

**[[DTOバリデーション]]とは、外部から入ってくるデータが「期待した形・型・制約」を満たしているかを、業務処理に入る前にチェックする仕組みである。**

- [[DTOバリデーション|DTO]] = Data Transfer Object（データ受け渡し専用の入れ物）
- [[DTOバリデーション|バリデーション]] = 正しさの検証

目的はただ一つ。

> ビジネスロジックに不正なデータを入れないこと

---

## 2. なぜ[[DTOバリデーション|DTO]]で[[DTOバリデーション|バリデーション]]するのか

### 問題のある構造

```plain text
リクエスト
   ↓
handler / Controller
   ↓
Service（途中でif文チェック）
   ↓
DB

```

- if文が散らばる
- チェック漏れが起きる
- Serviceが汚れる

### 望ましい構造

```plain text
リクエスト
   ↓
DTOバリデーション（関門）
   ↓
Service（正しいデータ前提）
   ↓
DB

```

[[DTOバリデーション]]は**入口の関門**である。

---

## 3. 「実体は関数か？」への答え

- 内部的には **関数の集合**
- 実務では **自分で関数を書かない**
- ルールを[[基礎 Part 8 宣言・代入・再代入|宣言]]すると、フレームワークが自動実行する

### イメージ

```plain text
@IsEmail()   → メール形式チェック関数
@Min(0)     → 最小値チェック関数

↓

validate(dto) が裏で実行される

```

---

## 4. Lambda文脈での[[DTOバリデーション]]

### 素朴な実装（原始的）

```typescript
export const handler = async (event) => {
  if (!event.name) {
    return { statusCode: 400 }
  }
}

```

考え方としては正しいが、実務ではスケールしない。

---

### 実務的な進化（middleware化）

```plain text
API Gateway
   ↓
Validation Middleware（DTOバリデーション）
   ↓
Lambda handler（業務処理のみ）

```

- handlerに入る前にチェック
- NGなら handler は呼ばれない

---

## 5. 具体例：ユーザー作成API

### リクエスト

```json
{
  "name": "田中",
  "email": "tanaka@example.com",
  "age": 20
}

```

### [[DTOバリデーション|DTO]]（制約の[[基礎 Part 8 宣言・代入・再代入|宣言]]）

```typescript
class CreateUserDto {
  name: string      // 必須、文字列
  email: string     // メール形式
  age: number       // 0以上の整数
}

```

ここでは「処理」は書かず、「ルール」だけを書く。

---

### 実行フロー（図解）

```plain text
① リクエスト到達
② JSON → DTOに変換
③ DTOバリデーション実行
   ├ OK  → handler / Service 実行
   └ NG  → 400エラー即返却

```

Serviceは「正しいデータが来る」前提で書ける。

---

## 6. [[DTOバリデーション]]でやること／やらないこと

### やること（形式チェック）

- 必須かどうか
- 型（文字列・数値など）
- 長さ・範囲
- フォーマット（メールなど）

### やらないこと（重要）

- 権限チェック
- 在庫チェック
- 「このユーザーは登録可能か？」などの業務ルール

これらは **Serviceの責務** である。

---

## 7. [[DTOバリデーション]]の本質

- handlerの先頭にif文を書く話ではない
- NestJSでもLambdaでも構造は同じ

```plain text
外部入力
   ↓
共通の関門（DTOバリデーション）
   ↓
業務ロジック

```

**「必ず一度通る入口で、自動的にチェックされる」**

これが[[DTOバリデーション]]の本質である。

---

## 8. まとめ

- [[DTOバリデーション]]は入力チェックの仕組み
- 実体は関数だが、[[基礎 Part 8 宣言・代入・再代入|宣言]]的に使う
- Lambdaでは middleware として実装されることが多い
- 目的は Service を安全かつシンプルに保つこと

[[DTOバリデーション]]は設計の話であり、単なる技術テクニックではない。