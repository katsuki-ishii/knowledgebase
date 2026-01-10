---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - CS
作成日時: 2026-01-08T00:12:00
---
# SessionStorage 基礎まとめ

## 1. SessionStorageとは何か

SessionStorageとは、**ブラウザが提供する一時的なデータ保存領域**である。

JavaScriptからAPI経由で読み書きでき、**同じタブが開いている間だけ有効**という特徴を持つ。

重要なのは、これは変数でもサーバーでもなく、**ブラウザ内部で管理されるストレージ**だという点である。

---

## 2. SessionStorageの保存場所（実体）

### 結論

- 保存の実体は **ブラウザ内部** にある
- Chromeの場合、**主にメモリ上で管理**される
- 仕様上、保存場所（メモリかディスクか）は規定されていない

開発者は保存場所を意識せず、APIの挙動だけを信頼すればよい。

---

## 3. タブとSessionStorageの関係（図解）

```plain text
[PC]
 └─ [Chromeブラウザ]
     ├─ タブA
     │   └─ SessionStorage（A専用）
     │        ├─ step: "2"
     │        └─ name: "Tanaka"
     │
     └─ タブB
         └─ SessionStorage（B専用・別物）
              └─ （空）

```

- 同じURLでもタブが違えばSessionStorageは共有されない
- タブを閉じた瞬間に、そのタブのSessionStorageは破棄される

---

## 4. なぜ「タブを閉じると消える」のか

SessionStorageは**タブのライフサイクルに紐づいている**ためである。

- ページ更新：消えない
- ページ遷移：消えない
- タブを閉じる：消える

これは「一時的な状態管理」に特化した設計である。

---

## 5. DevTools（Applicationタブ）は何をしているのか

Chromeの開発者ツールの Application タブは、

**ブラウザが内部で管理しているSessionStorageの内容を可視化しているビュー**である。

```plain text
JavaScript
   ↓
sessionStorage API
   ↓
Chrome内部ストレージ
   ↓
DevTools Applicationタブ（表示）

```

- ファイルを直接見ているわけではない
- メモリの生データを直接覗いているわけでもない
- Chromeが提供する内部情報をデバッグ用に表示している

---

## 6. 基本的な使い方（具体例）

### データを保存

```javascript
sessionStorage.setItem("step", "2");

```

### データを取得

```javascript
const step = sessionStorage.getItem("step");

```

### データを削除

```javascript
sessionStorage.removeItem("step");

```

---

## 7. 実務での具体的な利用例

### 例1：会員登録のステップ管理

```plain text
Step1 → Step2 → Step3

```

```javascript
sessionStorage.setItem("registerStep", "2");

```

- ページをまたいでも進捗を保持できる
- タブを閉じれば最初からやり直しになる

---

### 例2：画面遷移をまたぐ一時データ

- ログイン前ページURLを一時保存
- ログイン後に元のページへ戻す

```javascript
sessionStorage.setItem("redirectTo", "/mypage");

```

---

## 8. LocalStorageとの違い（整理）

| 項目 | SessionStorage | LocalStorage |
| --- | --- | --- |
| 管理単位 | タブ | ブラウザ＋ドメイン |
| 保存期間 | タブが開いている間 | 明示的に削除するまで |
| 主な用途 | 一時的な状態管理 | 永続的な設定保存 |

---

## 9. 設計上の鉄則

- SessionStorageには「消えても問題ない情報」だけを入れる
- 業務データや重要情報は入れない
- state管理（Vue/Piniaなど）の補助として使う

---

## 10. 一文まとめ

SessionStorageとは、**ブラウザがタブ単位で管理する一時的な内部ストレージであり、Chromeでは主にメモリ上に存在し、その内容はDevToolsのApplicationタブから可視化できる**。

## 関連
- [[Cookie]]
- [[OAuth]]
