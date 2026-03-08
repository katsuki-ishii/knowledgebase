---
base: "[[AWS.DevOps_IaC.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
作成日時: 2026-02-13T12:00:00
aliases: [terraform plan, Terraform plan]
---

# [[terraform plan]]

## 本質：3つの状態を突き合わせる差分計算エンジン

```
① コード (desired state)    ← 自分が書いたHCL
② State (recorded state)   ← .tfstate が記憶してる現実
③ 実インフラ (actual state) ← AWSの今の状態
```

この3者を比較して「何を足して、何を消して、何を変えるか」をドライランで出力する。

---

## 処理の流れ

```
1. Refresh      → ②と③を同期（実インフラを読みに行く）
2. Diff         → ①と②を比較してdiffを計算
3. Dependency   → resource graph を構築して順序解決
4. Output       → +/-/~ で変更を表示
```

### 出力記号の意味

| 記号 | 意味 |
|------|------|
| `+` | create |
| `-` | destroy |
| `~` | update in-place |
| `-/+` | destroy & recreate（要注意） |

---

## 実行例：[[Lambda]]関数名を変えたケース

```bash
$ terraform plan

-/+ resource "aws_lambda_function" "handler" {
      ~ function_name = "next-handler" -> "next-handler-v2" # forces replacement
        role          = "arn:aws:iam::123456789:role/lambda-role"
    }

Plan: 1 to add, 0 to change, 1 to destroy.
```

**読み方：最終行の `Plan: x to add, x to change, x to destroy` を最初に確認。destroyが予想外に多かったら上に戻って原因を追う。**

---

## 落とし穴

### ① `-/+` が出たら必ず疑え
`~` と違って一度リソースを**消す**。DynamoDBに出たらデータも消える。

対処：`lifecycle` [[基礎 Part 9ブロックとブロックスコープ|ブロック]]で防御する。

```hcl
lifecycle {
  create_before_destroy = true  # 先に新しいものを作ってから消す
  prevent_destroy       = true  # destroyそのものを禁止
}
```

### ② コンソール操作はStateとズレる
コンソールで変えると③だけが更新されてStateが古いまま残る。次のplanで「知らないリソースを消そう」と判断される可能性がある。

**ルール：触ったら必ずコードに戻す（`terraform import` で取り込む）**

### ③ planは「管理下のリソース」しか見ない
コンソールで手動作成したリソースはplanに出てこない。「destroyが出てないから安全」は管理下のリソースにだけ成立する。

### ④ planが綺麗でもapplyは失敗しうる
planの出力は「最終状態」ではなく「操作」。applyが途中で失敗するとStateが中途半端になる可能性がある。

---

## Dependency（resource graph）

インフラには作る順番がある。Terraformは依存関係を自動解析して正しい順番で実行する。

```
VPC → Subnet → Security Group → Lambda
```

### 依存の書き方

```hcl
# 暗黙的依存（参照があれば自動検出）
resource "aws_subnet" "example" {
  vpc_id = aws_vpc.main.id
}

# 明示的依存（参照なしで順序だけ指定したい場合）
resource "aws_lambda_function" "example" {
  depends_on = [aws_iam_role_policy.example]
}
```

destroyは**逆順**で自動実行される。

### 循環依存エラー

AがBに依存、BがAに依存する状態。planの時点でエラーになる。

```
Error: Cycle: aws_lambda_function.a, aws_iam_role.b
```

**解決策：参照を断ち切る（wildcardに変える）か、リソースを[[基礎 Part 13 分割代入|分割]]して依存の方向を一方向に整理する。**

---

## APIレートリミット

`-refresh=true`（デフォルト）のとき、リソースの数だけAWSのDescribe系APIが飛ぶ。大規模構成ではスロットリングが起きることがある。

```bash
terraform plan -refresh=false   # Refreshをスキップ
terraform plan -parallelism=5   # 並列数を下げて回避（デフォルト10）
```

---

## 主要オプション

```bash
# Refreshをスキップ
terraform plan -refresh=false

# 特定リソースだけplan（デバッグ時。乱用禁止）
terraform plan -target=aws_lambda_function.handler

# 変数ファイルを指定
terraform plan -var-file="prod.tfvars"

# 変数を直接渡す
terraform plan -var="env=prod"

# planの結果をファイルに保存（CI/CDで必須）
terraform plan -out=tfplan

# 終了コードで変更有無を判定（CI/CD自動化で使う）
terraform plan -detailed-exitcode
# 0=変更なし / 1=エラー / 2=変更あり
```

---

## `-out` がCI/CDで必須な理由

planとapplyの間に別の変更が割り込む問題を防ぐ。

```bash
terraform plan -out=tfplan   # planをスナップショットとして保存
terraform apply tfplan        # 保存したplanだけを実行
```

CI/CDのパターン：

```
1. plan -out=tfplan  → PRレビュー時に実行
2. 承認
3. apply tfplan      → 承認されたplanだけを実行
```

planとapplyを**セットで一つの意図**として扱う。

---

## planは[[基礎 Part 21 副作用と純粋関数|副作用]]ゼロ

何度叩いてもインフラは変わらない。**怖かったらとりあえずplan。**
