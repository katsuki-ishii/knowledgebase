---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-06T18:21:00
---
## 1. 副作用とは何か

副作用とは「関数が値を返す以外に外の世界へ影響を与えること」を指す。

Vue のコンテキストでは特に次が該当する。

- reactive な変数を書き換える
- DOM を直接操作する
- API を呼ぶ
- タイマーを動かす
- console.log のように外部へ影響を出す

Vue のリアクティブシステムは「値の変化」をトリガーに再計算や再描画を行うため、

computed 内に副作用があると動作が不安定になりやすい。

---

## 2. 純粋関数とは何か

副作用を理解したうえで、その対概念となる「純粋関数」を説明する。

純粋関数には以下の条件がある。

### (1) 入力が同じなら出力も常に同じ

関数に渡された入力値だけで出力が決まり、

外部状態に影響されない。

### (2) 外部の状態を変更しない（副作用を持たない）

外部変数の変更、ログ出力、API 呼び出しなど、

関数外へ影響を及ぼさない。

### 具体例

純粋関数:

```javascript
function double(x) {
  return x * 2;
}

```

非純粋な関数（外部状態を変更）:

```javascript
let y = 0;
function double(x) {
  y++;
  return x * y;
}

```

非純粋な関数（外部へ影響）:

```javascript
function double(x) {
  console.log("呼ばれた");
  return x * 2;
}

```

---

## 3. computed と純粋関数の関係

Vue の computed は「純粋関数として振る舞うこと」を前提に作られている。

したがって、以下の性質を持つ。

- reactive な依存が変わらない限り再計算しない（キャッシュされる）
- 外部の状態を変更しないことを期待している

このため computed 内で副作用を行うと挙動が崩れる。

---

## 4. computed の正しい使い方と危険な例

### 無限ループの具体例

computed 内で reactive な値を更新すると、依存関係が自分自身に向かって再帰的につながり、無限ループが発生する。

### 例 1: 基本的な無限ループ

```javascript
const count = ref(0);

const doubled = computed(() => {
  count.value++; // reactive な値を書き換えてしまう
  return count.value * 2;
});

```

この処理の流れは次のようになる。

```plain text
count.value が更新される
       ↓
computed doubled が再評価される
       ↓
computed 内で count.value++ が実行される
       ↓
count.value が更新される
       ↓
computed doubled が再評価される
       ↓
...（無限に繰り返し）

```

結果として、CPU 使用率が急上昇し、ブラウザがフリーズまたはクラッシュすることがある。

### 例 2: 条件つきでも危険

```javascript
const count = ref(0);

const result = computed(() => {
  if (count.value < 5) {
    count.value++; // 条件つきでも副作用となり無限再評価につながる
  }
  return count.value;
});

```

この場合も count.value の更新により computed が再評価され続け、条件が揃うまでループが発生する。終了条件が曖昧な場合はフリーズの原因とな

### 純粋で正しい computed

```javascript
const count = ref(2);
const doubled = computed(() => count.value * 2);

```

### 危険例：computed 内で状態を変更する

```javascript
const doubled = computed(() => {
  count.value++;
  return count.value * 2;
});

```

この場合、

reactive value → computed 再計算 → reactive value 更新 → 再計算…

という無限ループが発生し、ブラウザがフリーズする可能性がある。

### 非推奨例：軽い副作用

```javascript
const doubled = computed(() => {
  console.log("評価");
  return count.value * 2;
});

```

再評価回数が Vue に依存するため、挙動理解を妨げる。

### 重大な副作用例

```javascript
const userName = computed(() => {
  fetch('/api/user');
  return data.value;
});

```

再計算のたびに API が発火する危険がある。

---

## 5. 副作用を書く場所

computed は計算専用であり、値変化に応じた処理（副作用）は watch または watchEffect で行う。

### 正しい例

```javascript
watch(() => count.value, (newVal) => {
  doSomething(newVal);
});

```

---

## 6. 図解：副作用と computed の仕組み

### 副作用を持たない computed

```plain text
 reactive value
       ↓
  computed getter
       ↓
   計算のみ
       ↓
  return result

```

### 副作用を持つ computed（危険）

```plain text
 reactive value
       ↓
  computed getter
       ↓
  外部状態を変更 ←──────┐
       ↓                   │
  return result            │
                           │
 reactive value が変化 ────┘
 （無限ループ）

```

---

## 7. まとめ

- 副作用とは「値を返す以外に外部へ影響する動作」。
- 純粋関数は「入力が同じなら出力が同じ」「外部へ影響しない」。
- computed は純粋関数を前提として動作する。
- 副作用は watch 系に切り出すのが正しい設計である。

## 関連
- [[基礎 Part 18 computed]]
- [[基礎 Part 21 watchEffect]]
- [[基礎 Part 22 watch]]
