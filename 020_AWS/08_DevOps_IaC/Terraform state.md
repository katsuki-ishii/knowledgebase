---
base: "[[AWS.DevOps_IaC.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
作成日時: 2026-02-13T12:00:00
aliases: [Terraform State, terraform state]
---

# [[Terraform State]]

---

## 0. stateとは

Terraformがクラウド上のリソースを管理するために保持する「現在の状態の記録」。

実体はJSONファイル（`terraform.tfstate`）で、どのリソースをどの設定で作ったか、AWSが採番したARNや[[HTMLのclass, id, style|ID]]は何か、を記録している。

```
コード (.tf)  →  「こうあるべき」という理想
state         →  「前回この状態にした」という記録
AWS           →  実際に存在するリソース
```

Terraformは `plan` 実行時にこの3者を照合し、理想と現実の差分だけをAWSに対して実行する。stateがなければ「何が既にあるか」を知る手段がなく、毎回作り直すか何もしないかの二択しかなくなる。

---

## 1. なぜstateが必要か

### AWSのAPIは[[冪等性|冪等]]じゃない

[[冪等性|冪等]]（idempotent）とは「何度実行しても結果が同じになる性質」。

AWSのAPIは操作の種類によって[[冪等性]]が異なる。

```
読み取り系  GET /functions/my-lambda     → 冪等
書き込み系  POST /functions              → 既に存在したら409エラー
           PUT  /functions/my-func       → 存在しないと404エラー
           DELETE /functions/my-func     → 存在しないと404エラー
```

つまり「[[Lambda]]を作って」とAWSに命令するとき、既に存在するかどうかで呼ぶAPIが変わる。

### stateが解決すること

```
コード (.tf)        state (.tfstate)       実際のAWS
  "理想"    ←diff→   "前回の認識"   ←diff→   "現実"
```

Terraformはstateに「前回apply時の状態」を記録することで、次回以降は差分だけを計算できる。AWSのAPIが[[冪等性|冪等]]でなくても、[[冪等性|冪等]]に振る舞えるレイヤーを作っている。

### 具体的な動き

```
# 初回apply
state: (空)  →  CreateFunction APIを呼ぶ  →  stateに記録

# memory_size を 128 → 256 に変更してapply
state: { memory: 128 }  →  差分あり  →  UpdateFunctionConfiguration APIを呼ぶ

# 何も変えずにapply
state: { memory: 256 }, .tf: { memory: 256 }  →  差分なし  →  APIコール0回
```

---

## 2. stateの中身

実体はただのJSON。AWSが生成したARNや[[HTMLのclass, id, style|ID]]も記録される。

```json
{
  "resources": [
    {
      "type": "aws_lambda_function",
      "name": "hello",
      "instances": [
        {
          "attributes": {
            "function_name": "hello",
            "memory_size": 256,
            "arn": "arn:aws:lambda:ap-northeast-1:123456789:function:hello"
          }
        }
      ]
    }
  ]
}
```

ARNがstateに記録されているから、次回applyでも「どのリソースか」を特定できる。

---

## 3. Remote State

### なぜローカルに置いてはいけないか

デフォルトはローカルに `terraform.tfstate` が生成される。これは個人開発専用。

- 競合 ── 複数人が同時にapplyするとstateが壊れる
- 消失 ── ファイルが消えたら管理不能
- 秘密情報 ── DBパスワード等が平文で入る場合がある

### S3 + DynamoDB構成

```
S3バケット         →  stateファイルの保管庫
DynamoDBテーブル   →  「今誰かが使用中」の排他ロック台帳
```

**DynamoDBのロックの仕組み**

```
apply開始  →  DynamoDBにLockレコードをPutItem
              LockID: "my-tfstate/prod/terraform.tfstate"
              Who:    "katsuki@macbook-pro"   ← OS username@hostname（自動取得）

既にレコードがある  →  「ロック中」でエラー（同時実行防止）

apply完了  →  LockレコードをDeleteItem
```

`Who` はTerraformがOSから自動取得する（入力不要）。CI/CDからだと `runner@ip-xxx` になるので、誰が実行したかを追うにはCloudTrailを使う。

### 設定例

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tfstate-bucket"
    key            = "envs/prod/terraform.tfstate"
    region         = "ap-northeast-1"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```

### 暗号化

| 設定 | 暗号化方式 | 備考 |
|------|-----------|------|
| `encrypt = true` のみ | SSE-S3（AWS管理鍵） | 通常はこれで十分 |
| `kms_key_id` も指定 | SSE-KMS（自己管理鍵） | 監査要件が出たら追加 |

---

## 4. State[[基礎 Part 13 分割代入|分割]]の設計思想

### アンチパターン：全部1つのstateに入れる

- [[Lambda]] 1つ変えるためにVPC全体をplanする
- `terraform destroy` が怖すぎる
- 影響範囲が可視化できない

### [[基礎 Part 13 分割代入|分割]]軸は3つある

**軸1：変更頻度**

```
infrastructure/   ← VPC, IAM, S3バケット（変更頻度：低）
application/      ← Lambda, DynamoDB, API Gateway（変更頻度：高）
```

**軸2：ライフサイクル**

```
bootstrap/    ← 一度だけ作る（tfstateバケット自体）
persistent/   ← 長期間維持（VPC, RDS）
ephemeral/    ← 環境ごとに作り直す（Lambda, ECS）
```

**軸3：チーム所有権**

```
infra-team/     ← ネットワーク層
app-team/       ← アプリリソース
security-team/  ← IAM, WAF
```

小規模チームなら軸1か軸2で十分。軸3はチームが複数になってから考える。

### state間の参照

[[基礎 Part 13 分割代入|分割]]するとstate間でVPC [[HTMLのclass, id, style|ID]]などを参照する必要が生まれる。

```hcl
# infrastructure/ 側で出力
output "vpc_id" {
  value = aws_vpc.main.id
}

# application/ 側で参照
data "terraform_remote_state" "infra" {
  backend = "s3"
  config = {
    bucket = "my-tfstate"
    key    = "infrastructure/terraform.tfstate"
  }
}
```

スケールするとSSM Parameter Store経由の疎結合の方が安全。

---

## 5. 重要コマンド

```bash
# 管理中のリソース一覧
terraform state list

# 特定リソースの詳細確認
terraform state show aws_lambda_function.hello

# リソース名の変更（リファクタ時に使う）
# これをやらないと「削除→再作成」になる
terraform state mv aws_lambda_function.old aws_lambda_function.new

# stateから除外（AWSリソースは残したまま管理対象から外す）
terraform state rm aws_lambda_function.hello

# 既存リソースをTerraform管理に取り込む
terraform import aws_lambda_function.hello my-function-name
```

---

## 6. driftに注意

コンソールで手動変更するとstateと現実がズレる。

```
state: { memory: 256 }   ← Terraformの認識
AWS:   { memory: 512 }   ← 現実（手動で変えた）
.tf:   { memory: 256 }   ← コード

→ planすると「512 → 256 に変更あり」と検出される
→ applyすると手動変更が上書きされる
```

Terraformはstateを現実より信頼する。手動変更は原則禁止。

---

## 7. Workspaceの罠

```bash
terraform workspace new staging
terraform workspace new prod
```

一見便利に見えるが、環境差異が出始めた瞬間にカオスになる。環境分離はディレクトリ[[基礎 Part 13 分割代入|分割]]の方が長期的に安全。

---

## まとめ

| 概念 | 一言 |
|------|------|
| state | Terraformが管理するリソースの「前回の記録」 |
| S3 | stateファイルの保管庫 |
| DynamoDB | 同時実行を防ぐロック台帳 |
| drift | stateと現実のズレ |
| state mv | リファクタ時にstateの対応付けを更新するコマンド |
| state rm | 管理対象から外すコマンド（AWSリソースは残る） |
| [[基礎 Part 11 ESM　Import＆Export|import]] | 既存リソースをTerraform管理に取り込むコマンド |
