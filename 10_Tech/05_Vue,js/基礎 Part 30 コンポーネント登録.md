---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-21T23:29:00
aliases: [コンポーネント登録, グローバル登録, ローカル登録, global registration, local registration]
---
# [[基礎 Part 30 コンポーネント登録|コンポーネント登録]]

## 1. [[基礎 Part 29 コンポーネント|コンポーネント]]とは何か

[[基礎 Part 29 コンポーネント|コンポーネント]]とは、画面を構成する再利用可能な部品のこと。

例：

- ボタン
- アイコン
- モーダル

同じ見た目・同じ振る舞いを何度も使い回すための仕組み。

---

## 2. 登録とは何か

Vue に対して

「この部品を使ってよい」

と教える行為を **登録** と呼ぶ。

登録しない[[基礎 Part 29 コンポーネント|コンポーネント]]は Vue から見えない。

---

## 3. [[基礎 Part 30 コンポーネント登録|グローバル登録]]

### 定義

アプリ全体のどこからでも使えるようにする登録方法。

### イメージ図

```plain text
Vueアプリ全体
 ├─ 画面A  → 使用可能
 ├─ 画面B  → 使用可能
 └─ 画面C  → 使用可能

```

### 基本例

```javascript
app.component('BaseButton', BaseButton)

```

### 特徴

- [[基礎 Part 11 ESM　Import＆Export|import]] なしで使える
- 便利だが依存関係が見えない
- 使いすぎると設計が壊れる

---

## 4. [[基礎 Part 30 コンポーネント登録|ローカル登録]]

### 定義

特定の[[基礎 Part 29 コンポーネント|コンポーネント]]の中だけで使える登録方法。

### イメージ図

```plain text
親コンポーネント
 ├─ 子A（登録あり → 使える）
 └─ 子B（登録なし → 使えない）

```

### Vue 3（[[基礎 Part 10 setup構文|script setup]]）の例

```plain text
<script setup>
import MyButton from './MyButton.vue'
</script>

<template>
  <MyButton />
</template>

```

### 特徴

- 依存関係が明示される
- 安全で保守しやすい
- 実務ではこちらがデフォルト

---

## 5. Vue 3 の書き方について

### Options API（旧来・今も使用可）

```javascript
export default {
  components: { MyButton }
}

```

### Composition API（Vue 3 主流）

```javascript
<script setup>
import MyButton from './MyButton.vue'
</script>

```

意味は同じだが、書き方が違う。

---

## 6. 複数の[[基礎 Part 30 コンポーネント登録|グローバル登録]]

### メソッドチェーンによる登録

```javascript
app
  .component('BaseIcon', BaseIcon)
  .component('BaseButton', BaseButton)

```

- Vue 3として正式
- 少数なら可読性あり
- 数が増えると管理が破綻

---

## 7. 推奨されるまとめ登録パターン

### [[基礎 Part 29 コンポーネント|コンポーネント]]一覧を作る

```javascript
export const baseComponents = {
  BaseIcon,
  BaseButton,
  BaseModal,
}

```

### ループで一括登録

```javascript
Object.entries(baseComponents).forEach(([name, component]) => {
  app.component(name, component)
})

```

### 意味

- [[基礎 Part 30 コンポーネント登録|グローバル登録]]の許可リスト
- 設計ルールを1か所に集約
- 将来の削除・移行が容易

---

## 8. 実務での基本ルール

- 原則：[[基礎 Part 30 コンポーネント登録|ローカル登録]]
- 例外：BaseButton / BaseIcon など最小単位
- [[基礎 Part 30 コンポーネント登録|グローバル登録]]は「少なく・明示的に」

---

## 9. 学習の流れまとめ（図解）

```plain text
コンポーネントとは何か
        ↓
登録の必要性
        ↓
グローバル登録の便利さ
        ↓
危険性の理解
        ↓
ローカル登録の重要性
        ↓
Vue 3 的な書き方（script setup）
        ↓
複数グローバル登録の設計

```

---

## 10. 現在地

- Vue 3 の構文理解
- 登録スコープの違いを把握
- 設計としての是非を判断可能

実務レベルの基礎はすでに押さえている状態。

## 関連
- [[基礎 Part 29 コンポーネント]]
- [[基礎 Part 31  属性フォークスルー]]
