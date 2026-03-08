---
base: "[[AWS.DevOps_IaC.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
作成日時: 2026-02-12T12:00:00
aliases:
  - Terraform ブロック
  - ブロック
---

# [[Terraformブロック|Terraformブロック]]

## [[基礎 Part 9ブロックとブロックスコープ|ブロック]]とは

HCLの基本単位。Terraformのコードは全部[[基礎 Part 9ブロックとブロックスコープ|ブロック]]の組み合わせで構成される。

構造は「[[基礎 Part 9ブロックとブロックスコープ|ブロック]]タイプ + ラベル + 中身」の3要素。

```hcl
ブロックタイプ "ラベル1" "ラベル2" {
  引数 = 値

  ネストブロック {
    引数 = 値
  }
}
```

[[基礎 Part 9ブロックとブロックスコープ|ブロック]]の中に[[基礎 Part 9ブロックとブロックスコープ|ブロック]]を入れることができる（ネスト[[基礎 Part 9ブロックとブロックスコープ|ブロック]]）。

---

## ラベルとは

[[基礎 Part 9ブロックとブロックスコープ|ブロック]]を識別するための名前。ラベルの数は[[基礎 Part 9ブロックとブロックスコープ|ブロック]]タイプによって決まっている。

`resource` の場合はラベルが2つ：

```hcl
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-bucket-name"
}
```

- ラベル1 `"aws_s3_bucket"` → 何を作るか（Providerが定義したリソース種別）
- ラベル2 `"my_bucket"` → Terraform内での呼び名（ローカル名）

他の[[基礎 Part 9ブロックとブロックスコープ|ブロック]]から参照するときは `リソース種別.ローカル名.属性値` の形で使う：

```hcl
resource "aws_s3_bucket_policy" "policy" {
  bucket = aws_s3_bucket.my_bucket.id
}
```

ラベルの数は[[基礎 Part 9ブロックとブロックスコープ|ブロック]]タイプによって異なる：

| [[基礎 Part 9ブロックとブロックスコープ]]タイプ | ラベル数 | 理由 |
|---|---|---|
| `resource` | 2 | 種別 + ローカル名 |
| `provider` | 1 | どのProviderか |
| `terraform` | 0 | 設定が1つしか存在しないから |

---

## 属性とは

[[基礎 Part 9ブロックとブロックスコープ|ブロック]]の中身に書く設定値。`キー = 値` の形式。

属性には2種類ある：

**引数（Arguments）** → 自分が設定する値

```hcl
bucket = "my-bucket-name"
```

**属性値（Attributes）** → Terraform/AWSが返してくる値

```hcl
aws_s3_bucket.my_bucket.id   # AWSが採番したID
aws_s3_bucket.my_bucket.arn  # AWSが決めるARN
```

引数は自分で書く。属性値はapply後にstateから参照できる。

---

## 依存関係の仕組み

属性値を参照するだけで、リソース間の作成順序が自動的に決まる。

```
aws_s3_bucket.my_bucket
        ↓ .id を参照
aws_s3_bucket_policy.policy
```

この参照関係からTerraformがDAG（有向非巡回グラフ）を構築し、依存のないリソースは並列で作成される。

注意点として、同じ値でも文字列で直書きすると依存関係が生まれない：

```hcl
# NG：依存関係が生まれない
bucket = "my-bucket-name"

# OK：依存関係が生まれる
bucket = aws_s3_bucket.my_bucket.id
```

---

## 4つの主要[[基礎 Part 9ブロックとブロックスコープ|ブロック]]

```
terraform {}     → Terraform自体の動作設定
provider {}      → クラウドへの接続設定
resource {}      → 作るリソースの定義
data {}          → 既存リソースの読み込み
```

### terraform {}

Terraform本体の動作設定を書く場所。

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket = "my-tfstate"
    key    = "next/terraform.tfstate"
    region = "ap-northeast-1"
  }
}
```

書けるのは3つ：

- `required_version` → Terraform本体のバージョン縛り
- `required_providers` → 使うProviderとバージョン。`terraform init`でここを読んでダウンロードする
- `backend` → stateファイルの保存先。書かなければローカルに保存される

**この[[基礎 Part 9ブロックとブロックスコープ|ブロック]]だけ[[AI激動期における「定数」思考|変数]]や参照が使えない。** Terraformが起動する一番最初に読まれるため、その時点では[[AI激動期における「定数」思考|変数]]が解決されていないから。

### provider {}

ProviderはTerraformとクラウドAPIの間の翻訳者。Terraform本体はAWSのことを何も知らず、APIへの変換を担うのがProviderの役割。実体としてはAWS SDKをTerraformから呼び出せるようにラップしたプラグイン。

```
.tfファイル
    ↓
AWS Provider（SDKのラッパー）
    ↓
AWS SDK（Go製）
    ↓
AWS API（HTTP）
    ↓
実際のリソース
```

`provider {}` [[基礎 Part 9ブロックとブロックスコープ|ブロック]]はそのProviderへの接続設定を書く場所：

```hcl
provider "aws" {
  region  = "ap-northeast-1"
  profile = "my-profile"
}
```

`alias` を使うと同じProviderを複数定義できる（マルチリージョン構成など）：

```hcl
provider "aws" {
  region = "ap-northeast-1"
  alias  = "tokyo"
}

provider "aws" {
  region = "us-east-1"
  alias  = "virginia"
}
```

### resource {}

実際に作るインフラを定義する場所。最もよく書く[[基礎 Part 9ブロックとブロックスコープ|ブロック]]。

```hcl
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-bucket-name"

  tags = {
    Environment = "dev"
  }
}
```

`lifecycle` ネスト[[基礎 Part 9ブロックとブロックスコープ|ブロック]]で挙動を制御できる：

```hcl
resource "aws_lambda_function" "my_func" {
  function_name = "my-function"

  lifecycle {
    prevent_destroy       = true    # 削除を防ぐ
    ignore_changes        = [tags]  # タグの変更を無視
    create_before_destroy = true    # 新しいものを作ってから古いものを消す
  }
}
```

本番リソースには `prevent_destroy = true` を入れておくと安全。

### data {}

既存のリソースを読み込む場所。自分では作らない。applyしても何も変わらない。

```hcl
data "aws_vpc" "main" {
  id = "vpc-xxxxxxxx"
}

resource "aws_subnet" "new" {
  vpc_id = data.aws_vpc.main.id  # 既存VPCの中にサブネットを作る
}
```

`resource` と `data` の違い：

| | resource | data |
|---|---|---|
| 役割 | 作る | 読む |
| applyの影響 | 作成・変更・削除される | 何も変わらない |

Terraform管理外のリソース（手動作成・別のTerraformで管理）を参照するときや、AWSが管理している情報（アカウント[[HTMLのclass, id, style|ID]]など）を取得するときに使う：

```hcl
data "aws_caller_identity" "current" {}

resource "aws_s3_bucket_policy" "policy" {
  bucket = aws_s3_bucket.my_bucket.id
  policy = jsonencode({
    Principal = {
      AWS = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:root"
    }
  })
}
```

---

## 全体像

```
terraform {}
  └── Terraform本体の設定（バージョン・Provider・backend）

provider {}
  └── クラウドへの接続設定（リージョン・認証）

resource {}
  └── 作るリソースの定義
        └── lifecycle {} でライフサイクル制御

data {}
  └── 既存リソースの読み込み（何も作らない）
```

この4つの[[基礎 Part 9ブロックとブロックスコープ|ブロック]]で、Terraformのコードの大部分は表現できる。
