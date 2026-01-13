---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-10-15T00:34:00
aliases: [reactive関数, reactive, reactive(), Reactive]
---
## [[基礎 Part 5 reactive関数|reactive関数]]とは

`reactive`関数は、Vue.jsのComposition APIで提供される[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]システムの一部であり、**オブジェクトや配列を[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]に変換するための関数**である。[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]とは、データが変化したときに自動的にビュー（画面）へ反映される性質を指す。

Vue.jsでは、開発者が手動でDOMを更新するのではなく、状態（データ）を更新すればVueが自動的に画面を再描画する。この[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]な仕組みを作るために、`reactive`はオブジェクトを**Proxy（プロキシ）**でラップし、プロパティの変更を検知するようになっている。

---

## [[基礎 Part 4 ref関数|ref関数]]との違い

`ref`と`reactive`はどちらも[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]な状態を作るための関数だが、用途と動作に違いがある。

| 項目 | [[基礎 Part 4 ref関数|ref()]] | [[基礎 Part 5 reactive関数|reactive()]] |
| --- | --- | --- |
| 対象 | プリミティブ値（数値・文字列など） | オブジェクト・配列など複合データ |
| [[基礎 Part 19 リアクティブエフェクト|リアクティブ]]化の範囲 | 1つの値のみ | オブジェクト全体（ネストも含む） |
| アクセス方法 | `.value` が必要 | `.value` 不要 |
| 内部構造 | `{ value: ... }` というオブジェクト | Proxyでオブジェクト全体を監視 |
| テンプレート内の使用 | `.value` を省略可能 | そのまま使える |

---

## [[基礎 Part 5 reactive関数|reactive関数]]の特徴

1. **オブジェクト全体を[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]化する**
オブジェクトや配列を渡すと、その中のすべてのプロパティが監視対象になる。
2. **ネストしたオブジェクトも自動で[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]になる**
内部に入れ子構造を持つ場合でも、すべて自動で[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]化される。
3. `**.value**`** が不要**
`ref`と違い、値を参照するときに`.value`を挟む必要がない。

---

## 使用例

### [[基礎 Part 4 ref関数|ref]]の例

```javascript
import { ref } from 'vue'

const count = ref(0)

// script内では .value が必要
count.value++

// template内では .value は不要
// <p>{{ count }}</p>

```

### [[基礎 Part 5 reactive関数|reactive]]の例

```javascript
import { reactive } from 'vue'

const user = reactive({
  name: 'Alice',
  age: 25,
  address: { city: 'Tokyo' }
})

// ネストしたプロパティもリアクティブ
user.address.city = 'Osaka'

```

---

## [[基礎 Part 4 ref関数|ref関数]]にオブジェクトを渡した場合の挙動

```javascript
import { ref } from 'vue'

const user = ref({
  name: 'Alice',
  age: 25
})

```

このように `ref` にオブジェクトを渡すと、Vueは内部的に次のように処理する。

4. 渡された値がオブジェクトや配列の場合、内部で`reactive()`を呼び出す。
5. 生成された[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]オブジェクトを`ref`の`.value`に格納する。
6. 結果的に`ref({ ... })`は`{ value: reactive({ ... }) }`という構造になる。

図で表すと以下のようになる。

```plain text
ref({ ... }) → { value: reactive({ ... }) }

```

つまり、`ref`にオブジェクトを渡すと、内部的には`reactive`によってProxy化されたオブジェクトをラップする形になる。

この仕組みにより、`ref`を使えばプリミティブ値でもオブジェクトでも同じような書き方で[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]変数を扱える。Vueが内部で適切に処理してくれるため、開発者は型を意識せずに`ref`を使うことができる。

---

## 注意点

7. `reactive`はプリミティブ値に対しては効果がない。プリミティブ値を扱う場合は`ref`を使う。
8. `reactive`で作ったオブジェクトを[[基礎 Part 13 分割代入|分割代入]]すると、[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]性が失われる。その場合は`toRefs()`を使用する。
```javascript
const state = reactive({ count: 0 })
const { count } = toRefs(state)

```
9. `ref`と`reactive`は併用可能であり、オブジェクト内に`ref`を含めても問題ない。

---

## まとめ

- 単一の値には`ref()`を使う。
- 複数の状態を持つオブジェクトや配列には`reactive()`を使う。
- `reactive`ではネストされたオブジェクトも自動的に[[基礎 Part 19 リアクティブエフェクト|リアクティブ]]化される。
- script内で`ref`は`.value`が必要だが、`reactive`は不要。
- `ref`にオブジェクトを渡すと、内部的には`reactive`が呼ばれてProxy化されている。

## 関連
- [[基礎 Part 4 ref関数]]
- [[基礎 Part 8 reactiveとrefの併用]]
- [[基礎 Part 19 リアクティブエフェクト]]
