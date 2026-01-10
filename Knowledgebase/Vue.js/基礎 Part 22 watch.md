---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-07T16:42:00
---
# Vue Composition API における watch のまとめ

## 1. watch の本質

watch はリアクティブな値が変化した際に副作用処理を実行する仕組み。副作用とは API の呼び出し、バリデーション、localStorage との同期、ログ出力などを指す。値を返すのではなく、処理を実行する点が computed との決定的な違い。

## 2. 基本構文

```javascript
watch(source, callback, options?)

```

- source: ref、reactive の getter 関数、複数値の配列など
- callback: (newValue, oldValue) => {...}
- options: immediate、deep、flush など

## 3. 基本的な流れ（図解）

```plain text
[ reactive 値 ]
       │ 値が変化する
       ▼
    watch(source)
       │ 発火判定
       ▼
 callback(newValue, oldValue)

```

## 4. 具体例

### ref をそのまま監視する

```javascript
const count = ref(0)

watch(count, (newVal, oldVal) => {
  console.log("カウントが変化", oldVal, "→", newVal)
})

```

### reactive のプロパティを監視（getter を使う）

```javascript
const state = reactive({ name: "katsuki", age: 25 })

watch(
  () => state.age,
  (newVal, oldVal) => {
    console.log("age が変化した")
  }
)

```

### 複数の値を同時に監視

```javascript
const first = ref("")
const last = ref("")

watch([first, last], ([newF, newL], [oldF, oldL]) => {
  console.log("どちらかが変わった")
})

```

## 5. getter 関数とは

getter 関数は「値を返すだけの関数」。watch が何を監視するべきかを正確に指定するために使用する。

具体例:

```javascript
() => state.age

```

### 図解

```plain text
 state ----------------------
  │      │        │
 name   age      email
          ▲
          └ watch が getter により監視対象を特定

```

## 6. watch の代表的な使用場面

### 1. 入力値に応じて外部処理を行う

```javascript
watch(keyword, (v) => {
  fetchSearch(v)
})

```

### 2. フォームのバリデーション

```javascript
watch(email, (value) => {
  emailError.value = validateEmail(value)
})

```

### 3. 外部ストレージとの同期

```javascript
watch(currentPage, (v) => {
  localStorage.setItem("page", v)
})

```

### 4. ID の変更に応じてデータ再取得

```javascript
watch(selectedId, async (id) => {
  detail.value = await fetchDetail(id)
})

```

## 7. watchEffect との違い（図解）

### watch

```plain text
あなたが指定した値のみを監視

[count] ───► watch ───► callback

```

### watchEffect

```plain text
watchEffect 内でアクセスした全ての reactive を監視

         ┌──────── count
         │
watchEffect()
         │──────── state.name
         └──────── その他参照値

```

## 8. watch を使うべきでない場面

### 単なる値の計算

```javascript
const fullName = computed(() => first.value + last.value)

```

### UI 表示の切り替えだけの場合

watch を使う必要はない。テンプレート側の条件分岐や computed で対応可能。

## 9. 実務でのベストプラクティス

1. ref はそのまま watch する
2. reactive の部分監視は getter 関数で統一する
3. 複数監視は配列で行う
4. immediate を使う場合は理由をコメントで明示する
5. watchEffect は用途を限定する

## 10. watch の options の代表例

### immediate

watch 登録時に初回の callback を即時実行する。

例:

```javascript
watch(keyword, handler, {
  immediate: true
})

```

用途:

- 画面ロード時に初回データを取得したい
- 現在の状態に基づく初期処理を行いたい

---

### deep

オブジェクトや配列のネストした変更も検知する。

Composition API で reactive をそのまま監視した場合は内部的に deep 相当になるが、getter を使う監視では deep は不要。

例:

```javascript
watch(state, handler, {
  deep: true
})

```

用途:

- オブジェクト内部のプロパティが変更された時に処理したい

---

### flush

watch の callback を実行するタイミングを制御する。

- `flush: 'pre'` コンポーネント更新前に実行
- `flush: 'post'` DOM 更新後に実行（デフォルト）
- `flush: 'sync'` 同期的に即実行（特殊用途）

例:

```javascript
watch(count, handler, {
  flush: 'post'
})

```

用途:

- DOM 更新後でないと正しく取得できない情報を扱う場合（post）
- DOM 更新前にロジックを実行したい場合（pre）
- 特殊なパフォーマンスチューニング（sync）

---

## 11. 総合図解

```plain text
 ref / getter / reactive
          │
          ▼
       watch
          │ options(immediate / deep / flush) の動作に従う
          ▼
 callback(newValue, oldValue)

```

```plain text
 ref / getter / reactive
          │
          ▼
       watch
          │ flush タイミングに従って発火
          ▼
 callback(newValue, oldValue)

```

以上を押さえておくと、Composition API における watch の動作と使いどころが明確に理解できる。

## 関連
- [[基礎 Part 21 watchEffect]]
- [[基礎 Part 20 副作用と純粋関数]]
- [[基礎 Part 18 computed]]
