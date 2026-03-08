---
base: "[[AWS.Compute.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
作成日時: 2026-02-12T12:00:00
aliases:
  - Lambda ライフサイクル
  - Init
  - Invoke
  - Shutdown
  - ウォームスタート
  - コールドスタート
---

# [[Lambda ライフサイクル]]

## ライフサイクルは「1回の呼び出し」ではなく「1つの microVM の一生」

[[Lambda]] の実行環境には3つのフェーズがある。

```
Init（初期化）→ Invoke（実行）→ Shutdown（終了）
```

ここで重要なのは、これが「1回のリクエスト」の話ではなく、**1つの microVM の一生**の話だということ。

```
時間 ──────────────────────────────────────────────────→

リクエスト①  ─→ [Init] → [Invoke] → 待機...
リクエスト②  ─→                      [Invoke] → 待機...    ← ウォームスタート
リクエスト③  ─→                                  [Invoke] → 待機...
                                                   ...しばらく来ない...
                                                   [Shutdown] → microVM 破棄
```

[[Lambda ライフサイクル|Init]] と [[Lambda ライフサイクル|Shutdown]] は microVM の生死に対応していて、[[Lambda ライフサイクル|Invoke]] だけが繰り返される。**[[Lambda ライフサイクル|ウォームスタート]]とは「同じ microVM が再利用されること」**に他ならない。

---

## [[Lambda ライフサイクル|Init]] フェーズ：最初の1回だけの特権

[[Lambda ライフサイクル|Init]] フェーズは3つのサブフェーズに分かれている。

```
┌─────────────────── Init フェーズ ───────────────────┐
│                                                    │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │  Extension  │→ │   Runtime    │→ │  Function  │ │
│  │    Init     │  │    Init      │  │    Init    │ │
│  │             │  │              │  │            │ │
│  │ Datadog等の │   │ Pythonや     │  │ グローバル  │ │
│  │ Extension   │  │ Node.jsの    │  │ スコープの   │ │
│  │ が起動       │  │ ランタイム    │  │ コード実行   │ │
│  └─────────────┘  └──────────────┘  └────────────┘ │
│                                                    │
│  ← AWS が管理 ──────────────────→  ← ユーザーの領域 → │
└─────────────────────────────────────────────────────
```

**① Extension [[Lambda ライフサイクル|Init]]** — [[Lambda]] Extensions（モニタリングツールやセキュリティエージェントなど）が初期化される。

**② Runtime [[Lambda ライフサイクル|Init]]** — Python インタープリタや Node.js ランタイムが起動する。

**③ Function [[Lambda ライフサイクル|Init]]** — **ここがユーザーのコードの[[基礎 Part 34 グローバルCSS|グローバル]]スコープ**。handler 関数の外に書いたコードが全てここで実行される。

```python
# ===== ここから Function Init =====
import boto3
import os

ENV = os.environ['ENVIRONMENT']
bedrock = boto3.client('bedrock-runtime')
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('NextUsers')
# ===== ここまで Function Init =====

def handler(event, context):
    # ===== ここから Invoke =====
    response = table.get_item(Key={'id': event['user_id']})
    return response
```

### [[Lambda ライフサイクル|Init]] に処理を寄せる理由

[[Lambda ライフサイクル|Init]] で作った[[基礎 Part 15 オブジェクト|オブジェクト]]はメモリ上に残り続ける。[[Lambda ライフサイクル|ウォームスタート]]時、handler が呼ばれたとき `bedrock` も `table` もすでに生きている。毎回クライアントを作り直すのと比べて、ウォーム時のレイテンシが大きく変わる。

### [[Lambda ライフサイクル|Init]] に置くべきもの / 置くべきでないもの

**置くべきもの：** boto3 クライアントの生成、テーブル参照の取得、環境[[AI激動期における「定数」思考|変数]]の読み込み。これらは軽量で、ミリ秒〜数百ミリ秒で終わる。

**置くべきでないもの：** 外部 API 呼び出し、大量データの読み込み、タイムアウト未設定のネットワーク通信。時間が読めないので handler 内でやるか、やるにしてもタイムアウトを必ず設定すべき。

原則は「**[[Lambda ライフサイクル|Init]] には確実に速く終わるものだけ置く**」。

---

## [[Lambda ライフサイクル|Init]] のタイムアウト

[[Lambda ライフサイクル|Init]] には独立した固定のタイムアウト枠があるわけではない。**通常の [[Lambda]] では [[Lambda ライフサイクル|Init]] と [[Lambda ライフサイクル|Invoke]] で1つのタイムアウト枠を共有している。**

ただし、関数タイムアウトが10秒未満の場合でも、**[[Lambda ライフサイクル|Init]] には最低10秒が保証される**。

```
関数タイムアウト = 3秒 の場合：
  Init には最低10秒が保証される → Init 完了後、Invoke に3秒

関数タイムアウト = 30秒 の場合：
  Init + Invoke 合計で30秒
```

つまり10秒は「上限」ではなく「最低保証」。

### [[Lambda ライフサイクル|Init]] が重いと handler の持ち時間が削られる

[[Lambda ライフサイクル|Init]] でタイムアウトすること自体はそうそう起きない。実務的に怖いのは **handler の持ち時間が圧迫されること**。

```
関数タイムアウト: 10秒

ケースA（Init 軽い）:
  |■■ Init 0.5秒 |■■■■■■■■■■■■■■■■■■■ handler 9.5秒        |
  0s                                                          10s

ケースB（Init 重い）:
  |■■■■■■■■ Init 4秒 |■■■■■■■■■■■■ handler 6秒              |
  0s                                                          10s

ケースC（Init が handler を圧殺）:
  |■■■■■■■■■■■■■■■■ Init 8秒 |■■ handler 2秒 → タイムアウト💥|
  0s                                                          10s
```

[[Lambda ライフサイクル|コールドスタート]]時だけ [[Lambda ライフサイクル|Init]] が実行されるので、ウォーム時は問題なく動くのに[[Lambda ライフサイクル|コールドスタート]]時だけタイムアウトする——という間欠的な障害が起きうる。[[再現性]]が低く、原因特定に時間がかかりやすい。CloudWatch Logs の `Init Duration` メトリクスで実際の [[Lambda ライフサイクル|Init]] 時間を確認できる。

### Provisioned Concurrency の場合

[[Lambda ライフサイクル|Init]] と [[Lambda ライフサイクル|Invoke]] が**[[ITと不完全さに惹かれる理由|完全]]に別枠**になる。[[Lambda ライフサイクル|Init]] だけで最大15分使えて、その後の [[Lambda ライフサイクル|Invoke]] は関数タイムアウトの全時間を使える。

```
通常 Lambda:
  [========= Init + Invoke で合計タイムアウト =========]

Provisioned Concurrency:
  [==== Init (最大15分) ====][==== Invoke (フルにタイムアウト) ====]
                             ↑ 完全に別枠
```

---

## [[Lambda ライフサイクル|Invoke]] フェーズ：毎回の実行

トリガーから[[基礎 Part 15 イベントオブジェクト|イベント]]が来て、handler が呼ばれ、レスポンスを返す。[[Lambda ライフサイクル|Invoke]] ごとに `context` [[基礎 Part 15 オブジェクト|オブジェクト]]が新しく渡される。

```python
def handler(event, context):
    print(context.aws_request_id)                   # リクエストごとにユニーク
    print(context.get_remaining_time_in_millis())    # 残り実行時間
    print(context.memory_limit_in_mb)                # 割り当てメモリ
    print(context.function_name)                     # 関数名
```

`context.get_remaining_time_in_millis()` を使えば、タイムアウトが迫ったときに処理を打ち切ってグレースフルに終了する制御ができる。

### [[Lambda ライフサイクル|Invoke]] のタイムアウト

最大 **15分（900秒）**。設定は関数ごとに1秒単位で指定できる。

注意点として、**API Gateway 経由だとさらに短くなる**。API Gateway のタイムアウトは最大29秒。[[Lambda]] を15分に設定していても、API Gateway が29秒で切る。

### なぜタイムアウト時間がユーザー設定なのか

[[Lambda]] は汎用的な実行基盤で、中で何が動くかは[[ITと不完全さに惹かれる理由|完全]]にユーザー次第。DynamoDB から1件取得するだけなら100ms、Bedrock で文章生成なら5〜30秒、S3 の大量ファイル処理なら数分。AWS にはコードの中身を解析して所要時間を判断する手段がない。

タイムアウトの本質的な役割は**暴走防止**。「正常時に何秒で終わるか」ではなく「これ以上かかったら異常だから殺してくれ」という安全装置。この判断はビジネスロジックを知っているユーザーにしかできない。

15分という上限は、[[Lambda]] の「短命な関数実行」という設計思想の線引き。これを超える処理は EC2、ECS、Step Functions の領域。

---

## [[Lambda ライフサイクル|Invoke]] 間の待機状態 — [[Lambda ライフサイクル|ウォームスタート]]の正体

[[Lambda ライフサイクル|Invoke]] が終わった後、microVM はすぐには消えない。次のリクエストを待つ**待機状態**になる。

この待機中：

- メモリ上の[[基礎 Part 15 オブジェクト|オブジェクト]]は全部生きている（[[Lambda ライフサイクル|Init]] で作ったクライアントも）
- `/tmp` の中身もそのまま
- **課金はされていない**（[[Lambda ライフサイクル|Invoke]] の実行時間だけが課金対象）

待機中の microVM に次のリクエストが来れば [[Lambda ライフサイクル|Init]] をスキップして [[Lambda ライフサイクル|Invoke]] だけ実行される。これが**[[Lambda ライフサイクル|ウォームスタート]]**。待機中に AWS が microVM を回収した後にリクエストが来たら、新しい microVM をゼロから起動する。これが**[[Lambda ライフサイクル|コールドスタート]]**。

どのくらい維持されるかは AWS が内部的に決めていて非公開。一般的には数分〜数十分と言われているが、負荷状況やリソースの空き具合で変わる。[[Lambda ライフサイクル|コールドスタート]]か[[Lambda ライフサイクル|ウォームスタート]]かは、前回の [[Lambda ライフサイクル|Invoke]] から次の [[Lambda ライフサイクル|Invoke]] までの間隔が AWS の回収判断より短いか長いかで決まり、ユーザーがコントロールできるものではない。

[[Lambda ライフサイクル|コールドスタート]]を[[ITと不完全さに惹かれる理由|完全]]にゼロにしたいなら **Provisioned Concurrency** で microVM を事前起動しておくしかない。「サーバーを消す」思想で始まった [[Lambda]] が、パフォーマンスのために「サーバーを常駐させる」オプションを用意しているのは、思想的には皮肉だが実務的には重要な選択肢。

---

## 同時リクエスト時のライフサイクル

[[Lambda]] は **1つの microVM が同時に処理するリクエストは常に1つだけ**。microVM-A が [[Lambda ライフサイクル|Invoke]] 中に別のリクエストが来たら、新しい microVM-B が起動して独立に [[Lambda ライフサイクル|Init]] → [[Lambda ライフサイクル|Invoke]] が走る。

```
リクエスト①  ─→ [microVM-A] Init → Invoke → 待機...
リクエスト②  ─→ [microVM-B] Init → Invoke → 待機...   ← 同時なので別 microVM
リクエスト③  ─→ [microVM-A]         Invoke → 待機...   ← Aが空いたので再利用
```

つまり**同時実行数 = 起動中の microVM 数**。デフォルト上限はリージョンあたり1,000。

各 microVM は[[ITと不完全さに惹かれる理由|完全]]に独立しているので、microVM-A で [[Lambda ライフサイクル|Init]] 済みのクライアントや `/tmp` のキャッシュは microVM-B には共有されない。スパイクで100リクエストが同時に来たら、100個の microVM が個別に[[Lambda ライフサイクル|コールドスタート]]する可能性がある（[[Lambda ライフサイクル|コールドスタート]]ストーム）。

`/tmp` も microVM ごとに独立しているため、一貫性はない。

```
リクエスト① → microVM-A → /tmp にキャッシュ書く
リクエスト② → microVM-B → /tmp は空（別の箱だから）
リクエスト③ → microVM-A → /tmp にキャッシュある（たまたま同じ箱）
リクエスト④ → microVM-C → /tmp は空（また別の箱）
```

全リクエストで一貫したデータが必要なら、DynamoDB や ElastiCache のような外部の共有ストアに置くしかない。

---

## [[Lambda ライフサイクル|Shutdown]] フェーズ：静かな死

AWS が microVM を回収するとき、最大2秒間の [[Lambda ライフサイクル|Shutdown]] フェーズが与えられる。[[Lambda]] Extensions にシグナルが送られ、ログのフラッシュやメトリクス送信などの後処理ができる。

ただし：

- **いつ [[Lambda ライフサイクル|Shutdown]] が起きるかは予測できない**（AWS の内部判断）
- handler コードに [[Lambda ライフサイクル|Shutdown]] フックを書く仕組みはない（Extensions だけの世界）
- だから**関数コードは常に「いつ消えてもいい」前提で書く必要がある**

これが [[Lambda]] の「ステートレスであるべき」という設計思想と直結している。

---

## エラー時のライフサイクル

### [[Lambda ライフサイクル|Init]] で例外が発生した場合

[[Lambda]] はその microVM を破棄し、**新しい microVM で [[Lambda ライフサイクル|Init]] からやり直す**。[[Lambda ライフサイクル|Init]] のエラーは自動リトライされる。繰り返し失敗すると呼び出し元にエラーが返る。

### [[Lambda ライフサイクル|Invoke]]（handler）で例外が発生した場合

handler が例外を投げても **microVM は破棄されない**。次のリクエストでは同じ microVM が再利用される（[[Lambda ライフサイクル|ウォームスタート]]）。

ここが落とし穴で、handler 内で[[基礎 Part 34 グローバルCSS|グローバル]][[AI激動期における「定数」思考|変数]]を中途半端に書き換えた状態で例外が起きると、**次の [[Lambda ライフサイクル|Invoke]] が汚染された状態で始まる**。

```python
results = []  # グローバル

def handler(event, context):
    results.append(process(event))  # ← 毎回追加される

    if some_condition:
        raise Exception("失敗")
    # ↑ ここで死んでも results はリセットされない
    # 次の Invoke では前回の結果が残ったまま

    return results
```

```
Invoke① → results = [結果A]               → 例外で死亡
Invoke② → results = [結果A, 結果B]         → 前回のゴミが残ってる ⚠️
Invoke③ → results = [結果A, 結果B, 結果C]  → どんどん溜まる
```

これが「ステートレスに書け」と言われる実践的な理由の一つ。[[Lambda ライフサイクル|Init]] には boto3 クライアントのような**不変の[[基礎 Part 15 オブジェクト|オブジェクト]]**を置き、handler 内で**可変な状態を[[基礎 Part 34 グローバルCSS|グローバル]]に持たない**のが鉄則。

---

## `/tmp` とライフサイクルの関係

`/tmp` は microVM に紐づいている。

- [[Lambda ライフサイクル|Init]] で書いたものは [[Lambda ライフサイクル|Invoke]] で読める（当然）
- **前回の [[Lambda ライフサイクル|Invoke]] で書いたものが次の [[Lambda ライフサイクル|Invoke]] でも残っている**（[[Lambda ライフサイクル|ウォームスタート]]時）
- [[Lambda ライフサイクル|Shutdown]] で microVM が消えたら `/tmp` も消える

これを活用して、外部から取得したデータを `/tmp` にキャッシュするパターンがある。

```python
import json, os

CACHE_PATH = '/tmp/config_cache.json'

def handler(event, context):
    if os.path.exists(CACHE_PATH):
        # ウォーム時：キャッシュから読む（高速）
        with open(CACHE_PATH) as f:
            config = json.load(f)
    else:
        # コールド時：外部から取得してキャッシュ
        config = fetch_from_parameter_store()
        with open(CACHE_PATH, 'w') as f:
            json.dump(config, f)
```

鉄則：「あったら使う、なければ取り直す」。microVM がいつ回収されるか保証されないので、永続ストレージとしては使えない。

---

## [[Lambda ライフサイクル|コールドスタート]]の所要時間目安

[[Lambda ライフサイクル|コールドスタート]]は [[Lambda ライフサイクル|Init]] 全体（Extension [[Lambda ライフサイクル|Init]] + Runtime [[Lambda ライフサイクル|Init]] + Function [[Lambda ライフサイクル|Init]]）+ microVM 起動 + zip 展開の合計。構成によって大きく変わる。

- **最小構成（Python, 小さいコード）**：100〜300ms 程度
- **VPC あり**：Hyperplane 導入後は数百ms 追加程度（以前は +10秒）
- **重い依存（ML ライブラリなど）**：数秒〜
- **Java / .NET**：ランタイム自体が重いので1〜5秒
