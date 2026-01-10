---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - AWS
URL: https://dev.classmethod.jp/articles/iam-role-passrole-assumerole/
作成日時: 2026-01-10T11:59:00
---
![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/iamrole-omenn.png)

IAM ロールはお面のようなもの IAM ロールはお面のようなもの IAM ロールはお面のようなもの

この記事は公開されてから1年以上経過しています。情報が古い可能性がありますので、ご注意ください。

コンバンハ、千葉（幸）です。

皆さんは、 **PassRole** と **AssumeRole** についてきちんと理解ができていますか？どちらも IAM ロールに関するものですね。

私はカラダ（ボディ）の調子がいい時は思い出せるのですが、雨が降っている日や、ちょっと疲れて気を抜いた時にはすぐ分からなくなってしまいます。

ということで、**イメージとして脳に刻み付けることによって忘れられなくしてやろう**と思いました。

そこで出来上がったのが以下です。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/PassRoleAssumeRole.jpg)

間違えました。以下です。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/PassRoleAssumeRole2.png)

あ、でもやっぱり忘れづらいのはこちらかもしれませんね。

どうですか？もう忘れられなくなりましたね？

## 先にまとめ

- IAM ロールには以下ポリシーを設定できる 
    - アイデンティティベースポリシー
    - Permissions boundary
    - 信頼ポリシー
- AWS リソースに IAM ロールを引き渡す際には PassRole の権限が必要 
    - PassRole は独立したアクションではない
- AsumeRole することで IAM ロールに設定された権限を引き受けることができる 
    - AssumeRole を実行する際には STS を介している
    - AssumeRole を行うのは IAM ユーザーだけではない
    - AssumeRole は重ね掛けできる

## IAM ロールとはお面のようなものである

皆さんご存知かと思いますが、 IAM ロールはお面のようなものです。

間違えました。皆さんご存知ではないと思いますが、私は「 IAM ロールとはお面のようなものである」という風説を流布しようとしています。

AWS のアイコンでは IAM ロールはヘルメットで表現されていますが、日本情緒あふれるお面の方が好きです。あと、被って力を得るならお面の方が気分が出ますよね。

### IAM ロール自身はアクションを実行できない

IAM ロールには権限（**【力】**）を与えることができますが、 IAM ロール自身がその権限を行使することはありません。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAMRole1.png)

IAM ロールことお面を被る人がいて初めて、その力を使えるようになります。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAMRole2.png)

### IAM ロール と 3つのポリシー

「 **ちょっと IAM ロールに設定されたポリシーを確認して**」と言われたら皆さんはどこを確認しますか？

（以前の私なら「【ポリシー】とは具体的に何を指していますか。指示が曖昧なため対応できません。」と答えていましたが、流石にいまはいい大人なのでそんなことはしません。）

確認箇所として**アイデンティティベースポリシー**を想像する方が多いかと思いますが、IAM ロールに「設定」できるポリシーは以下の 3 つがあります。

- アイデンティティベースポリシー
- Permissions boundary
- 信頼ポリシー

3 つも設定できるのは IAM ロールだけです。こんなにも多くのポリシーをもらえる IAM ロールは特別な存在なんだと私は感じました。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAMRole3.png)

アイデンティティベースポリシーと Permissions boundary は以下から確認できます。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAM_Management_Console4.png)

信頼ポリシーはここから確認できます。ちなみに信頼ポリシーは、分類としてはリソースベースポリシーというものに属します。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAM_Management_Console2.png)

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAM_Management_Console3.png)

IAM ロールの力を得る際に一時的に設定できる**セッションポリシー**という概念もあるのですが、今日のところは忘れておきましょう。

### IAM ロールの力を得る

お面を被って IAM ロールの力を得ると、その IAM ロールが持っていた力を引き継ぎます。一方で、元々持っていた力は失われます。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAMRole4.png)

同じ IAM ロールを被ったエンティティは同じ権限を持ちます。エンティティは IAM ユーザーに限らず、 AWS サービス、外部 Idp である場合もあります。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAMRole5.png)

力を行使できるのは人間だけではない……覚えておいてください。

## PassRole とは

PassRole は、 **AWS サービスに IAM ロールをパスする**ための権限を表します。 PassRole という独立したアクションがあるわけではありません。

以下表でも、「アクセス許可のみ」の記述がありますね。

- [Identity and Access Management のアクション、リソース、および条件キー - AWS Identity and Access Management](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/list_identityandaccessmanagement.html)

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/AWS_Identity_and_Access_Management.png)

### PassRole が無いばっかりにインスタンス 1 台すら立ち上げられない

PassRole が必要となる代表的なケースとして、 EC2 インスタンスの作成があります。

多くの場合、EC2 インスタンスには初めからロールを設定した状態で作成を行うでしょう。

ここで、操作ユーザーにAWS 管理ポリシー`AmazonEC2FullAccess`と 参照権限 のみが与えられている状況を考えます。 IAM に関する権限は限られたユーザーにのみ割り当てる、というのはよくあるケースです。

`AmazonEC2FullAccess`には`iam:PassRole`の許可が含まれていません。その状態で IAM ロールをアタッチした EC2 インスタンスを作成しようとすると、最後のステップでエラーが発生します。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/iam_passrole22.png)

`You are not authorized to perform this operation. Encoded authorization failure message:`

以下のページの手順に従いエラーメッセージをデコードすると、メッセージに`"action":"iam:PassRole"`が含まれていることを確認できます。

- [EC2 インスタンスの起動時の UnauthorizedOperation エラーを解決する](https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-not-auth-launch/)

### 他のアクションに付随して PassRole が必要となる

このケースにおいては EC2 インスタンスの作成を行う`ec2:RunInstances`と一緒に `iam:PassRole`というアクションが呼び出されており、それを実行するための権限が必要となります。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAMRole6.png)

新規作成時だけでなく、既存の EC2 インスタンスのロール設定を変更する際にも PassRole は必要です。そして EC2 インスタンス以外にも IAM ロールを受け渡すことができる AWS サービスやそのリソースは多数存在します。

「人が使う IAM ロールを操作する権限は不要だが、 PassRole が必要となる」というケースは多々あります。権限設計の際には PassRole をどうするかを意識しておくとよいでしょう。

PassRole で渡せるロールやサービスを制限することもできます。

ちなみに IAM ロールを被った ユーザーが PassRole することもできます。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAMRole7.png)

## AssumeRole とは

AssumeRole は、**IAM ロールを引き受ける**ためのアクションを表します。ここまで「 IAM ロールのお面を被る」と表現してきたものは、**概ね** AssumeRole のことを表しています。

ちなみに AssumeRole のサービスプレフィックスは`iam`ではなく`sts`です。`iam:AssumeRole`ではなく`sts:AssumeRole`です。

- [AWS Security Token Service のアクション、リソース、および条件キー - AWS Identity and Access Management](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/list_awssecuritytokenservice.html)

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/AWS_Identity_and_Access_Management-8469693.png)

先ほど確認した IAM ロールの信頼ポリシーにおいても、アクションとして`sts:AssumeRole`が許可されています。

```plain text
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

```

### IAM ロールのお面を直接被れるわけでは無い

当然のように「 IAM ロールのお面を被る」と表現してきましたが、実際には IAM ロールを被ることはできません。もう少し実態に近い表現をすると、「 **STS を通じて祈りを捧げ、 IAM ロールと同じ力を一時的に得る**」という挙動となります。

力を得るために媒介を必要とする、よくある話ですね。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/IAMRole8.png)

ユーザーが `sts:AssumeRole`を行うと、STS は両者のポリシーを評価して AssumeRole を行って問題ないかを確認し、問題なければユーザーに以下の3つを与えます。

- アクセスキーID
- シークレットアクセスキー
- セッショントークン

上記により、ユーザーは IAM ロールが持っていた力を一時的に使用できるようになります。

### スイッチロールをする時も AssumeRole をしている

マネジメントコンソール上でロールを切り替えて作業する（スイッチロール）、という操作を行っている方は多いかと思います。その裏側では、AssumeRole が行われています。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/SwitchRole.png)

以下は CloudTrail に記録された SwitchRole イベントのイメージです。

スイッチ元のアカウントが`999999999999`、スイッチ先のアカウントが`000000000000`です。

```plain text
{
    "eventVersion": "1.08",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "xxxxxxxxxH732QEGJXBGU:セッション名（ユーザー名）",
        "arn": "arn:aws:sts::000000000000:assumed-role/スイッチ先ロール名/セッション名（ユーザー名）",
        "accountId": "000000000000"
    },
    "eventTime": "2020-12-20T11:55:50Z",
    "eventSource": "signin.amazonaws.com",
    "eventName": "SwitchRole",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "202.xx.xxx.xx",
    "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36",
    "requestParameters": null,
    "responseElements": {
        "SwitchRole": "Success"
    },
    "additionalEventData": {
        "SwitchFrom": "arn:aws:iam::999999999999:user/ユーザー名",
        "RedirectTo": "https://ap-northeast-1.console.aws.amazon.com/console/home?region=ap-northeast-1#"
    },
    "eventID": "xxxxxxxx-adc1-496d-9252-d59a3da0b7c2",
    "readOnly": false,
    "eventType": "AwsConsoleSignIn",
    "managementEvent": true,
    "eventCategory": "Management",
    "recipientAccountId": "000000000000"
}

```

マネジメントコンソールの操作で明示的に AssumeRole を行うことはありませんが、スイッチした時点で AssumeRole 後のユーザーが記録されていることが分かります。

AssumeRole を行う際は、そのセッションごとに**セッション名**が付与されます。スイッチロールの際は、自動的に「スイッチ元のユーザー名」がセッション名として設定されます。

### STS による仮想体が相手アカウントに作成されている

上記で確認した通り、スイッチ後のユーザーは以下のような ARN で評価されます。

`arn:aws:sts::000000000000:assumed-role/スイッチ先ロール名/セッション名（ユーザー名）`

一方で、スイッチ元のユーザーは以下のような ARN です。

`arn:aws:iam::999999999999:user/ユーザー名`

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/SwitchRole2.png)

スイッチ後は、 ARN のサービス名前空間（`arn:aws:`に続くセクション）が`iam`から`sts`に変わっています。

つまり、スイッチロール（ひいては AssumeRole）を行ったあとは、IAM ユーザーではなく「 STS によりロールを引き受けたセッション」が操作を行う主体となります。

AssumeRole をした後に操作を行うと、アクションを実行した ARN は`arn:aws:sts::000000000000:assumed-role/ロール名/セッション名`として CloudTrail に記録されます。

自分の実体（ボディ）は家にいながら、仮想体が外で活動をしている、という妄想が捗りますね。（もちろんアカウントを跨がない AssumeRole もあります。家の中でも仮想体にはなれます。）

### AssumeRole は重ねがけできる

AssumeRole は重ね掛けできます。しかしそれは、二つのお面の力を同時に得られる、ということではありません。

仮想体の状態で STS に祈りを捧げ、新たな仮想体を生み出すことができる、というものです。これは **IAM ロールの連鎖**と呼ばれます。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/SwitchRole3.png)

ちなみに、スイッチロールの場合は連鎖はできません。マネジメントコンソールで複数のロールにスイッチして作業する場合、ロールからロールへスイッチするわけではなく、裏側では都度 IAM ユーザーからスイッチし直しています。

### AWS サービスも AssumeRole している

「 AWS サービスが IAM ロールの権限を利用している」という話を何度かしていますが、その具体的な動きを見たい場合は CloudTrail から確認すると良いでしょう。「こいつこんなセッション名つけてんのかよ……」とか思ったりしましょう。

イベント名で`AssumeRole`を指定して検索すると、様々なイベントがヒットするかと思います。「発信元 IP アドレス」で AWS サービスが確認できます。

人が AssumeRole した場合にはグローバル IP が記録される箇所ですが、 AWS サービスの場合にはサービス名にとって変わります。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/CloudTrail_Management_Console.png)

夜な夜な色んな AWS サービスがお面を被っている……そんな妄想をすると楽しいですね。

## まとめ

- IAM ロールはお面のようなもの
- PassRole は人間以外にもお面を与えること
- AssumeRole は STS に祈りを捧げて一時的にお面の力を得た仮想体を出現させること

## 終わりに

最後にもう一度冒頭の絵を確認しておきましょう。

これでもう2度と忘れられなくなったかと思います。

当初はここまではっちゃけるつもりはなかったのですが、戯れに深夜テンションで描いた絵がだんだん捨てるには忍びなくなってきました。

> 
> 昨日の夜かいたのを一晩寝かせてから見直したけど、やっぱ深夜はテンションがおかしくなってるんだな [pic.twitter.com/Ov3msUpQcb](https://t.co/Ov3msUpQcb)
> 
> — Batchi (@batchicchi) [December 18, 2020](https://twitter.com/batchicchi/status/1339763160614457344?ref_src=twsrc%5Etfw)

2時間くらい悩んだ結果、このまま突き進むことにしました。

![](https://devio2023-media.developers.io/wp-content/uploads/2020/12/omen_tengu.png)

判断が遅い！

なお、本エントリでは「いらすとや」さんのイラストを多く使用していますが、無償利用の規定に則り、使用する素材は20点以下（重複したものは 1 点とカウント）となるようにしています。

皆さんのこれからの長い人生のなかで、 IAM ロールのことを考える度に天狗のお面が頭にちらつく、そんな風になっていれば幸いです。

いや、本当に本当に。

以上、[千葉（幸）](https://twitter.com/batchicchi)がお送りしました。

## 参考