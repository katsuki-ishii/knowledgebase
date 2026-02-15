---
base: "[[Web.HTML.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Web
作成日時: 2025-12-07T17:30:00
aliases: [HTMLのclass, id, style, class, id, style, HTML class id style]
---
# [[HTMLのclass, id, style|HTMLのclass]]、[[HTMLのclass, id, style|id]]、[[HTMLのclass, id, style|style]]属性のまとめ

## [[基礎 Part 24 class指定|class]]属性

[[基礎 Part 24 class指定|class]]属性は要素に対してグループ名を付けるために使用する。複数の要素に共通のデザインを適用したり、JavaScriptでまとめて操作したりする際に利用する。

### 特徴

- 同じ[[基礎 Part 24 class指定|class]]は複数の要素に付けることができる。
- 1つの要素に複数の[[基礎 Part 24 class指定|class]]を付けることができる。

### 例

```html
<p class="warning">注意事項</p>
<p class="warning bold">重要な注意事項</p>

```

```css
.warning {
  color: red;
}

.bold {
  font-weight: bold;
}

```

## [[HTMLのclass, id, style|id]]属性

[[HTMLのclass, id, style|id]]属性はページ内で一意の名前を付けるために使用する。1ページに同じ[[HTMLのclass, id, style|id]]を複数使うことは推奨されない。

### 特徴

- ページ内リンクや特定の1要素のみを操作したい場合に利用する。
- [[ULCOD CSS|CSS]]で指定する場合は `#id名` を使う。

### 例

```html
<h1 id="top">トップ</h1>

```

```css
#top {
  color: blue;
}

```

## [[HTMLのclass, id, style|style]]属性

[[HTMLのclass, id, style|style]]属性は、その要素に直接[[ULCOD CSS|CSS]]を記述するために使用する。ただし、保守性が低く、本番コードでは推奨されない。

### 問題点

- [[ULCOD CSS|CSS]]が分散し、可読性が低下する。
- 再利用ができず、複数箇所の修正が必要になる。
- [[ULCOD CSS|CSS]]の詳細度が高く、他のスタイルで上書きしにくい。

### 例

```html
<p style="color: green; font-size: 20px;">強調表示されたテキスト</p>

```

## 推奨される使い方

- 基本的には[[基礎 Part 24 class指定|class]]属性を使ってスタイルを管理する。
- [[HTMLのclass, id, style|id]]属性は特定要素のみを指す必要がある場合に限定して使う。
- [[HTMLのclass, id, style|style]]属性はデバッグなど一時的な用途のみにとどめる。