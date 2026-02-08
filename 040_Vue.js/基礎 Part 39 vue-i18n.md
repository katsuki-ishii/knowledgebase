---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2026-01-25T00:00:00
aliases: [vue-i18n, i18n, 国際化, 多言語対応]
---

# [[基礎 Part 39 vue-i18n|i18n]]

## 1. [[基礎 Part 39 vue-i18n|i18n]]とは何か

[[基礎 Part 39 vue-i18n|i18n]]（Internationalization）は、[[基礎 Part 39 vue-i18n|多言語対応]]を**後から安全に・低コストで行うための設計と仕組み**である。
Vueでは `vue-i18n` を使って、表示文言をキーで管理し、言語ごとに差し替える。

重要なのは、[[基礎 Part 39 vue-i18n|vue-i18n]]は**翻訳はしない**という点である。
翻訳された文言を「切り替えて出す」だけを担当する。

---

## 2. 全体構造（図解）

```
[User]
   │
   │ 言語選択（ja / en）
   ▼
[locale]
   │
   │ 参照
   ▼
[messages]
   │
   ├─ ja.json
   │     └─ common.submit = "送信"
   │
   └─ en.json
         └─ common.submit = "Submit"
   │
   ▼
[t('common.submit')]
   │
   ▼
[画面表示]
```

---

## 3. localeの正体

`locale` の実体は **言語コード（文字列）** である。

例：

* 日本語: `ja`
* 英語: `en`
* 地域含む場合: `ja-JP`, `en-US`

[[基礎 Part 39 vue-i18n|vue-i18n]]は、`locale` の値を見て「どの辞書を使うか」を決める。

### 実務例

* ユーザー設定として `preferredLocale = 'ja'` を DynamoDB に保存
* ログイン後に取得して `locale.value = preferredLocale` に反映

---

## 4. 辞書（messages）の実体はどこにあるか

翻訳辞書は**アプリ内で共通管理**するのが主流である。

### 推奨ディレクトリ構成

```
src/
 ├─ i18n.ts
 └─ locales/
     ├─ ja.json
     └─ en.json
```

### [[基礎 Part 39 vue-i18n|i18n]].ts（例）

```ts
import { createI18n } from 'vue-i18n'
import ja from '@/locales/ja.json'
import en from '@/locales/en.json'

export const i18n = createI18n({
  legacy: false,
  locale: 'ja',
  fallbackLocale: 'ja',
  messages: { ja, en }
})
```

辞書はページごとに[[基礎 Part 13 分割代入|分割]]管理してもよいが、
**使用時は必ず [[基礎 Part 39 vue-i18n|i18n]] に集約される**。

---

## 5. 基本的な使い方（具体例）

### 翻訳ファイル

```json
// ja.json
{
  "common": {
    "submit": "送信"
  },
  "login": {
    "title": "ログイン",
    "description": "ログインしてください"
  }
}
```

```json
// en.json
{
  "common": {
    "submit": "Submit"
  },
  "login": {
    "title": "Login",
    "description": "Please log in"
  }
}
```

### [[基礎 Part 30 コンポーネント|コンポーネント]]側

```ts
import { useI18n } from 'vue-i18n'

const { t } = useI18n()
```

```vue
<h1>{{ t('login.title') }}</h1>
<p>{{ t('login.description') }}</p>
<button>{{ t('common.submit') }}</button>
```

---

## 6. キー命名ルール（テンプレート）

### 基本原則

* キーは英語
* 意味ベース
* 文章は1キー
* 単語の組み立てで文章を作らない

### 推奨スコープ

```
common.actions.submit
common.actions.cancel
common.labels.email

login.title
login.description
login.form.submit
login.errors.invalidCredentials

errors.required
errors.invalidEmail

toast.saved
toast.updated
```

---

## 7. 単語と文章の扱い

### 単語

```json
"submit": "送信"
```

### 文章

```json
"pleaseLogin": "ログインしてください"
```

単語か文章かは区別しない。
**UIに表示される文字はすべてキー管理**する。

---

## 8. [[AI激動期における「定数」思考|変数]]を含む文章（例）

```json
// ja.json
"guide": {
  "clickButton": "このボタンを押すと{action}されます"
}
```

```json
// en.json
"guide": {
  "clickButton": "Click this button to {action}."
}
```

```vue
<p>{{ t('guide.clickButton', { action: t('common.submit') }) }}</p>
```

文章構造は言語ごとに変えてよい。

---

## 9. 直書きしてよい例外

以下は直書きでも問題になりにくい。

* デバッグログ
* システム[[HTMLのclass, id, style|ID]]、コード値
* ユーザー入力そのもの
* 規約・長文（CMSやバックエンド管理）

一方、以下は直書き禁止。

* ボタン文言
* 見出し
* 説明文
* エラーメッセージ

---

## 10. チーム運用での注意点

* 命名ルールを最初に決める
* commonを増やしすぎない
* 翻訳ファイルもPRレビュー対象にする
* missing [[基礎 Part 27 v-for 配列|key]] を開発環境で検知する
* 表記ゆれガイドを用意する
* 翻訳文にHTMLを混ぜない

---

## 11. 開発フロー（推奨）

```
画面設計
   ↓
表示要素洗い出し
   ↓
キー一覧作成
   ↓
翻訳ファイル作成（まずは ja）
   ↓
実装（t('key')）
```

---

## 12. まとめ

* localeは言語コード
* 辞書はアプリ内で共通管理
* 翻訳は人が行う
* キー設計が[[基礎 Part 39 vue-i18n|i18n]]の8割を決める
* 運用ルールがないと必ず崩れる

この設計を守れば、Vue3 + [[基礎 Part 39 vue-i18n|vue-i18n]]は安全にスケールする。
