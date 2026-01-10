---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
作成日時: 2025-10-04T15:01:00
---
> 目的：エラーの正体を素早く特定し、最短ルートで直す。EC2 に限らず、他の AWS サービスでも役立つ共通の仕組みとしてまとめました。

---

## 症状の例

```plain text
An error occurred (UnauthorizedOperation) when calling the RunInstances operation: You are not authorized to perform this operation. Encoded authorization failure message: <…>

```

このエラーは **IAM 権限の不足/制約** が原因で起きるときに返ってきます。EC2 に限らず、ECS/Lambda/Batch/KMS/S3 などのサービスでも発生します。共通するのは「エンコードされた失敗メッセージ」が含まれており、これを **デコード** することで詳細が確認できる点です。

---

## 最短トリアージ（90秒）

- **メッセージをデコード**
```bash
aws sts decode-authorization-message --encoded-message <ペースト>

```
→ これには `sts:DecodeAuthorizationMessage` 権限が必要。
- **DecodedMessage の要点を確認**
    - `action`（例: `ec2:RunInstances`, `iam:PassRole`, `kms:Decrypt`, `s3:GetObject`）
    - `resource`（どのリソースに対して拒否か）
    - `condition`（リージョン制約、タグ条件など）
- **どの層でブロックかを特定**
    - IAM ポリシー / Permission Boundary / SCP / セッションポリシー / リソースポリシー / KMS キー

---

## 代表的な原因と対処

### A. `iam:PassRole` 不足（ECS/Lambda/EC2/Batch で頻出）

- **症状**：デコード結果に `iam:PassRole` が登場。実行主体がロールを他のサービスに渡せない。
- **対処**：必要なロールに絞って `iam:PassRole` を付与。

### B. サービスアクション自体が Deny

- **例**：`ec2:RunInstances`, `ecs:RunTask`, `s3:PutObject` など。
- **対処**：実行主体に該当アクションを Allow。明示的 Deny がないか確認。

### C. KMS 暗号化リソースの権限不足

- **対象サービス**：EBS/S3/RDS/Secrets Manager など。
- **症状**：`kms:Decrypt` / `kms:CreateGrant` の不足で失敗。
- **対処**：キーポリシーと IAM ポリシー両方で許可を追加。必要ならサービスリンクドロールも追加。

### D. リソース可視性/共有設定の問題

- **例**：プライベート AMI、スナップショット、S3 バケットの共有漏れ。
- **対処**：共有設定やリソースポリシーを確認。

### E. 条件付き制約（タグやリージョン）

- **例**：`aws:RequestedRegion` やタグ条件による Deny。
- **対処**：Condition ブロックを調整。

### F. サービスクォータ/容量不足

- **症状**：`LimitExceeded` や `InsufficientInstanceCapacity`。
- **対処**：クォータ引き上げ申請や別リソース利用。

---

## デコードが「出る/出ない」パターン

- **出るケース**：EC2/ECS/Lambda/Batch/KMS など、IAM ポリシーや SCP/Boundary が拒否の原因となった場合。
- **出ないケース**：
    - シンプルに `AccessDenied: User is not authorized to perform: s3:ListBucket` のように返る場合。
    - 容量不足やクォータ超過など権限以外の理由。
    - 信頼ポリシー違反（AssumeRole 失敗など）。

---

## 実務チェックリスト

- [ ] **エラー全文をコピー**し、`Encoded authorization failure message` が含まれているか確認
- [ ] 含まれていれば `**aws sts decode-authorization-message**` を実行
- [ ] **DecodedMessage** 内の `action`, `resource`, `condition`, `decision` を確認
- [ ] `iam:PassRole` や `kms:Decrypt` の不足がないかを重点チェック
- [ ] IAM ポリシー / Permission Boundary / SCP / リソースポリシーを確認
- [ ] タグやリージョン制約など **Condition による制御**を見直す
- [ ] KMS キー利用時は **IAM ポリシーとキーポリシーの両方**を確認
- [ ] **シンプルな AccessDenied** の場合はメッセージ通りの権限を追加
- [ ] サービスクォータや容量不足なら decode せず **クォータ・リソース状況**を確認

---

## 便利コマンド

```bash
# 認可失敗メッセージのデコード
aws sts decode-authorization-message --encoded-message <ENCODED>

# サービスクォータ確認
aws service-quotas list-service-quotas --service-code ec2

# S3 バケットポリシー確認
aws s3api get-bucket-policy --bucket <BUCKET_NAME>

```

---

## 参考リソース

- [AWS re:Post: EC2 インスタンスの起動失敗に対する UnauthorizedOperation エラーをデコードして分析する方法](https://repost.aws/ja/knowledge-center/ec2-not-auth-launch)
- [AWS CLI ドキュメント: sts decode-authorization-message](https://docs.aws.amazon.com/cli/latest/reference/sts/decode-authorization-message.html)
- [IAM ドキュメント: インスタンスプロファイル](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)
- [EC2 User Guide: Troubleshooting Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/troubleshooting.html)
- [AWS KMS Developer Guide: Using Grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html)

---

## まとめ

- **EC2 に特有ではなく、AWS 全般で利用できる仕組み**。
- **常に出るわけではない**が、出たときは decode が最強のデバッグ手段。
- 出なければ通常の `AccessDenied` をそのまま読んで対処。
