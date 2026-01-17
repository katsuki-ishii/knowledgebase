---
base: "[[CS.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - CS
作成日時: 2025-11-24T20:17:00
aliases: [OAuth, oauth, OAuth2, OAuth 2.0]
---
## 1. そもそも [[OAuth]] とは何か

[[OAuth]] は、一言でいうと「パスワードを渡さずに、サービスの権限だけを安全に貸し出す仕組み」だと理解した。

ポイントは次の通り。

- 本質は認可（Authorization）であり、「誰が」「何をしてよいか」をコントロールする仕組み
- パスワードそのものではなく、「権限付きチケット（アクセストークン）」をやり取りする
- ログインにもよく使われるが、元々は「他サービスの機能を安全に使わせる」ための仕組み

参照：[一番分かりやすい OAuth の説明 - Qiita](https://qiita.com/TakahikoKawasaki/items/e37caf50776e00e733be)

## 2. [[OAuth]] がない世界線で起きていたこと

疑問：「[[OAuth]] がなかった時代はどうしていたのか」

昔は、ユーザーが外部アプリに ID/パスワードを直接渡していた。

例：

- 外部の Twitter クライアントに Twitter の ID/パスワードを入力していた
- そのアプリがユーザーの代わりに Twitter へログインして操作していた

この方式の問題点：

- アプリがパスワードを盗める
- パスワードを渡したら「全権限」を明け渡すことになる（部分的な権限付与ができない）
- 「どのアプリにパスワードを渡したか」をユーザーが管理できない
- 怪しいアプリに一度でも渡すと、パスワード変更以外に対処法がない

この「パスワードを第三者アプリに渡す」やり方が危険すぎたため、それを根本から解決するために [[OAuth]] が生まれた。

## 3. [[OAuth]] が解決すること

[[OAuth]] が導入されることで次のことが可能になった。

- アプリはユーザーのパスワードを一切扱わない
- ユーザーは信頼できる認可サーバー（Google、[[Cognito Authorizer|Cognito]] など）だけにパスワードを入力すればよい
- アプリには「期限付き・権限付きのトークン」だけを渡す
- トークンはいつでも無効化できる（リボーク）
- 「このアプリにはメールアドレスの閲覧だけ許可」など、権限を細かく制御できる

## 4. [[OAuth]] の登場人物

学習の中で整理した役割は次の通り。

- ユーザー: 本人
- クライアントアプリ: Memento のような SPA
- 認可サーバー: [[Cognito Authorizer|Cognito]]（Hosted UI やトークン発行を行う）
- リソースサーバー: API Gateway + Lambda など、守られたデータを持つ側

## 5. [[Cognito Authorizer|Cognito]] の場合の位置づけ

疑問：「[[Cognito Authorizer|Cognito]] の場合はどうなっているのか」

[[Cognito Authorizer|Cognito]] は [[OAuth]] / OIDC の世界で主に次を担当する。

- 認可サーバー: Hosted UI でログイン画面を提供し、ユーザー認証を行う
- [[HTMLのclass, id, style|ID]] プロバイダ: ログイン成功後に [[HTMLのclass, id, style|ID]] トークン、アクセストークンなどの JWT を発行する

クライアントアプリ（SPA）は次だけ行う。

- [[Cognito Authorizer|Cognito]] の Hosted UI にユーザーをリダイレクトする
- 認可コード（code）を受け取る
- code を使ってトークンエンドポイントからトークン群を取得する
- アクセストークンを API に渡してデータにアクセスする

## 6. なぜトークンを直接返さず「認可コード」を返すのか

疑問：「最初からトークンを返せばいいのでは。二度手間ではないか」

これが二段階になっている理由は安全性のため。

- [[URI・URL・URN|URL]] は履歴、リファラ、ログ、ブラウザ拡張など多くの経路で漏洩しやすい
- トークンそのものを [[URI・URL・URN|URL]] に載せると、漏れた瞬間になりすましが可能になってしまう

そのため、認可サーバーは次のように振る舞う。

1. 危険な領域（[[URI・URL・URN|URL]]）では、本命のトークンではなく「短命の引換券（code）」だけを渡す
2. クライアントアプリは、その code を使って、HTTPS の POST リクエストで安全にトークンと交換する

こうすることで「[[URI・URL・URN|URL]]で漏れても致命的ではない情報（code）」と「決して [[URI・URL・URN|URL]] に載せない本命（token）」を分離している。

## 7. 「ログイン成功後、なぜ SPA に戻ってくるのか」の詳細

疑問：「[[Cognito Authorizer|Cognito]] のフロントでログインしたあと、なぜ SPA に戻るのか。具体的に何が起きているのか」

流れの詳細は次の通り。

3. SPA が `/oauth2/authorize` にリクエストを送る際、`redirect_uri` を指定する
    - 例: `https://memento.dev/auth/callback`
    - 「ログインが終わったらここに戻してほしい」という約束
4. [[Cognito Authorizer|Cognito]] の Hosted UI でユーザーがログインに成功する
5. [[Cognito Authorizer|Cognito]] はブラウザに HTTP 302 レスポンスを返す
    - `Location: https://memento.dev/auth/callback?code=...&state=...`
    - これは「この [[URI・URL・URN|URL]] に移動せよ」という指示
6. ブラウザは HTTP の仕様に従って、`Location` の [[URI・URL・URN|URL]] に自動で GET リクエストを送る
    - 結果として、SPA の `/auth/callback` に遷移する
7. SPA の `/auth/callback` ルートが読み込まれ、JavaScript で `code` と `state` を [[URI・URL・URN|URL]] から取得する

この時点ではトークンはまだなく、「code を持って SPA に戻ってきた」という状態。

## 8. SPA がトークンを取得する具体的なリクエスト

疑問：「code を使って本命のトークンを取りに行くとき、どのようにリクエストするのか」

SPA は、[[Cognito Authorizer|Cognito]] のトークンエンドポイントに対して、次のような HTTP POST を送る。

- メソッド: POST
- パス: `/oauth2/token`
- ヘッダ: `Content-Type: application/x-www-form-urlencoded`
- Body（フォームデータ）:
    - `grant_type=authorization_code`
    - `client_id=...`
    - `code=...`
    - `redirect_uri=...`
    - `code_verifier=...`（PKCE 用）

[[Cognito Authorizer|Cognito]] はこれを検証し、正しければ [[設定ファイル言語|JSON]] で以下を返す。

- `access_token`
- `id_token`
- `refresh_token`
- `expires_in`

SPA はこれらをメモリや [[Session storage|sessionStorage]] に保存し、以後の API 呼び出しに `access_token` を利用する。

## 9. client_id とは何か

疑問：「client_id は誰が持つのか。どこから来た値なのか」

結論として、client_id は「アプリケーションを識別するための [[HTMLのclass, id, style|ID]]」であり、ユーザーごとではなく、アプリケーションごとに [[Cognito Authorizer|Cognito]] が発行する。

- [[Cognito Authorizer|Cognito]] の User Pool で [[基礎 Part 3 App.vue|App]] Client を作成したときに発行される
- 認可サーバー側から見て「どのアプリが認証を要求しているか」を判別するための識別子

## 10. redirect_uri は共通か

疑問：「redirect_uri は、その SPA を使っているユーザーごとに違うのか、それとも共通なのか」

結論として、redirect_uri は「そのアプリケーションで共通」の [[URI・URL・URN|URL]] であり、ユーザー単位では変わらない。

- [[Cognito Authorizer|Cognito]] の設定で、[[基礎 Part 3 App.vue|App]] Client ごとに「有効な redirect_uri」を事前登録する
- 登録されていない redirect_uri にはリダイレクトしない
- これにより、攻撃者が勝手な [[URI・URL・URN|URL]] を指定して code を奪うことを防いでいる

SPA 側から見ると、全ユーザーが同じ `/auth/callback` などに戻ってくるが、そこで受け取る `code` はユーザーごとに異なる。

## 11. 他人の code を使ってトークンを盗めない理由

疑問：「client_id と redirect_uri が同じであれば、他人の code を知っているだけでトークンを取れてしまわないか」

これについては、次の理由で成立しない。

8. PKCE による検証
    - SPA はログイン開始時に `code_verifier`（秘密のランダム文字列）を生成し、[[Cognito Authorizer|Cognito]] にはハッシュ済みの `code_challenge` だけを送る
    - トークン交換時に `code_verifier` を送ると、[[Cognito Authorizer|Cognito]] は `SHA256(code_verifier) == code_challenge` か検証する
    - 攻撃者は `code_verifier` を知らないため、一致せず拒否される
9. 認可コードは短命かつ一度しか使えない
    - 一度トークン交換に使われた code は再利用できない
    - 有効期限も非常に短いため、後から盗んでもほぼ意味がない
10. 認可コードは [[Cognito Authorizer|Cognito]] 内部の特定ユーザーセッションと紐づいている
    - 認可コードは単なる文字列ではなく、「どのユーザーがどのクライアントでログインしたか」という情報と関連付けられている

このため、たとえ他人の code を盗んだとしても、正しい `code_verifier` を持たない第三者はトークンを取得できない。

## 12. パスワードとトークンの分離

重要な理解ポイントとして、次がある。

- ユーザーのパスワードは [[Cognito Authorizer|Cognito]] の Hosted UI でのみ入力される
- クライアントアプリ（SPA）はパスワードを一切扱わない
- アプリが扱うのは認可コードとトークンだけ

これにより、

- パスワード漏洩のリスクをアプリ側から完全に切り離せる
- アプリが持つ情報（トークン）は期限付きであり、権限も限定できる

## 13. 現時点での自分の理解の要点

- [[OAuth]] は「パスワードを渡さずに権限だけ貸し出す」ための仕組み
- 昔はパスワードを外部アプリに渡していて、それが危険だった
- [[Cognito Authorizer|Cognito]] は認可サーバーとして、ログイン画面とトークン発行を担当する
- SPA は code を受け取り、自分からトークンを取りに行く
- トークンは必ず POST で受け取り、[[URI・URL・URN|URL]]には載せない
- redirect_uri はアプリ単位で共通であり、事前登録されたものだけが使われる
- client_id はアプリの識別子であり、[[Cognito Authorizer|Cognito]] が発行する
- 他人の code を知っていても、PKCE や一度きりの性質によりトークンは盗めない
- パスワードは [[Cognito Authorizer|Cognito]] の画面とユーザーの頭の中にのみ存在し、アプリは一切触れない

## 関連
- [[RFC]]
- [[REST 概要]]
