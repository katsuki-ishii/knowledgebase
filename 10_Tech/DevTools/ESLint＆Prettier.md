---
base: "[[DevTools.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - 開発ツール
作成日時: 2025-09-22T13:57:00
aliases: [ESLint＆Prettier, ESLint, Prettier, eslint, prettier]
---
### [[ESLint＆Prettier|ESLint]] と [[ESLint＆Prettier|Prettier]] の概要

**[[ESLint＆Prettier|ESLint]]**

- **役割**: コード品質のチェック、潜在的なバグの検出、コーディング規約の強制。
- **詳細**: 未使用の変数、構文エラー、非推奨のパターンなど、プログラムの動作に影響を与える可能性のある問題を特定する。

**[[ESLint＆Prettier|Prettier]]**

- **役割**: コードの自動フォーマット。
- **詳細**: インデント、[[改行コード|改行]]、スペース、引用符の種類など、コードの見た目を統一する。チーム開発において、コードスタイルの議論をなくし、一貫性を保つために使用される。

---

### プロジェクトへの導入方法

### 1. パッケージのインストール

TypeScriptを使用しない場合、以下のコマンドで必要なパッケージをインストールする。

Bash

```bash
npm install eslint prettier eslint-plugin-vue eslint-config-prettier --save-dev

```

### 2. [[設定ファイル言語|設定ファイル]]の作成

プロジェクトのルートディレクトリに、以下の[[設定ファイル言語|設定ファイル]]を作成する。

`**.eslintrc.cjs**`

このファイルは、[[ESLint＆Prettier|ESLint]]の動作を制御する。

- `root: true`: [[ESLint＆Prettier|ESLint]]に、このファイルが設定のルートであることを伝える。これより上のディレクトリにある[[設定ファイル言語|設定ファイル]]は無視される。
- `extends`: 他の[[設定ファイル言語|設定ファイル]]からルールを継承する。
    - `'plugin:vue/vue3-essential'`: Vue 3の基本的な[[ESLint＆Prettier|ESLint]]ルールセットを適用する。
    - `'eslint:recommended'`: [[ESLint＆Prettier|ESLint]]が推奨する基本的なルールを適用する。
    - `'@vue/eslint-config-prettier'`: [[ESLint＆Prettier|Prettier]]と競合する[[ESLint＆Prettier|ESLint]]のスタイル関連ルールを無効にする。
- `parserOptions`: [[ESLint＆Prettier|ESLint]]が構文を解析する方法を設定する。
    - `ecmaVersion: 'latest'`: 最新のECMAScript仕様に対応する。

JavaScript

```javascript
/* eslint-env node */
require('@rushstack/eslint-patch/modern-module-resolution')

module.exports = {
  root: true,
  'extends': [
    'plugin:vue/vue3-essential',
    'eslint:recommended',
    '@vue/eslint-config-prettier'
  ],
  parserOptions: {
    ecmaVersion: 'latest'
  }
}

```

`**.prettierrc.cjs**`

このファイルは、[[ESLint＆Prettier|Prettier]]のフォーマットルールを定義する。

- `singleQuote: true`: 文字列をシングルクォート `'` で記述する。
- `semi: false`: 文末の[[基礎 Part 5セミコロン・パース|セミコロン]] `;` を省略する。

JavaScript

```javascript
module.exports = {
  singleQuote: true,
  semi: false
}

```

---

### 修正コマンド

ターミナルで以下のコマンドを実行することで、コードを自動で修正・整形できる。

**[[ESLint＆Prettier|ESLint]]の自動修正**

Bash

```bash
npx eslint . --fix

```

**[[ESLint＆Prettier|Prettier]]の自動フォーマット**

Bash

```bash
npx prettier --write .

```