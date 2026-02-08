---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-06T21:25:00
aliases: [watchEffect, watch effect, WatchEffect]
---
# [[基礎 Part 22 watchEffect|watchEffect]] のまとめ

## 1. [[基礎 Part 22 watchEffect|watchEffect]] の本質

[[基礎 Part 22 watchEffect|watchEffect]] は「関数内で参照された [[基礎 Part 5 reactive関数|reactive]] 変数が変化したときに自動で再実行される仕組み」である。監視対象を指定する必要はなく、Vue が関数実行時に依存関係を自動で収集する。

## 2. 初回実行とは何か

[[基礎 Part 22 watchEffect|watchEffect]] は "登録された瞬間に、その場で一度実行される"。[[基礎 Part 10 setup構文|setup]] 内で [[基礎 Part 22 watchEffect|watchEffect]] を書くと、その行に到達した時点で即座に関数が実行される。

### 実行順序の例

```javascript
setup() {
  const count = ref(0)

  console.log('A')

  watchEffect(() => {
    console.log('B', count.value)
  })

  console.log('C')
}

```

実行順序は A → B(初回実行) → C となる。

### 図解

```plain text
コンポーネント作成
       ↓
setup() が呼ばれる
       ↓
watchEffect に到達
       ↓
★ 依存を集めるために関数を即時実行（初回）
       ↓
関数内で使われた reactive を記録
       ↓
依存が変わるたびに再実行

```

## 3. [[基礎 Part 22 watchEffect|watchEffect]] の役割

[[基礎 Part 22 watchEffect|watchEffect]] の目的は「状態の変化に応じて[[基礎 Part 21 副作用と純粋関数|副作用]]を発生させる」ことである。[[基礎 Part 21 副作用と純粋関数|副作用]]とは、Vue の[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]システムの外側に影響を与える処理のこと。

### よく使われるケース

- localStorage への保存
- ページタイトル更新
- [[URI・URL・URN|URL]] クエリ更新
- 計算結果を別の変数へ反映

## 4. 具体例

### localStorage への自動保存

```javascript
const theme = ref('light')

watchEffect(() => {
  localStorage.setItem('theme', theme.value)
})

```

テーマ変更のたびに保存され、初回にも保存が行われる。

### 選択週をもとに割合を計算

```javascript
const selectedWeek = ref(100)
const totalWeeks = 90 * 52
const percentage = ref(0)

watchEffect(() => {
  percentage.value = (selectedWeek.value / totalWeeks) * 100
})

```

selectedWeek を読むが percentage は依存関係に含まれないため安全に更新できる。

## 5. 無限ループが起きる理由

[[基礎 Part 22 watchEffect|watchEffect]] は「関数内で読んだ [[基礎 Part 5 reactive関数|reactive]] が変わると再実行される」ため、読んだ値を自分で更新すると無限に再実行される。

### 典型的な危険例

```javascript
watchEffect(() => {
  count.value++
})

```

count を読み、count を更新し、また読み…の繰り返しで無限ループになる。

### 安全な更新との違い

```plain text
読む値と書く値が同じ → 無限ループ
読む値と書く値が異なる → 安全

```

## 6. [[基礎 Part 22 watchEffect|watchEffect]] を使うときの判断基準

[[基礎 Part 22 watchEffect|watchEffect]] の初回実行は必ず発生するため、次を基準に選択する。

- 初期化時にその処理が動いて問題ないなら [[基礎 Part 22 watchEffect|watchEffect]] が適している
- 初回に実行されては困る処理（例: API 取得）は [[基礎 Part 22 watchEffect|watchEffect]] には適さない

## まとめ

- [[基礎 Part 22 watchEffect|watchEffect]] は依存を自動で追跡する仕組み
- 初回実行は「依存関係を集めるための一回」
- [[基礎 Part 21 副作用と純粋関数|副作用]]を発生させる用途に向いている
- 読んだ [[基礎 Part 5 reactive関数|reactive]] を自分で更新すると無限ループになる
- 初回実行されても問題ない処理に使うのが基本
