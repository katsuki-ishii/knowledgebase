---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-01T17:24:00
aliases: [XSSとsanitize, XSS, sanitize, サニタイズ, DOMPurify]
---
### 1. [[基礎 Part 7 v-html|v-html]]とは

Vue.jsで`v-html`は、要素の中にHTML文字列をそのまま描画する[[基礎 Part 17 ディレクティブ|ディレクティブ]]である。通常の`{{ }}`による出力はHTMLをエスケープして無害化するが、`v-html`はそのままHTMLとしてブラウザに挿入する。このため、見た目の装飾を柔軟に行える一方で、セキュリティ上のリスクが生じる。

### 2. 危険性：[[基礎 Part 11 XSSとsanitize|XSS]]（クロスサイトスクリプティング）

`v-html`の危険性は、[[基礎 Part 11 XSSとsanitize|XSS]]攻撃（Cross-Site Scripting）にある。ユーザーが入力欄などにHTMLタグやJavaScriptを含むコードを入力し、それが`v-html`でそのまま描画されると、ブラウザがそのスクリプトを実行してしまう。

### 攻撃の例

```html
<script>alert('XSS');</script>

```

このようなスクリプトがユーザー入力として保存・表示されると、他のユーザーがページを閲覧した際に任意のJavaScriptが実行される。これにより、[[Cookie]]情報の窃取、セッション乗っ取り、悪意あるサイトへのリダイレクトなどが発生する可能性がある。

### 3. 攻撃が成立する条件

[[基礎 Part 11 XSSとsanitize|XSS]]は「ユーザーが入力したデータが他人のブラウザに表示される」場合に発生する。自分の入力が自分だけに表示されるケースでは問題は限定的だが、コメント欄、プロフィール、投稿内容など他者に共有される要素で脆弱性が発生する。

### 4. 攻撃者はどう見抜くか

攻撃者はソースコードを直接見る必要がない。以下のような方法で挙動から推測できる。

- HTMLタグを入力して反映されるかを確認する
- `<script>`タグを試して実行されるかを観察する
- ブラウザの開発者ツールでDOM構造を確認する
- 直接HTTPリクエストを送り、サーバーからのレスポンスを解析する

これらの方法により、`v-html`が使われているか、またはHTMLを無害化していないかを判断できる。

### 5. フレームワーク共通の問題

`v-html`に限らず、HTMLを直接挿入するあらゆる仕組み（`innerHTML`操作）は同じリスクを持つ。

| フレームワーク | 危険なコード | 説明 |
| --- | --- | --- |
| Vue.js | `<div v-html="input"></div>` | HTMLをそのまま描画 |
| React | `<div dangerouslySetInnerHTML={{ __html: input }} />` | 名前の通り危険な手段 |
| Angular | `[innerHTML]="input"` | HTMLバインディングで[[基礎 Part 11 XSSとsanitize|XSS]]の可能性 |
| Svelte | `{@html input}` | HTMLを展開する構文 |
| Vanilla JS | `element.innerHTML = input;` | もっとも単純だが危険な手法 |

つまり、問題の本質は「ユーザー入力をHTMLとして描画する」ことであり、Vue固有のものではない。

### 6. 対策：[[基礎 Part 11 XSSとsanitize|サニタイズ]]

危険を防ぐには、ユーザー入力を**[[基礎 Part 11 XSSとsanitize|サニタイズ]]（[[基礎 Part 11 XSSとsanitize|sanitize]]）**する。これは、危険なタグや属性を削除して安全なHTMLだけを残す処理である。

### [[基礎 Part 11 XSSとsanitize|DOMPurify]]による[[基礎 Part 11 XSSとsanitize|サニタイズ]]例

```javascript
import DOMPurify from 'dompurify';

const safeHtml = DOMPurify.sanitize(userInput);

```

これを`v-html`に渡すことで、危険なスクリプトを排除しつつ安全にHTMLを描画できる。

### Vueでの安全な使用例

```plain text
<template>
  <div v-html="safeHtml"></div>
</template>

<script setup>
import { computed, ref } from 'vue';
import DOMPurify from 'dompurify';

const userInput = ref('<img src=x onerror=alert(1)><b>こんにちは</b>');
const safeHtml = computed(() => DOMPurify.sanitize(userInput.value));
</script>

```

### 7. エスケープとの違い

| 処理方法 | 挙動 | Vueでの例 |
| --- | --- | --- |
| エスケープ | HTMLを文字列として表示 | `{{ text }}` |
| [[基礎 Part 11 XSSとsanitize|サニタイズ]] | 危険部分だけ削除しHTMLとして描画 | `v-html="DOMPurify.sanitize(text)"` |

### 8. 追加の防御策

- 入力フィルタを設け、不要なHTMLを禁止する
- Content Security Policy（CSP）を設定してスクリプト実行を制限する
- テスト段階で[[基礎 Part 11 XSSとsanitize|XSS]]を試し、脆弱性を確認する

### 9. まとめ

- `v-html`自体が危険なのではなく、`innerHTML`によるHTML描画が危険
- 攻撃者はアプリの挙動から容易に脆弱性を発見できる
- 対策の基本は[[基礎 Part 11 XSSとsanitize|サニタイズ]]（[[基礎 Part 11 XSSとsanitize|DOMPurify]]など）
- Vueに限らずReact、Angular、Svelte、jQueryなどでも同様の注意が必要

結論として、ユーザー入力をHTMLとして描画する場合は、必ず[[基礎 Part 11 XSSとsanitize|サニタイズ]]を行い、安全な範囲でのみHTMLレンダリングを許可することが重要である。
