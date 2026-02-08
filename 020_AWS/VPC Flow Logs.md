---
base: "[[AWS.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
作成日時: 2025-02-03T12:00:00
aliases: [VPC Flow Logs, vpc flow logs, Flow Logs, flow logs, VPCフローログ, フローログ]
---
# [[VPC Flow Logs]] 基礎〜運用設計まとめ

## 1. [[VPC Flow Logs]]

VPC内を流れる通信（L3/L4レベル）を記録するログ機能。
「誰が・誰に・どのポートで・許可/拒否されたか」を後から確認できる。

* パケットの中身（HTTP内容など）は見えない
* トラブル調査・セキュリティ・監査が主目的

---

## 2. [[VPC Flow Logs|Flow Logs]]はどこで有効化できるか

* VPC単位
* Subnet単位
* [[ENI]]（[[ENI|Elastic Network Interface]]）単位

実務では以下が基本。

* 通常運用：VPC or Subnet単位
* インシデント対応：特定EC2の[[ENI]]単位

---

## 3. [[VPC Flow Logs|Flow Logs]]は何を記録しているか（本質）

[[VPC Flow Logs|Flow Logs]]は「[[ENI]]を通過した通信」を記録する。

```
通信元        AWSネットワーク            通信先
  │                 │                     │
  │  Packet         │                     │
  ├──────────────▶ [ ENI ] ─────────────▶ │
                    │
                Flow Logs
```

* [[ENI]]を通らない通信は記録されない
* 記録されるのは L3/L4（IP・Port・Protocol）

---

## 4. サービス別：通信は見えるか

### EC2

* 常に[[ENI]]を持つ
* すべての通信が対象

### ALB / NLB

* AZごとに[[ENI]]を持つ
* Subnet/VPC単位で[[VPC Flow Logs|Flow Logs]]を有効化すれば記録される

```
[ Internet ]
     │
     ▼
[ ALB ENI (AZ-a) ]
     │
     ▼
[ EC2 / ECS ]

※ ALBのENIはSubnet内に存在
※ Subnet Flow Logsで捕捉される

Client → ALB(ENI) → EC2
```

### Lambda（VPC Lambda）

* 実行時にAWSが[[ENI]]を自動作成
* Subnet内に一時的に存在
* Subnet/VPC [[VPC Flow Logs|Flow Logs]]で通信を確認可能

```
Lambda Invoke
│
▼
[ 一時ENI (10.0.1.x) ]
│
├──▶ RDS (3306)
│
└──▶ NAT Gateway ──▶ Internet (443)

※ 実行終了後、ENIは消える or 再利用される

Lambda実行
  ↓
一時ENI作成（10.0.1.x）
  ↓
RDS / EC2 / NAT Gateway
```

※ VPCに入っていないLambdaは対象外

### RDS / ElastiCache

* [[ENI]]を持つ
* 通信は記録される

---

## 5. [[VPC Flow Logs|Flow Logs]]の典型的なログ例

### Lambda → RDS

```
srcaddr=10.0.1.123 dstaddr=10.0.2.45 srcport=49152 dstport=3306 protocol=6 action=ACCEPT
```

意味：

* Lambda（10.0.1.123）が
* RDS（3306）へ
* TCPで接続し
* 許可された

### Lambda → インターネット（NAT経由）

```
srcaddr=10.0.1.123 dstaddr=54.xxx.xxx.xxx srcport=51012 dstport=443 protocol=6 action=ACCEPT
```

---

## 6. REJECT / ログ無し / ACCEPT の意味

### REJECTが出る

* 原因はNACL

### ログが出ない

* [[セキュリティグループ|Security Group]]で黙殺
* ルートテーブル不備
* [[ENI]]に到達していない

### ACCEPTなのに通信不可

* 戻り通信の[[セキュリティグループ|SG]]不備
* アプリケーションレベルの問題

---

## 7. [[VPC Flow Logs|Flow Logs]]は必ず設定するものか

結論：必須ではないが、以下に該当するVPCはほぼ必須。

* 外部通信がある
* 本番・準本番
* 個人情報・顧客データを扱う
* 説明責任がある

一時的・[[ITと不完全さに惹かれる理由|完全]]検証用VPCは不要な場合もある。

---

## 8. 保存先の設計思想

### CloudWatch Logs

* 即時調査向き
* 高コスト
* 短期保存

### S3

* 長期保存向き
* 監査・セキュリティ調査向き

```
障害対応        → CloudWatch Logs
後追い調査・監査 → S3
```

---

## 9. 保存期間の判断基準

### CloudWatch Logs

目的：直近トラブル対応

* 小規模：7日
* 本番あり：14日
* 組織大きめ：30日

30日超は原則不要。

### S3

目的：説明責任・監査

判断軸：

* 法令・社内規程
* インシデントの発覚遅延リスク
* コスト

実務例：

* 社内ツール：90日
* 顧客向け本番：180日
* 基幹・個人情報：1年

---

## 10. 実際の設定方法

### CloudWatch Logs

* Log Group
* Retention settings
* 無期限は禁止

### S3

* Lifecycle Ruleを設定
* 例：

  * 30日後 Glacier
  * 180日後 削除

---

## 11. 普段の運用スタンス

* 平常時：見ない
* 障害・インシデント時：初めて確認
* 「見る時の手順」を事前に決めておく

---

## 12. 一言まとめ

[[VPC Flow Logs]]は常時監視ツールではない。
事故発生時に事実を説明するための証拠である。

目的・保存期間・コストを言語化できて初めて
「設計された[[VPC Flow Logs|Flow Logs]]運用」になる。
