---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - CS
作成日時: 2025-08-11T23:44:00
aliases: [URI・URL・URN, URI, URL, URN, uri, url, urn]
---
## 1. [[URI・URL・URN|URI]]とは

- **動きや役割**：Web上のあらゆるリソース（Webページ、画像、動画、APIなど）を統一的な形式で一意に識別するための[[HTMLのclass, id, style|ID]]。場所を使う場合もあれば、名前だけで特定する場合もある。
- **用語名**：[[URI・URL・URN|URI]]（Uniform Resource Identifier）
- **背景**：[[RFC]] 3986で仕様が定義されており、[[URI・URL・URN|URL]]や[[URI・URL・URN|URN]]を含む上位概念。
- **主な構成要素（役割／ポイント／例）**：
    1. **scheme（スキーム）**
        - *役割*：通信方法・解釈ルールの指定（例：`http`, `https`, `ftp`, `mailto`, `data`）
        - *文字*：先頭は英字、以降は英数と `+`  `.` が利用可（慣例的に**小文字**推奨）
        - *例*：`https:`、`mailto:`、`urn:`
    2. **authority（オーソリティ）**（任意）
        - *役割*：接続先の権威的情報（「誰に繋ぐか」）
        - *構成*：`[userinfo@]host[:port]`
            - **userinfo**：`user:pass@` の形。現代は**非推奨**（ログに残る等）
            - **host**：ホスト名 or IP。IPv6は角括弧で囲む（例：`[2001:db8::1]`）
            - **port**：省略時はデフォルト（HTTP=80, HTTPS=443 など）
        - *例*：`user:pass@example.com:8443`
    3. **path（パス）**
        - *役割*：サーバー内でのリソースの階層位置（`/` 区切り）
        - *種類*：`/` から始まる *absolute*、空でもよい *abempty* など
        - *規則*：`.` `..` の**ドットセグメント**は正規化で除去／折りたたみ
        - *注意*：パスの**大文字小文字**はサーバー実装に依存（UNIX系は区別が一般的）
        - *例*：`/articles/2025/08/11`、`/search/%E6%97%A5%E6%9C%AC`
    4. **query（クエリ）**（任意）
        - *役割*：追加パラメータ（検索条件やフィルタ）
        - *形式*：`key=value` を `&`（または `;`）で連結。順序に意味がある場合あり
        - *エンコード*：空白は `%20`（`application/x-www-form-urlencoded` では `+`）
        - *注意*：**機密情報は置かない**（[[URI・URL・URN|URL]]はログや履歴に残る）
        - *例*：`?q=URI&lang=ja`
    5. **fragment（フラグメント）**（任意）
        - *役割*：**取得後**のリソース内の位置指定（サーバーには送信されない）
        - *例*：`#section3`、`#page=5`

---

## 2. [[URI・URL・URN|URL]]（Uniform Resource Locator）

- **動きや役割**：リソースの「場所」と「アクセス方法」を明示する。スキーム（通信方法）・ホスト名（サーバーの住所）・パス（ファイルやページの位置）を指定する。
- **何を受け取り → 何をして → どこに渡すか**：
    1. 受け取る → スキーム、ホスト、パス、必要に応じてポート番号やクエリ
    2. 実行する → 指定プロトコルでサーバーへアクセス
    3. 渡す場所 → サーバー内の特定リソース
- **具体例**：
    - `https://example.com/index.html`（HTTPSで`example.com`のトップ階層のファイル）
    - `ftp://ftp.example.com/file.txt`（FTPで`file.txt`を取得）

---

## 3. [[URI・URL・URN|URN]]（Uniform Resource Name）

- **動きや役割**：場所情報を含まず、名前空間と識別子だけでリソースを特定する。物理的な場所やアクセス方法は別の解決プロセスで決定される。
- **何を受け取り → 何をして → どこに渡すか**：
    1. 受け取る → 名前空間＋識別子（例：ISBN）
    2. 実行する → 名前解決サービスに問い合わせ
    3. 渡す場所 → 実体のある場所を返す仕組み
- **具体例**：
    - `urn:isbn:978-4-123456-47-2`（書籍ISBN）
    - `urn:uuid:123e4567-e89b-12d3-a456-426614174000`（UUID識別）

---

## 4. AWSにおける使われ方

- **[[URI・URL・URN|URL]]型**（場所型[[URI・URL・URN|URI]]）：直接アクセス可能なエンドポイント
    - S3オブジェクト[[URI・URL・URN|URL]]：`https://my-bucket.s3.amazonaws.com/image.png`
    - API Gatewayエンドポイント：`https://abc123.execute-api.ap-northeast-1.amazonaws.com/prod`
- **[[URI・URL・URN|URN]]型**（名前型[[URI・URL・URN|URI]]）：管理や権限設定に利用される一意な名前
    - ARN（Amazon Resource Name）：`arn:aws:s3:::my-bucket/my-object`
    - Lambda関数ARN：`arn:aws:lambda:ap-northeast-1:123456789012:function:MyFunction`

---

## 5. [[URI・URL・URN|URL]]の記号の役割

| 記号 | 動きや役割 | 詳細例 |
| --- | --- | --- |
| `:` | 種類や意味を区切る | `https:`（スキーム）、`:8080`（ポート番号）、`user:pass`（認証情報区切り） |
| `/` | 階層やルートの区切り | `/blog/2025/`（ディレクトリ階層） |
| `#` | 内部位置を指定 | `#section3`（HTML要素やページ内位置） |

---

## 6. フラグメント識別子（`#`）

- **動きや役割**：取得済みリソース内の特定箇所を指す。スクロールやタブ表示切り替えなどに利用され、サーバーには送信されない。
- **例**：`https://example.com/manual.html#section3`（[[HTMLのclass, id, style|id]]="section3"へジャンプ）
- **活用例**：HTMLアンカーリンク、PDFページ指定、SPA（シングルページアプリ）のルーティング

---

## 7. [[URI・URL・URN|URL]]内のユーザー名・パスワード

- **形式**：`https://user:pass@example.com/`
- **歴史的背景**：[[RFC]] 1738など古い仕様では許容され、昔はFTPやHTTP Basic認証の簡易入力に使われていた。しかしセキュリティリスクが高いため、現在はブラウザや仕様面で非推奨・禁止に近い扱い。
- **危険性**：
    - 平文でログや履歴に残る
    - HTTPSで暗号化しなければ盗聴可能
    - モダンブラウザでは警告または[[基礎 Part 9ブロックとブロックスコープ|ブロック]]対象
- **代替案**：
    - HTTPヘッダーの`Authorization`（BasicやBearerトークン）
    - [[OAuth]]やAPIキーを利用

## 関連
- [[REST 概要]]
- [[RFC]]
