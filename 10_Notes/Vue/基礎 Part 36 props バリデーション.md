---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-27T15:23:00
aliases: [props バリデーション, propsバリデーション, props validation, type, default, required, validator]
---
# Vue 3 [[基礎 Part 35 defineProps()|defineProps]] 学習まとめ

## 1. [[基礎 Part 35 defineProps()|props]] とは何か

[[基礎 Part 35 defineProps()|props]] は「親[[基礎 Part 29 コンポーネント|コンポーネント]]から子[[基礎 Part 29 コンポーネント|コンポーネント]]に渡される入力値」。

子[[基礎 Part 29 コンポーネント|コンポーネント]]側では **読み取り専用** であり、直接変更してはいけない。

```javascript
// 親コンポーネント側の利用例
MyButton({
  label: '保存',
  disabled: true
})

```

```javascript
// 親コンポーネント側の利用例
MyButton({
  label: '保存',
  disabled: true
})

```

---

## 2. [[基礎 Part 35 defineProps()|defineProps]] の役割（軽い recap）

`defineProps` は子[[基礎 Part 29 コンポーネント|コンポーネント]]が受け取る [[基礎 Part 35 defineProps()|props]] を[[基礎 Part 8 宣言・代入・再代入|宣言]]するための仕組み。

- 親から何が渡ってくるかを明示する
- [[基礎 Part 29 コンポーネント|コンポーネント]]の API を分かりやすくする
- 開発中の警告や補完の材料になる

詳細な説明は別途まとめ済みのため、ここでは概要のみとする。

---

## 3. [[基礎 Part 36 props バリデーション|type]] とは何か

`type` は「その [[基礎 Part 35 defineProps()|props]] に入る値の大まかな種類」を示す。

```javascript
defineProps({
  title: String,
  count: Number,
  isOpen: Boolean,
  items: Array,
  user: Object,
  onClick: Function
})

```

### 特徴

- JavaScript に元からある型を指定する
- 開発中に型が違うと **警告（warning）** が出る
- アプリは止まらない

---

## 4. [[基礎 Part 36 props バリデーション|default]] とは何か

`default` は「親から [[基礎 Part 35 defineProps()|props]] が渡されなかった場合の初期値」。

```javascript
defineProps({
  count: {
    type: Number,
    default: 0
  }
})

```

### 注意点

配列・オブジェクトは関数で返す。

```javascript
defineProps({
  items: {
    type: Array,
    default: () => []
  }
})

```

---

## 5. [[基礎 Part 36 props バリデーション|required]] とは何か

`required: true` は「必ず渡してほしい [[基礎 Part 35 defineProps()|props]]」であることを示す。

```javascript
defineProps({
  title: {
    type: String,
    required: true
  }
})

```

渡されなかった場合、開発中に警告が出るが、アプリは止まらない。

---

## 6. [[基礎 Part 36 props バリデーション|validator]] とは何か

`validator` は `type` よりも **細かい条件** を自分で定義するための関数。

```javascript
defineProps({
  size: {
    type: String,
    default: 'medium',
    validator: (value) => {
      return ['small', 'medium', 'large'].includes(value)
    }
  }
})

```

### [[基礎 Part 36 props バリデーション|validator]] の性質

- 引数は渡された [[基礎 Part 35 defineProps()|props]] の値
- true を返すと OK
- false を返すと **警告が出るだけ**
- 値は修正されない

---

## 7. 警告どまりとはどういう意味か

### 例

```javascript
// 親コンポーネント側の利用例
MyButton({
  label: '保存',
  disabled: true
})

```

```javascript
defineProps({
  count: Number
})

```

この場合：

- コンソールに警告が出る
- 画面は普通に表示される
- 処理は継続する

本番[[コンパイルとビルド|ビルド]]では警告自体が出ない。

---

## 8. 図解：[[基礎 Part 35 defineProps()|props]] チェックの位置づけ

```plain text
親コンポーネント
    │
    │ props を渡す
    ▼
子コンポーネント
    │
    ├─ type チェック（開発中のみ）
    │
    ├─ validator チェック（開発中のみ）
    │
    ▼
実際の処理・表示
（警告が出ても止まらない）

```

---

## 9. 実務での正しい理解

- [[基礎 Part 36 props バリデーション|type]] / [[基礎 Part 36 props バリデーション|required]] / [[基礎 Part 36 props バリデーション|validator]] は **安全装置ではない**
- 開発中にミスへ気づくための仕組み
- 設計意図を伝えるドキュメントの役割が大きい

---

## 10. 本当に値を守りたい場合

TypeScript を使う。

```javascript
// TypeScript を使った場合の例（概念）
defineProps({
  size: 'small | medium | large',
  count: 'number'
})

```

この場合：

- 型が違うと[[コンパイルとビルド|コンパイル]]エラー
- 本番に届く前に止まる

---

## 11. まとめ

- [[基礎 Part 36 props バリデーション|type]]：値の大分類を示す
- [[基礎 Part 36 props バリデーション|default]]：未指定時の初期値
- [[基礎 Part 36 props バリデーション|required]]：必須[[基礎 Part 8 宣言・代入・再代入|宣言]]（警告のみ）
- [[基礎 Part 36 props バリデーション|validator]]：より細かい条件チェック（警告のみ）
- いずれも開発中の補助であり、実行時の保証ではない

## 関連
- [[基礎 Part 35 defineProps()]]
- [[基礎 Part 37 props 命名規則]]
