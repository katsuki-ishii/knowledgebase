---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-03T11:39:00
---
## Vueのイベントオブジェクトについて

### イベントオブジェクトとは

Vue.jsのイベントオブジェクトは、ユーザーの操作（クリック、入力、スクロールなど）によって発生したイベントに関する情報を持つオブジェクト。ブラウザの標準的な `Event` オブジェクトに基づいており、Vueではテンプレート内でこれを簡単に扱えるようにしている。

---

### 基本構造

Vueでは `v-on` ディレクティブ（または `@` 省略形）を使ってイベントをバインドする。

```plain text
<template>
  <button @click="handleClick">クリック</button>
</template>

<script setup>
function handleClick(event) {
  console.log(event) // イベントオブジェクト
}
</script>

```

このとき `handleClick` に渡される `event` がイベントオブジェクト。

---

### 主なプロパティ

| プロパティ名 | 説明 |
| --- | --- |
| `type` | イベントの種類（例: `click`, `input`） |
| `target` | イベントを発生させた要素（例: `<button>`、`<input>`） |
| `currentTarget` | 現在イベントリスナーがバインドされている要素 |
| `preventDefault()` | デフォルトの動作をキャンセルする（例: フォーム送信を止める） |
| `stopPropagation()` | イベントのバブリング（親要素への伝搬）を止める |
| `key` | キーボードイベント時に押されたキー |
| `clientX`, `clientY` | マウスクリック時の座標 |

---

### イベントオブジェクトの渡し方

テンプレートで引数を指定しない場合、Vueは自動的にイベントオブジェクトを渡す。

```plain text
<button @click="handleClick">Click</button>

```

上記は内部的に `handleClick(event)` を呼び出している。

複数の引数を渡したい場合は `$event` を明示的に指定する。

```plain text
<button @click="handleClick('week-12', $event)">クリック</button>

```

```javascript
function handleClick(weekId, event) {
  console.log(weekId, event.target)
}

```

---

### 第一引数に `event` を統一する利点

1. **暗黙的な理解ができる**
`event` という名前は一般的にイベントオブジェクトを指すと認識されるため、コードの意味が一目で分かる。
2. **引数の順序が明確になる**
`$event` の位置を明示的に書けるため、イベントが第一引数に来る規則を統一できる。
```plain text
<button @click="handleClick($event, 'week-12')">クリック</button>

```
3. **型推論・補完が効きやすい**
TypeScriptやエディタがイベントオブジェクトの型を推論しやすくなる。
```typescript
function handleInput(event: InputEvent) {
  const target = event.target as HTMLInputElement
  console.log(target.value)
}

```
4. **意図が明確で、統一的なコードになる**
すべてのハンドラで `event` を第一引数にすれば、どの引数がイベントかを即座に判断できる。

---

### まとめ

| ポイント | 内容 |
| --- | --- |
| イベントオブジェクトはDOMイベント情報を持つ標準オブジェクト |   |
| Vueはテンプレート内で自動的に `event` を渡す |   |
| `$event` を使えば他の引数と併用可能 |   |
| `event` を第一引数に統一すると可読性・保守性が向上する |   |
| Composition APIでも同じ仕組みで動作する |   |

## 関連
- [[基礎 Part 13 v-on]]
- [[基礎 Part 13 v-on  ハンドラー引数]]
