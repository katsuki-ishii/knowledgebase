---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-15T02:28:00
aliases: [v-html, vhtml, v html]
---
# 1. [[基礎 Part 17 ディレクティブ|ディレクティブ]]とは

Vue.jsの[[基礎 Part 17 ディレクティブ|ディレクティブ]]は、HTMLを拡張するための特別な属性。`v-`で始まる形で要素に記述し、Vueの[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]なデータを使って動的な挙動を実現する。

例：

```html
<p v-if="isVisible">表示される</p>

```

上記の例では、`isVisible`がtrueのときのみ要素が描画される。

---

## 2. 主な組み込み[[基礎 Part 17 ディレクティブ|ディレクティブ]]

| [[基礎 Part 17 ディレクティブ|ディレクティブ]] | 機能 | 例 |
| --- | --- | --- |
| `v-bind` | 属性値をデータにバインド | `<img :src="imageUrl" />` |
| `v-on` | [[基礎 Part 15 イベントオブジェクト|イベント]]を監視 | `<button @click="handleClick">クリック</button>` |
| `v-if` / `v-else` / `v-show` | 条件で表示切り替え | `<div v-if="isActive">有効</div>` |
| `v-for` | 配列やオブジェクトをループ表示 | `<li v-for="item in items">{{ item }}</li>` |
| `v-model` | フォームとの[[基礎 Part 18 v-model|双方向バインディング]] | `<input v-model="username" />` |
| `v-html` | 生のHTMLを挿入 | `<div v-html="htmlContent"></div>` |

---

## 3. [[基礎 Part 7 v-html|v-html]]の概要

`v-html`は、指定した文字列をHTMLとしてそのままDOMに挿入する[[基礎 Part 17 ディレクティブ|ディレクティブ]]。通常の[[基礎 Part 6 マスタッシュ構文|Mustache]]構文（`{{ ... }}`）はHTMLタグをエスケープして出力するが、`v-html`はタグを解釈して描画する。

例：

```html
<div id="app">
  <p>通常出力: {{ rawHtml }}</p>
  <p>v-html出力: <span v-html="rawHtml"></span></p>
</div>

```

```javascript
export default {
  data() {
    return {
      rawHtml: "<strong>太字テキスト</strong>"
    }
  }
}

```

結果：

- 通常出力 → `<strong>太字テキスト</strong>` と文字列として表示
- [[基礎 Part 7 v-html|v-html]]出力 → 太字の「太字テキスト」として表示

---

## 4. [[基礎 Part 7 v-html|v-html]]の注意点

### [[基礎 Part 11 XSSとsanitize|XSS]]（クロスサイトスクリプティング）リスク

`v-html`は渡されたHTMLをそのまま挿入するため、ユーザー入力や外部データを直接扱うと危険。悪意あるスクリプトが実行される可能性がある。

例：

```javascript
rawHtml = '<img src=x onerror="alert(\'攻撃\')">'

```

→ ページに挿入されるとスクリプトが実行される。

対策：

安全なHTMLのみ使用するか、[[基礎 Part 11 XSSとsanitize|サニタイズ]]処理を行う。

例：[[基礎 Part 11 XSSとsanitize|DOMPurify]]を使用

```javascript
import DOMPurify from 'dompurify'
const safeHtml = DOMPurify.sanitize(userInputHtml)

```

### Vue構文が無効

`v-html`で挿入されたHTML内では、`{{ }}`や`v-on`などのVueテンプレート構文は動作しない。`v-html`は静的なHTML挿入に限定される。

### スコープ付き[[ULCOD CSS|CSS]]が適用されない場合がある

スコープ付き[[ULCOD CSS|CSS]]はVueがテンプレート内のタグにのみ作用するため、`v-html`で追加された要素には適用されない可能性がある。

---

## 5. 使用が適切なケース

### 使用してよいケース

- 信頼できるソース（Markdown変換結果や管理者入力）のHTML表示
- CMSやブログ記事のリッチテキスト表示

### 避けるべきケース

- ユーザー入力を直接HTML化する場合
- Vueの[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]バインディングを必要とする場合

---

## 6. 安全な利用例（Markdownレンダリング）

```plain text
<template>
  <div v-html="safeHtml"></div>
</template>

<script setup>
import { marked } from 'marked'
import DOMPurify from 'dompurify'
import { ref, computed } from 'vue'

const markdownText = ref('# タイトル\n\nこれは **Markdown** の例')

const safeHtml = computed(() =>
  DOMPurify.sanitize(marked.parse(markdownText.value))
)
</script>

```

---

## 7. まとめ

| 項目 | 内容 |
| --- | --- |
| 構文 | `<div v-html="someHtml"></div>` |
| 機能 | 文字列をHTMLとして挿入 |
| セキュリティ | [[基礎 Part 11 XSSとsanitize|XSS]]のリスクが高い |
| Vue構文の有効性 | 無効（`{{}}`や`v-on`は動作しない） |
| スタイル適用 | [[基礎 Part 33 style scoped|scoped CSS]]が効かない場合あり |
| 対策 | [[基礎 Part 11 XSSとsanitize|DOMPurify]]等で[[基礎 Part 11 XSSとsanitize|サニタイズ]] |

`v-html`は強力だが安全に扱う必要がある。信頼できるHTMLのみを挿入し、ユーザー入力には必ず[[基礎 Part 11 XSSとsanitize|サニタイズ]]処理を行う。
