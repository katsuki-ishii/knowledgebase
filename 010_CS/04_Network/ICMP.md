---
base: "[[CS.Network.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - CS
作成日時: 2025-02-03T12:00:00
aliases: [ICMP, icmp, Internet Control Message Protocol, インターネット制御メッセージプロトコル]
---
# [[ICMP]]

## 1. [[ICMP]]とは何か

[[ICMP]]（[[ICMP|Internet Control Message Protocol]]）は、IP通信を補助するための制御用プロトコルである。
データ転送を担うTCPやUDPとは異なり、「通信が成立しなかった理由」や「途中経路で起きた事象」を通知する役割を持つ。

[[ICMP]]は単独では存在せず、必ずIPパケットのペイロードとして送信される。

---

## 2. プロトコル階層と関係

```
[ アプリケーション ]  ping / tracert
        |
[ ICMP ]  制御・通知
        |
[ IP ]    ルーティング・TTL管理
        |
[ L2 ]    Ethernetなど
```

* ping / tracert はプロトコルではなくコマンド（アプリケーション）
* [[ICMP]]はL3（ネットワーク層）のプロトコル
* IPヘッダの Protocol フィールドで「次は[[ICMP]]」と示される

包含関係ではなく「利用関係」である。

---

## 3. IPヘッダとTTL

TTL（Time To Live）はIPヘッダに含まれるフィールドであり、通過可能なルータ数を表す。

```
IP Header
+-------------------------+
| TTL | Protocol |  ...   |
+-------------------------+
```

* ルータを1台通過するごとにTTLは1減少
* TTLが0になった時点でパケットは破棄される
* その際、ルータは [[ICMP]] Time Exceeded を送信元に返す

TTLは[[ICMP]]のフィールドではなく、原因はIP、通知が[[ICMP]]という関係。

---

## 4. [[ICMP]]ヘッダ構造

```
0               8              16             24            31
+---------------+---------------+---------------+---------------+
|     Type      |     Code      |        Checksum               |
+---------------+---------------+---------------+---------------+
|            Message-specific Data             |
+---------------------------------------------------------------+
```

### 各フィールドの役割

* [[基礎 Part 37 props バリデーション|Type]]: [[ICMP]]の大分類（何の通知か）
* Code: [[基礎 Part 37 props バリデーション|Type]]の詳細理由
* Checksum: [[ICMP]]ヘッダとデータの整合性確認
* Message-specific Data: [[基礎 Part 37 props バリデーション|Type]]ごとに異なる付加情報

---

## 5. 代表的な[[ICMP]] [[基礎 Part 37 props バリデーション|Type]]

| [[基礎 Part 37 props バリデーション|Type]] | 内容                      | 用途例     |
| ---- | ----------------------- | ------- |
| 8    | Echo Request            | ping送信  |
| 0    | Echo Reply              | ping応答  |
| 3    | Destination Unreachable | 宛先不可通知  |
| 11   | Time Exceeded           | TTL切れ通知 |

---

## 6. pingと[[ICMP]]の関係

pingは[[ICMP]] Echo Request / Echo Replyを利用した疎通確認コマンドである。

```
送信元
  |
  | ICMP Echo Request (Type 8)
  v
宛先
  |
  | ICMP Echo Reply (Type 0)
  v
送信元
```

pingで分かるのは以下のみ:

* IPレベルで届くか
* 相手が応答可能か
* 往復時間

TCPやアプリケーションの状態は分からない。

---

## 7. tracertと[[ICMP]]の関係

tracert（traceroute）はTTLと[[ICMP]] Time Exceededを利用して経路を可視化する。

```
TTL=1  →  ルータA → ICMP Time Exceeded
TTL=2  →  ルータB → ICMP Time Exceeded
TTL=3  →  宛先     → ICMP Echo Reply
```

各ルータはTTL切れ時に[[ICMP]]を返すことで、自身の存在を通知する。

---

## 8. Message-specific Dataの重要性

エラー系[[ICMP]]（[[基礎 Part 37 props バリデーション|Type]] 3 / 11）では、以下が格納される。

```
[ 元のIPヘッダ ]
[ 元データの先頭8バイト ]
```

これにより送信元は「どの通信でエラーが起きたか」を特定できる。
TCPが接続を失敗と判断できるのは、この情報があるため。

---

## 9. AWSにおける[[ICMP]]の扱い（詳細）

AWSでは、[[ICMP]]は設計上「補助的な存在」として扱われており、オンプレミス環境と同じ感覚で利用すると誤解を生みやすい。

### 9.1 [[セキュリティグループ|Security Group]]と[[ICMP]]

[[セキュリティグループ|Security Group]]（[[セキュリティグループ|SG]]）はステートフルなL4ファイアウォールであり、デフォルトでは[[ICMP]]通信を許可しない。

```
Inbound Rules
- TCP 443 0.0.0.0/0
( ICMPなし )
```

この状態では以下が成立する。

* ping 不可
* HTTPSアクセスは正常

これは異常ではなく、AWSでは一般的な構成である。

[[ICMP]]を許可する場合は明示的にルールを追加する必要がある。

```
ICMP - Echo Request - 0.0.0.0/0
```

### 9.2 NACLと[[ICMP]]

Network ACL（NACL）はステートレスなL4フィルタであり、[[ICMP]]を通す場合は以下が必要となる。

* Inbound: [[ICMP]] 許可
* Outbound: [[ICMP]] 許可

片方向のみ許可した場合、pingは成立しない。

---

### 9.3 EC2・マネージドサービスとping

| サービス        | ping    | 理由           |
| ----------- | ------- | ------------ |
| EC2         | 可（[[セキュリティグループ|SG]]次第） | OSが[[ICMP]]に応答可能 |
| ALB         | 不可      | L7ロードバランサ    |
| NLB         | 不可      | L4だが[[ICMP]]非対応  |
| API Gateway | 不可      | HTTP抽象化      |
| Lambda      | 不可      | 実行環境非公開      |

AWSでping可能なのは、原則として「自分でOSを管理しているEC2のみ」である。

---

### 9.4 tracerouteが機能しない理由

AWS VPC内部のルーティングはマネージドであり、以下の制約がある。

* 中継ルータが[[ICMP]] Time Exceededを返さない
* 経路情報はユーザーに公開されない

そのため traceroute / tracert を実行すると以下のようになる。

```
 1  * * *
 2  * * *
 3  * * *
```

これは障害ではなく、仕様である。

---

### 9.5 Path MTU Discoveryと[[ICMP]]

TCP通信では Path MTU Discovery により、最適なパケットサイズを自動調整する。

この仕組みでは以下の[[ICMP]]が利用される。

* [[基礎 Part 37 props バリデーション|Type]] 3 Code 4 (Fragmentation Needed)

[[ICMP]]を過剰に遮断すると以下の現象が起きる。

* 小さい通信は成功する
* 大きいレスポンスで通信が停止する
* タイムアウトが断続的に発生する

AWSではこの問題が表面化しづらいが、[[IPsecとVPN|VPN]]やオンプレ接続では重要となる。

---

### 9.6 AWSにおける疎通確認の正解

AWSでは[[ICMP]]ではなく、以下を基準に疎通を判断する。

```
L7: HTTPステータス (200 / 500)
L4: TCP接続可否
L3: VPC Flow Logs
```

具体例:

```
ping x.x.x.x        -> NG
nc -zv x.x.x.x 443  -> OK
curl https://x.x.x.x -> 200
```

この場合、AWS的には「正常」である。

---

### 9.7 VPC Flow Logsという事実

VPC Flow Logsは、AWSにおける通信の最終的な事実となる。

* ACCEPT / REJECT
* src / dst
* port / protocol

pingやtracerouteが示さない情報を、Flow Logsは確実に示す。

---

### 9.8 AWSで持つべき判断基準

* [[ICMP]]は参考情報
* TCP/HTTPが主判断材料
* ユーザー体験を最優先

AWSでは「通信できるか」ではなく「使えるか」が正義となる。

---

## 10. 重要な思考整理

* TTLはIPの責務
* [[ICMP]]は通知役
* ping / tracert は補助ツール
* 実務では「使えるか」をL4/L7で判断する

ICMPはネットワークを理解するための基礎であり、最終判断基準ではない。
