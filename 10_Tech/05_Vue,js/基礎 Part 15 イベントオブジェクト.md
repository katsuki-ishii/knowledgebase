---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-03T11:39:00
aliases: [イベントオブジェクト, イベント, event, $event, Event]
---
## Vueの[[基礎 イベントオブジェクト|イベントオブジェクト Part 14]]について

### [[基礎 イベントオブジェクト|イベントオブジェクト Part 14]]とは

Vue.jsの[[基礎 イベントオブジェクト|イベントオブジェクト Part 14]]は、ユーザーの操作（クリック、入力、スクロールなど）によって発生した[[基礎 イベントオブジェクト|イベント Part 15]]に関する情報を持つオブジェクト。ブラウザの標準的な `Event` オブジェクトに基づいており、Vueではテンプレート内でこれを簡単に扱えるようにしている。

---

### 基本構造

Vueでは `v-on` [[基礎 ディレクティブ|ディレクティブ Part 16]]（または `@` 省略形）を使って[[基礎 イベントオブジェクト|イベント Part 15]]をバインドする。

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

このとき `handleClick` に渡される `event` が[[基礎 イベントオブジェクト|イベントオブジェクト Part 14]]。

---

### 主なプロパティ

| プロパティ名 | 説明 |
| --- | --- |
| `type` | [[基礎 イベントオブジェクト|イベント Part 15]]の種類（例: `click`, `input`） |
| `target` | [[基礎 イベントオブジェクト|イベント Part 15]]を発生させた要素（例: `<button>`、`<input>`） |
| `currentTarget` | 現在[[基礎 イベントオブジェクト|イベント Part 15]]リスナーがバインドされている要素 |
| `preventDefault()` | デフォルトの動作をキャンセルする（例: フォーム送信を止める） |
| `stopPropagation()` | [[基礎 イベントオブジェクト|イベント Part 15]]のバブリング（親要素への伝搬）を止める |
| `key` | キーボード[[基礎 イベントオブジェクト|イベント Part 15]]時に押されたキー |
| `clientX`, `clientY` | マウスクリック時の座標 |

---

### [[基礎 イベントオブジェクト|イベントオブジェクト Part 14]]の渡し方

テンプレートで引数を指定しない場合、Vueは自動的に[[基礎 イベントオブジェクト|イベントオブジェクト Part 14]]を渡す。

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
`event` という名前は一般的に[[基礎 イベントオブジェクト|イベントオブジェクト Part 14]]を指すと認識されるため、コードの意味が一目で分かる。
2. **引数の順序が明確になる**
`$event` の位置を明示的に書けるため、[[基礎 イベントオブジェクト|イベント Part 15]]が第一引数に来る規則を統一できる。
```plain text
<button @click="handleClick($event, 'week-12')">クリック</button>

```
3. **型推論・補完が効きやすい**
TypeScriptやエディタが[[基礎 イベントオブジェクト|イベントオブジェクト Part 14]]の型を推論しやすくなる。
```typescript
function handleInput(event: InputEvent) {
  const target = event.target as HTMLInputElement
  console.log(target.value)
}

```
4. **意図が明確で、統一的なコードになる**
すべてのハンドラで `event` を第一引数にすれば、どの引数が[[基礎 イベントオブジェクト|イベント Part 15]]かを即座に判断できる。

---

### まとめ

| ポイント | 内容 |
| --- | --- |
| [[基礎 イベントオブジェクト|イベントオブジェクト Part 14]]はDOM[[基礎 イベントオブジェクト|イベント Part 15]]情報を持つ標準オブジェクト |   |
| Vueはテンプレート内で自動的に `event` を渡す |   |
| `$event` を使えば他の引数と併用可能 |   |
| `event` を第一引数に統一すると可読性・保守性が向上する |   |
| Composition APIでも同じ仕組みで動作する |   |

## 関連
- [[基礎 Part 13 v-on Part 13]]
- [[基礎 v-on  ハンドラー引数]]
