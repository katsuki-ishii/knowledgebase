---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-26T22:11:00
aliases: [reactiveとrefの併用, reactiveとref, reactive ref]
---
## 1. [[基礎 Part 8 reactiveとrefの併用|reactiveとref]]の役割

Vue 3の[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]システムには、2つの主要な関数がある。

| 関数 | 対象 | 主な用途 | 備考 |
| --- | --- | --- | --- |
| `reactive()` | オブジェクト・配列 | 複数のプロパティを持つデータを[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]化 | Proxyで包む |
| `ref()` | プリミティブ値 | 数値や文字列など単一の値を[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]化 | `{ value: ... }` で包む |

`reactive` はオブジェクトや配列を直接 `Proxy` 化する。一方、プリミティブ値（数値・文字列など）はオブジェクトではないため、`Proxy` を直接かけることができない。このため、Vueはそれを `{ value: ... }` というラッパーオブジェクトに包んで管理する。これが `ref()` の仕組みである。

---

## 2. [[基礎 Part 4 ref関数|ref]]を[[基礎 Part 5 reactive関数|reactive]]の中で使うときの挙動

`reactive()` の中に `ref()` を入れると、Vueは特別なアンラップ（unwrap）ルールを適用する。つまり、状況によって `.value` の有無が変わる。

| 使用箇所 | `.value` 必要 | 挙動 |
| --- | --- | --- |
| オブジェクトのプロパティ | 不要 | 自動的にアンラップされる |
| ネストされたオブジェクト | 不要 | 再帰的にアンラップされる |
| 配列の要素 | 必要 | アンラップされない |
| テンプレート内 | 不要 | Vueが自動で`.value`を解決する |

### 例1：[[基礎 Part 9 オブジェクト内のref|オブジェクト内のref]]

```javascript
const state = reactive({
  count: ref(0)
});

state.count++; // OK（アンラップされる）

```

### 例2：配列内の[[基礎 Part 4 ref関数|ref]]

```javascript
const state = reactive({
  list: [ref(1), ref(2)]
});

state.list[0].value++; // .valueが必要

```

配列内ではアンラップされないため、テンプレートでも `.value` を明示する必要がある。

---

## 3. Proxyがプリミティブに使えない理由

JavaScriptの`Proxy`は、オブジェクトを監視するための仕組みである。プリミティブ値（数値、文字列、真偽値など）はオブジェクトではないため、`Proxy`を直接適用できない。

```javascript
new Proxy(1, {}); // TypeError: Cannot create proxy with a non-object

```

このため、Vueはプリミティブ値を `{ value: ... }` というオブジェクトで包み、そのオブジェクトを`reactive`化することで[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]性を実現している。

---

## 4. 混乱を避けるための指針

1. オブジェクトを[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]にしたいときは `reactive()` を使う。
2. 単一の値を[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]にしたいときは `ref()` を使う。
3. 配列を扱う場合は、配列全体を `ref([])` で包むと管理しやすい。

```javascript
const todos = ref([
  { id: 1, text: 'Study Vue' },
  { id: 2, text: 'Exercise' }
]);

todos.value.push({ id: 3, text: 'Sleep' });

```

このように設計すれば、`.value` の扱いが一貫し、混乱を最小限にできる。

## 関連
- [[基礎 Part 4 ref関数]]
- [[基礎 Part 5 reactive関数]]
- [[基礎 Part 9 オブジェクト内のref]]
