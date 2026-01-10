---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-07T18:05:00
---
# Vue.js v-if 

## 1. v-if の基本

v-if は「条件が true のときだけ HTML 要素を DOM に生成する」仕組み。false のときは要素そのものが DOM から消える。

### 例

```html
<p v-if="isLoggedIn">ログイン中</p>

```

---

## 2. 実務での主な使用パターン

### 2-1. ローディング表示の切り替え

API 通信時の読み込み状態を表示する。

```html
<div v-if="loading">読み込み中...</div>
<div v-else>データ表示</div>

```

### 図解

```plain text
loading = true        loading = false
+-----------+         +----------------+
| Loading… |         | Data Content   |
+-----------+         +----------------+

```

---

### 2-2. 認証状態による UI 切り替え

ログインしているかどうかで表示内容を切り替える。

```html
<div v-if="isLoggedIn">
  ようこそ {{ user.name }}
</div>
<div v-else>
  <LoginForm />
</div>

```

### 図解

```plain text
isLoggedIn = true         isLoggedIn = false
+------------------+      +----------------+
| Welcome, User    |      | Login Form    |
+------------------+      +----------------+

```

---

### 2-3. 条件を満たしたときだけボタンを表示

フォームが十分に入力されているときだけ送信ボタンを出す。

```html
<button v-if="form.name && form.email">送信</button>

```

### 図解

```plain text
入力不足                  入力完了
(ボタンなし)             [送信ボタン表示]

```

---

### 2-4. モーダルの表示制御

必要なときだけコンポーネントを DOM に載せる。

```html
<Modal v-if="isOpen" @close="isOpen = false" />

```

モーダルを閉じたとき、内部状態が破棄されるため初期化処理が楽になる。

### 図解

```plain text
isOpen = true         isOpen = false
+-------------+       (DOM から削除)
| Modal DOM   |
+-------------+

```

---

### 2-5. データの有無による表示切り替え

一覧ページなどでよく使う。

```html
<p v-if="items.length === 0">データなし</p>
<ul v-else>
  <li v-for="item in items" :key="item.id">{{ item.name }}</li>
</ul>

```

### 図解

```plain text
items = []                   items = [a,b,c]
[ データなし ]              • a
                            • b
                            • c

```

---

### 2-6. 権限による表示切り替え

管理者だけ操作できる UI を作る。

```html
<button v-if="user.role === 'admin'">削除</button>

```

### 図解

```plain text
role = admin                role = user
[ 削除ボタン ]             (ボタンなし)

```

---

## 2-7. 複数の p タグをまとめて条件で制御する場合

複数の要素をまとめて v-if の対象にしたい場合、ラッパーとして **div** または **template** を使う。どちらを使うべきかは、UI の構造や DOM の意図によって変わる。

---

### どちらを使うべきか（理由つき）

## div を使うべきケース

- **レイアウト上、囲むブロックが必要なとき**
- CSS のスタイル適用のために親要素が欲しい場合
- 論理的にも見た目的にも「ひとまとまり」として扱う意味があるとき

### 例

```html
<div v-if="showTexts" class="message-group">
  <p>メッセージ1</p>
  <p>メッセージ2</p>
  <p>メッセージ3</p>
</div>

```

### 理由

- DOM に div が残るため、スタイル指定が容易
- まとまりとしてのコンテナが必要な UI では div のほうが自然

### 図解

```plain text
DOM:
<div class="message-group">
  <p>1</p>
  <p>2</p>
  <p>3</p>
</div>

```

---

## template を使うべきケース

- **余計な親タグを増やしたくないとき**
- レイアウト上コンテナ要素が不要なとき
- 既に親要素が存在し、その中だけ切り替えたいとき

### 例

```html
<template v-if="showTexts">
  <p>メッセージ1</p>
  <p>メッセージ2</p>
  <p>メッセージ3</p>
</template>

```

### 理由

- template は DOM に出現しない（中身だけ展開される）
- 不要な div が増えないため、HTML がシンプルになる
- CSS の影響範囲を広げたくない場合に有効

### 図解

```plain text
DOM（template は存在しない）:
<p>1</p>
<p>2</p>
<p>3</p>

```

---

## 結論（判断基準）

- 見た目や意味的に“ひとまとまりのブロック”が必要 → **div**
- DOM を汚したくない、親要素がすでにある → **template**

この判断基準を持っておくと UI 設計の時に迷わなくなる。

複数の要素をまとめて v-if の対象にしたい場合、ラッパー要素で囲む。Vue では 1 つの v-if は 1 つのタグにしか付与できないため、グループ化が必要になる。

### 例

```html
<div v-if="showTexts">
  <p>メッセージ1</p>
  <p>メッセージ2</p>
  <p>メッセージ3</p>
</div>

```

### 図解

```plain text
showTexts = true
+-------------------------------+
| <div>                         |
|   <p>メッセージ1</p>         |
|   <p>メッセージ2</p>         |
|   <p>メッセージ3</p>         |
| </div>                        |
+-------------------------------+

showTexts = false
(DOM から <div> ごと削除される)

```

### ポイント

- 複数タグに条件を適用したい場合は **ラッパー div** や `<template>` を使う
- `<template v-if="...">` を使うと余計な div を生まない

### template を使った例

```html
<template v-if="showTexts">
  <p>メッセージ1</p>
  <p>メッセージ2</p>
  <p>メッセージ3</p>
</template>

```

※ template 自体は DOM に出現せず、中の p タグが直接追加・削除される。

---

## 3. 実務的な感覚

- 必要なときだけ DOM に存在させる
- 状態の変化に合わせて Vue が自動で UI を書き換える
- 読み込み中、エラー中、成功時など複数の状態に対応するときに有効

以上が実務での v-if の代表的な使い方になる。不足しているケースや、特定システムでの応用例が必要なら追加してまとめる。

## 関連
- [[基礎 Part 25 v-show]]
- [[基礎 Part 26 v-for 配列]]
