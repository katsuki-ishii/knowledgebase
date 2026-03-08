---
base: "[[AWS.DevOps_IaC.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
作成日時: 2026-02-13T12:00:00
aliases:
  - Terraform Modules
  - terraform modules
  - Modules
  - Module
---

# [[Terraform Modules]]

## [[Terraform Modules|Modules]]とは何か

Terraformにおいて、すべてのコードは[[Terraform Modules|Module]]の中にある。普段書いている `main.tf` があるディレクトリ自体が **Root [[Terraform Modules|Module]]** であり、`module` [[基礎 Part 9ブロックとブロックスコープ|ブロック]]で呼び出すものが **Child [[Terraform Modules|Module]]** である。[[Terraform Modules|Module]]は特別な追加機能ではなく、Terraformの実行単位そのもの。

本質的には「関数」に近い。HCLのリソース定義を再利用可能な単位にパッケージングする仕組みであり、`variable`（引数）で入力を受け取り、`output`（戻り値）で結果を返す。

```
呼び出し側 (Root Module)
│
│  source    = モジュールの場所
│  variable  = 入力値
│
▼
Module (Child Module)
│
│  main.tf      → リソース定義
│  variables.tf → 入力の定義
│  outputs.tf   → 出力の定義
│
▼
output → 呼び出し側に返る値
```

---

## [[Terraform Modules|Modules]]の本質的な役割

### カプセル化

[[Terraform Modules|Module]]内部のリソースは外から直接参照できない。`output` で明示的に公開したものだけが外に見える。プログラミングにおけるスコープと同じ概念。

### 抽象化

呼び出し側は「何が作られるか」の詳細を知らなくていい。`variable` と `output` というインターフェースだけで対話する。

### 構成の単位

`terraform plan` の時、[[Terraform Modules|Module]]単位で依存グラフが構築される。Terraformはこのグラフを見て並列実行できるリソースを判断する。

---

## なぜ[[Terraform Modules|Modules]]が必要か

### DRY（Don't Repeat Yourself）

同じリソース定義をDev/Eval/Prodでコピペすると、同じコードが3回出現する。[[Terraform Modules|Modules]]はこの構造的な重複を排除する。

ただしインフラにおけるDRYは、通常のコードの「メンテナンスコスト削減」より一段深い意味を持つ。インフラの重複は **環境間の構成ドリフト**（Dev/Eval/Prodが気づかないうちに乖離していくこと）を引き起こす。

[[Terraform Modules|Module]]で定義を一元化すると、差分は `variables` に閉じ込められる。「何が環境間で違うのか」が[[AI激動期における「定数」思考|変数]]定義を見るだけで明確になる。

より正確には **Single Source of Truth（信頼できる唯一の定義元）** という表現のほうが近い。DRYは手段であり、目的は「インフラの構成がコードから一意に決定できる状態を保つこと」。

### 関心事の分離（Separation of Concerns）

「1つのまとまりは1つの責務だけを持つべき」という設計原則。

```
【水平方向の分離（再利用）】

  Dev ──┐
        ├── networking Module
  Prod ─┘

  → 同じModuleを複数環境で使い回す


【垂直方向の分離（関心事）】

  同一環境内：
  ┌─────────────────┐
  │ networking Module │ ← VPC, サブネット, SG
  ├─────────────────┤
  │ cdn Module        │ ← CloudFront, WAF
  ├─────────────────┤
  │ data Module       │ ← DynamoDB
  └─────────────────┘

  → 責務が違うリソース群を別Moduleに切る
```

垂直方向の分離により、変更の影響範囲が予測可能になる。全リソースが1つのRoot [[Terraform Modules|Module]]にベタ書きだと `plan` の出力が長大になり、意図した変更だけかどうかが読めなくなる。

再利用は結果であって目的ではない。1回しか使わなくても、関心事の分離として意味があるなら[[Terraform Modules|Module]]化する価値はある。ただし、そのメリットと間接参照のコスト（ファイルを跨いで読む手間）を天秤にかける判断が必要。

---

## [[基礎 Part 1 ディレクトリ構造|ディレクトリ構造]]

```
modules/
  networking/
    main.tf
    variables.tf
    outputs.tf
  cloudfront/
    main.tf
    variables.tf
    outputs.tf

envs/
  dev/
    main.tf        ← moduleを呼び出す側
  eval/
    main.tf
  prod/
    main.tf
```

呼び出し側の記述：

```hcl
module "networking" {
  source = "../../modules/networking"

  vpc_cidr     = "10.0.0.0/16"
  environment  = "dev"
  project_name = "next"
}
```

`source` がモジュールの場所、それ以降が `variables.tf` で定義した入力[[AI激動期における「定数」思考|変数]]に渡す値。

---

## Stateとの関係

[[Terraform Modules|Module]]を使うとStateに記録されるリソースのアドレスが変わる。

```
# Module未使用
aws_vpc.main

# Module使用
module.networking.aws_vpc.main

# Moduleのネスト
module.networking.module.subnet.aws_subnet.main
```

[[Terraform Modules|Module]]のネスト ＝ Stateのアドレスの深さ。深くしすぎると `state mv` や `import` が困難になる。実務的には **2階層まで** がメンテ可能な限界。

**重要：既存リソースを後から[[Terraform Modules|Module]]に移す場合、`terraform state mv` が必要。** これを知らずにリファクタリングすると、Terraformが「古いリソースを削除→新しいリソースを作成」と解釈して本番環境が破壊される。

---

## [[Terraform Modules|Module]]化の判断基準

### Rule of Three

同じパターンが3回出てきたら抽象化する。2回では早い。

- 1回目 → そのまま書く
- 2回目 → 「似てるな」と気づくが、コピペでいい
- 3回目 → 共通構造が確定する。ここで初めて[[Terraform Modules|Module]]化

2回の時点では、共通部分と差分の境界が見えきっていない。3回目で「ここは本当に共通で、ここが[[AI激動期における「定数」思考|変数]]になるべき」というパターンが確信に変わる。

Terraformでは通常のコードより早すぎる抽象化のコストが高い。State移動が発生し、[[AI激動期における「定数」思考|変数]]設計を間違えると全環境に波及する。

### ただし自明なものは最初から[[Terraform Modules|Module]]化していい

Rule of Threeは「迷ったら慎重に」という判断基準であり、「絶対に3回待て」ではない。

判断の軸は **「このリソース群の構造が今後変わる可能性があるか？」**。

- 低い → 最初から[[Terraform Modules|Module]]化していい（例：Route53のレコード構造）
- 高い → ベタ書きで始めて、構造が安定してから[[Terraform Modules|Module]]化する

不確実性が高い時にRule of Threeが効く。確実性が高い時に待つ必要はない。ルールが守ろうとしているもの（早すぎる抽象化のリスク）を理解した上で、リスクがないなら先にやる。

### 何でもかんでも[[Terraform Modules|Module]]化しない

[[Terraform Modules|Module]]は抽象化のレイヤーを1つ追加する行為。読む人が追うべきコードの階層が増える。

例えばセキュリティグループ1個だけの[[Terraform Modules|Module]]を作ると、リソース定義を見に行くのに `main.tf → module呼び出し → modules/sg/main.tf → variables.tf → outputs.tf` と3ファイル跨ぐ。ベタ書きなら1ファイルで完結する。

抽象化のコストが再利用のメリットを上回ったら、それはただの間接参照の迷路。

```
【Module化すべきかのフローチャート】

そのリソース群を2回以上使うか？
├── Yes → 構造が自明か？
│         ├── Yes → 最初からModule化
│         └── No  → Rule of Three で判断
└── No  → リソース数が多く関心事の分離が必要か？
          ├── Yes → Module化を検討
          └── No  → ベタ書きで十分
```

---

## 実務上の注意点

### [[AI激動期における「定数」思考|変数]]の設計

1つの[[Terraform Modules|Module]]で[[AI激動期における「定数」思考|変数]]5〜10個程度が健全な範囲。それを大きく超えていたら、[[Terraform Modules|Module]]の責務が広すぎるサイン。「環境ごとに本当に変わる値」だけを[[AI激動期における「定数」思考|変数]]にする。

### SAMとの責務分担

デプロイ頻度が異なるリソースは、ツールとStateを分ける。

```
Terraform（変更頻度：低）
  → VPC, Route53, CloudFront, WAF, ACM, DynamoDB...

SAM（変更頻度：高）
  → Lambda, API Gateway, Cognito Authorizer...
```

同じツール・同じStateで管理すると、[[Lambda]]を1個直したいだけなのにCloudFrontまで `plan` の対象になり、意図しない差分が出たりデプロイが遅くなる。

### 推奨ステップ

1. まずフラットに書く → `terraform apply` に慣れる
2. Dev環境で動くものができたら、Prod/Evalとの差分を見て共通部分を特定
3. その共通部分を[[Terraform Modules|Module]]に切り出す

早すぎる抽象化はTerraformでも敵。実際のリソース定義が固まってから[[Terraform Modules|Module]]化するほうが、[[AI激動期における「定数」思考|変数]]設計の判断が正確になる。
