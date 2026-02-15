---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-06T18:21:00
aliases: [副作用と純粋関数, 副作用, 純粋関数, side effect, pure function]
---
## 1. [[基礎 Part 21 副作用と純粋関数|副作用]]とは何か

[[基礎 Part 21 副作用と純粋関数|副作用]]とは「関数が値を返す以外に外の世界へ影響を与えること」を指す。

Vue のコンテキストでは特に次が該当する。

- [[基礎 Part 5 reactive関数|reactive]] な変数を書き換える
- DOM を直接操作する
- API を呼ぶ
- タイマーを動かす
- console.log のように外部へ影響を出す

Vue の[[基礎 Part 20 リアクティブエフェクト|リアクティブ]]システムは「値の変化」をトリガーに再計算や再描画を行うため、

[[基礎 Part 19 computed|computed]] 内に[[基礎 Part 21 副作用と純粋関数|副作用]]があると動作が不安定になりやすい。

---

## 2. [[基礎 Part 21 副作用と純粋関数|純粋関数]]とは何か

[[基礎 Part 21 副作用と純粋関数|副作用]]を理解したうえで、その対概念となる「[[基礎 Part 21 副作用と純粋関数|純粋関数]]」を説明する。

[[基礎 Part 21 副作用と純粋関数|純粋関数]]には以下の条件がある。

### (1) 入力が同じなら出力も常に同じ

関数に渡された入力値だけで出力が決まり、

外部状態に影響されない。

### (2) 外部の状態を変更しない（[[基礎 Part 21 副作用と純粋関数|副作用]]を持たない）

外部変数の変更、ログ出力、API 呼び出しなど、

関数外へ影響を及ぼさない。

### 具体例

[[基礎 Part 21 副作用と純粋関数|純粋関数]]:

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

## 3. [[基礎 Part 19 computed|computed]] と[[基礎 Part 21 副作用と純粋関数|純粋関数]]の関係

Vue の [[基礎 Part 19 computed|computed]] は「[[基礎 Part 21 副作用と純粋関数|純粋関数]]として振る舞うこと」を前提に作られている。

したがって、以下の性質を持つ。

- [[基礎 Part 5 reactive関数|reactive]] な依存が変わらない限り再計算しない（キャッシュされる）
- 外部の状態を変更しないことを期待している

このため [[基礎 Part 19 computed|computed]] 内で[[基礎 Part 21 副作用と純粋関数|副作用]]を行うと挙動が崩れる。

---

## 4. [[基礎 Part 19 computed|computed]] の正しい使い方と危険な例

### 無限ループの具体例

[[基礎 Part 19 computed|computed]] 内で [[基礎 Part 5 reactive関数|reactive]] な値を更新すると、依存関係が自分自身に向かって再帰的につながり、無限ループが発生する。

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

この場合も count.value の更新により [[基礎 Part 19 computed|computed]] が再評価され続け、条件が揃うまでループが発生する。終了条件が曖昧な場合はフリーズの原因とな

### 純粋で正しい [[基礎 Part 19 computed|computed]]

```javascript
const count = ref(2);
const doubled = computed(() => count.value * 2);

```

### 危険例：[[基礎 Part 19 computed|computed]] 内で状態を変更する

```javascript
const doubled = computed(() => {
  count.value++;
  return count.value * 2;
});

```

この場合、

[[基礎 Part 5 reactive関数|reactive]] value → [[基礎 Part 19 computed|computed]] 再計算 → [[基礎 Part 5 reactive関数|reactive]] value 更新 → 再計算…

という無限ループが発生し、ブラウザがフリーズする可能性がある。

### 非推奨例：軽い[[基礎 Part 21 副作用と純粋関数|副作用]]

```javascript
const doubled = computed(() => {
  console.log("評価");
  return count.value * 2;
});

```

再評価回数が Vue に依存するため、挙動理解を妨げる。

### 重大な[[基礎 Part 21 副作用と純粋関数|副作用]]例

```javascript
const userName = computed(() => {
  fetch('/api/user');
  return data.value;
});

```

再計算のたびに API が発火する危険がある。

---

## 5. [[基礎 Part 21 副作用と純粋関数|副作用]]を書く場所

[[基礎 Part 19 computed|computed]] は計算専用であり、値変化に応じた処理（[[基礎 Part 21 副作用と純粋関数|副作用]]）は [[基礎 Part 23 watch|watch]] または [[基礎 Part 22 watchEffect|watchEffect]] で行う。

### 正しい例

```javascript
watch(() => count.value, (newVal) => {
  doSomething(newVal);
});

```

---

## 6. 図解：[[基礎 Part 21 副作用と純粋関数|副作用]]と [[基礎 Part 19 computed|computed]] の仕組み

### [[基礎 Part 21 副作用と純粋関数|副作用]]を持たない [[基礎 Part 19 computed|computed]]

```plain text
 reactive value
       ↓
  computed getter
       ↓
   計算のみ
       ↓
  return result

```

### [[基礎 Part 21 副作用と純粋関数|副作用]]を持つ [[基礎 Part 19 computed|computed]]（危険）

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

- [[基礎 Part 21 副作用と純粋関数|副作用]]とは「値を返す以外に外部へ影響する動作」。
- [[基礎 Part 21 副作用と純粋関数|純粋関数]]は「入力が同じなら出力が同じ」「外部へ影響しない」。
- [[基礎 Part 19 computed|computed]] は[[基礎 Part 21 副作用と純粋関数|純粋関数]]を前提として動作する。
- [[基礎 Part 21 副作用と純粋関数|副作用]]は [[基礎 Part 23 watch|watch]] 系に切り出すのが正しい設計である。
