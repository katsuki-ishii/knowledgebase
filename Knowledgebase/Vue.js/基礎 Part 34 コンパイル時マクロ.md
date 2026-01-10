---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-27T12:17:00
---
# Vueにおけるコンパイル時マクロまとめ

## 1. コンパイル時マクロとは何か

コンパイル時マクロとは、**プログラムを実行する前（ビルド時）に、コードを解析・書き換えするための特別な記法**である。

Vue 3 では主に `<script setup>` 内で使われ、JavaScriptとして実行される関数ではない。

重要な点は以下の通り。

- 実行時には存在しない
- import しない（できない）
- Vue コンパイラだけが意味を理解する
- 最終的な JavaScript には残らない

## 2. なぜ import しないのか

`defineProps` や `defineEmits` は、Vue のランタイムライブラリ（`vue` パッケージ）に**関数の実体が存在しない**。

それらは Vue コンパイラがコードを読む際の「合図（マーカー）」であり、実行時に呼び出される対象ではないため、import する必要がない。

## 3. Vue の二つの世界

Vue には明確に二つのフェーズが存在する。

### (1) コンパイル時（ビルド時）

- .vue ファイルを解析する
- `<script setup>` を通常のコンポーネント定義に変換する
- コンパイル時マクロを解釈する

### (2) 実行時（ランタイム）

- ブラウザで動作する
- `ref`, `computed`, `watch` などが実行される
- import された関数が実体として存在する

## 4. 図解：コンパイル時マクロの位置づけ

```plain text
[ 開発者が書くコード ]

<script setup>
  defineProps({ msg: String })
</script>
        |
        |  Vue コンパイラが解析
        v
[ 書き換え後のコード（イメージ） ]

export default {
  props: {
    msg: String
  }
}
        |
        |  JavaScriptとして実行
        v
[ ブラウザで動作 ]

```

この過程で `defineProps` という名前自体は完全に消える。

## 5. 具体例①：defineProps

### 記述例

```typescript
<script setup>
const props = defineProps<{ msg: string }>()
</script>

```

### ポイント

- 関数呼び出しに見えるが実行されない
- TypeScript の型情報だけをコンパイラに渡す
- 実行時コストはゼロ

## 6. 具体例②：defineEmits

```typescript
<script setup>
const emit = defineEmits<{
  (e: 'submit'): void
}>()
</script>

```

これも同様に、イベント定義をコンパイラに伝えるためのマクロであり、実行時に関数として処理されるわけではない。

## 7. 実行時関数との違い

| 項目 | コンパイル時マクロ | 実行時関数 |
| --- | --- | --- |
| import | 不要 | 必要 |
| 実行されるか | されない | される |
| bundleに残るか | 残らない | 残る |
| if文で囲めるか | 不可 | 可 |

## 8. なぜ関数の形をしているのか

- TypeScript のジェネリクス構文を自然に使える
- 学習コストを下げられる
- 完全な独自構文を導入せずに済む

その結果、「関数に見えるが関数ではない」という設計になっている。

## 9. まとめ（重要）

Vue のコンパイル時マクロとは、`<script setup>` 内で使用される、Vue コンパイラ専用の構文である。

それは実行時の関数ではなく、コード生成のための合図であり、import を必要とせず、最終的な JavaScript には存在しない。

この理解ができると、`<script setup>` や Vue 3 の設計思想が一段深く理解できる。

## 関連
- [[基礎 Part 35 defineProps()]]
- [[基礎 Part 10 setup構文]]
