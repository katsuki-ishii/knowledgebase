---
base: "[[Development_Environment.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - 開発環境
作成日時: 2026-02-09T12:00:00
aliases: [Prettier, prettier, プリティア]
---

# [[Prettier]]

**役割**: コードの自動フォーマット。

インデント、[[改行コード|改行]]、スペース、引用符の種類など、見た目を統一する。チーム開発でコードスタイルの議論を減らし、一貫性を保つために使う。[[ESLint]] は品質・バグ検出、[[Prettier]] は見た目のみを担当する別ツールである。

---

## プロジェクトへの導入

### パッケージのインストール

[[ESLint]] と併用する場合は `eslint-config-prettier` も入れる（[[ESLint]] 側のスタイルルールを無効化するため）。

```bash
npm install prettier eslint-config-prettier --save-dev
```

### [[設定ファイル言語|設定ファイル]]（.prettierrc.cjs）

プロジェクトルートに `.prettierrc.cjs` を置き、[[Prettier]] のフォーマットルールを定義する。

- `singleQuote: true`: 文字列をシングルクォート `'` で書く。
- `semi: false`: 文末の[[基礎 Part 5セミコロン・パース|セミコロン]] `;` を付けない。

```javascript
module.exports = {
  singleQuote: true,
  semi: false
}
```

---

## 自動フォーマットコマンド

```bash
npx prettier --write .
```
