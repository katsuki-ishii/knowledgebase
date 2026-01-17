---
base: "[[CS.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - CS
作成日時: 2025-08-14T00:22:00
aliases: [Cookie, cookie]
---
# 1. [[Cookie]]とは何か

- **動きや役割**：ウェブサイトがブラウザに渡す「小さなメモ」。
- **目的**：ログイン状態の維持、ショッピングカート情報保持、サイト設定（言語/テーマ）の保存、広告や分析のためのトラッキング。
- **やりとりの流れ**：
    1. サーバーから「Set-Cookie」ヘッダーでブラウザに渡される。
    2. ブラウザはローカルに保存。
    3. 次回アクセス時、条件に合えば自動で送信。

---

## 2. 保存場所（実体）

- ブラウザは[[Cookie]]を**ローカルのデータベース（SQLite形式）**に保存。
- 保存場所例（WindowsのChrome）：
`C:\Users\<ユーザー名>\AppData\Local\Google\Chrome\User Data\Default\Cookies`
- SQLiteは**サーバー不要の小型データベース**で、1つのファイルにデータを格納。

**用語**：

- **SQLite**：ファイル単体で動く軽量データベース。ブラウザやアプリ内蔵の保存に多用される。

**例え**：

- 通常のDB＝立派な図書館
- SQLite＝自分の机の引き出しにある辞書

---

## 3. [[Cookie]]が送られる条件

ブラウザはHTTPリクエスト送信前にローカルDBを参照し、以下をチェック：

1. **Domain**：アクセス先ドメインと一致またはサブドメインか。
2. **Path**：現在の[[URI・URL・URN|URL]]パスと一致または上位階層か。
3. **Secure**：HTTPS通信時のみ送信。
4. **有効期限**：期限内か。
5. **SameSite**：クロスサイト送信可否設定。

---

## 4. [[Cookie]]の送信形式（HTTPリクエスト）

```plain text
GET /mypage HTTP/1.1
Host: example.com
Cookie: sessionid=abc123; theme=dark; lang=ja

```

- `Cookie:` ヘッダーに条件一致した[[Cookie]]を`;`区切りで送信。
- JavaScript不要（HTTPプロトコルの標準機能）。

**用語**：

- **HTTPヘッダー**：ブラウザとサーバーがやり取りする追加情報の部分。[[Cookie]]はこの中で送られる。

---

## 5. [[Cookie]]のライフサイクル

6. **作成（サーバー→ブラウザ）**

```plain text
Set-Cookie: sessionid=abc123; Domain=example.com; Path=/; Secure; HttpOnly; Max-Age=3600

```

7. **保存（ローカルDB）**
    - Domain, Path, Secure, 有効期限などの情報付きで保存。
8. **送信（ブラウザ→サーバー）**

```plain text
Cookie: sessionid=abc123; theme=dark

```

9. **更新・削除**
    - 同名で再発行→更新。
    - 有効期限を過去に設定→削除。

---

## 6. 名前の由来と歴史

- **由来**：1970年代の"Magic [[Cookie]]"（やり取りする小さなデータの塊）から。さらに"Fortune [[Cookie]]"の比喩も。
- **誕生**：1994年、Netscape社のLou Montulliが開発。
- **目的**：Webのステートレス性を補い、ショッピングカートやログイン継続を可能にするため。

---

## 7. 重要ポイントまとめ

- [[Cookie]]は**ブラウザが自動送受信**する状態管理用メモ。
- 保存先は**ローカルのSQLite DB**。
- 送信対象はDomain/Path/Secure/有効期限などで自動判定。
- JavaScript不要でHTTP標準機能として動作。
- 履歴・広告トラッキングにも利用されるためプライバシー配慮が必要。

## 関連
- [[Session storage]]
- [[OAuth]]
