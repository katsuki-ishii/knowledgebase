# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

Obsidian ナレッジベース（個人の学習ノート集）。コードプロジェクトではなく、Markdown ファイルの集合体。ビルド・テスト・リントは存在しない。

## 言語・文体ルール

- **すべての応答は日本語**で行う
- **常体**で記述（敬語は使わない）
- **絵文字は使用しない**
- 必要に応じて図解・具体例を含める
- 明らかな誤変換やタイポは確認せずに修正する

## ノート作成・編集ルール

### フロントマター（必須）
新規 `.md` ファイルには必ず以下の YAML フロントマターを付ける：
```yaml
---
base: "[[ディレクトリ名.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - [カテゴリ名]
作成日時: YYYY-MM-DDTHH:mm:ss
aliases: [エイリアス1, エイリアス2]
---
```

### base プロパティ
ディレクトリ名から3桁数字プレフィックス（`000_`, `010_` 等）を除いた名前を使う。サブフォルダがある場合はドットで繋ぐ。
- `010_CS/04_Network/` → `base: "[[CS.Network.base]]"`
- `015_Web/Browser/` → `base: "[[Web.Browser.base]]"`
- `020_AWS/` → `base: "[[AWS.base]]"`
- `030_JavaScript/` → `base: "[[JavaScript.base]]"`
- `200_Life/Psychology/` → `base: "[[Life.Psychology.base]]"`

### aliases ルール
Automatic linker プラグイン（完全一致）のため、以下を網羅的に設定：
- タイトルそのまま、キーワード分解、英語の大文字小文字バリエーション、スペース有無、技術用語の別表記、日英両方

### マークダウン記法
- Obsidian のリンク記法 `[[リンク名]]` を使用
- セクション区切りには `---` を使用

## ディレクトリ構造

| フォルダ | 内容 |
|---|---|
| `000_Home/` | ホーム |
| `010_CS/` | CS基礎（01〜10のサブフォルダ: Hardware, Theory, OS, Network, Database, Security, Programming_Languages, Algorithm, AI, Software_Engineering） |
| `015_Web/` | Web技術（Browser, HTTP, WebSocket, REST, API） |
| `020_AWS/` | AWS（サービスカテゴリ別サブフォルダ: 00_Overview〜99_Architecture_Patterns） |
| `030_JavaScript/` | JavaScript |
| `040_Vue.js/` | Vue.js |
| `050_Development_Environment/` | 開発環境・ツール |
| `200_Life/` | ライフ系（Psychology, Health, Ideas） |
| `300_Journal/` | 日記・ログ |
| `300_News/` | ニュース |

## Obsidian プラグイン

- **automatic-linker**: エイリアス完全一致で自動リンク生成
- **remotely-save**: リモート同期
- **obsidian-kanban**: カンバンボード
- **obsidian-importer**: インポーター
