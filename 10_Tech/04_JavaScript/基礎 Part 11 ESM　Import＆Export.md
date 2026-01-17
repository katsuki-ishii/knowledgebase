---
base: "[[JavaScript.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-10-13T15:22:00
aliases: [ESM Import＆Export, ESM, Import, Export, import, export, ES Modules, ESModule]
---
# [[基礎 Part 11 ESM　Import＆Export|ES Modules]]（[[基礎 Part 11 ESM　Import＆Export|ESM]]）の仕組みまとめ

## 概要

[[基礎 Part 11 ESM　Import＆Export|ES Modules]]（略して [[基礎 Part 11 ESM　Import＆Export|ESM]]）は、JavaScript の公式モジュールシステムです。`import` と `export` を使って、コードをファイル単位で[[基礎 Part 13 分割代入|分割]]・共有できるようにする仕組みです。

---

## 基本構文

### 名前付きエクスポート（Named [[基礎 Part 11 ESM　Import＆Export|Export]]）

```javascript
// math.js
export const pi = 3.14
export function add(a, b) {
  return a + b
}

```

```javascript
// main.js
import { pi, add } from './math.js'
console.log(add(pi, 2))

```

- 複数エクスポート可能。
- `{}` の中で名前を指定して取り出す。
- [[基礎 Part 11 ESM　Import＆Export|import]]側の名前は [[基礎 Part 11 ESM　Import＆Export|export]]側と一致する必要がある。

---

### デフォルトエクスポート（[[基礎 Part 36 props バリデーション|Default]] [[基礎 Part 11 ESM　Import＆Export|Export]]）

```javascript
// greet.js
export default function greet(name) {
  console.log(`Hello, ${name}!`)
}

```

```javascript
// main.js
import greet from './greet.js'
greet('Memento')

```

- ファイルごとに1つだけ。
- [[基礎 Part 11 ESM　Import＆Export|import]]時に好きな名前をつけられる。
- ファイルの主役を示すのに使う。

---

### 両方使うことも可能

```javascript
// utils.js
export const version = '1.0'
export default function log(msg) {
  console.log(`[LOG] ${msg}`)
}

```

```javascript
// main.js
import log, { version } from './utils.js'
log(version)

```

---

## [[基礎 Part 11 ESM　Import＆Export|export]]できるもの

- 関数 (`function`)
- 変数 (`const`, `let`)
- クラス (`class`)
- オブジェクト、配列、プリミティブ値

```javascript
export const data = [1, 2, 3]
export class User {}
export default { appName: 'Memento' }

```

---

## [[基礎 Part 11 ESM　Import＆Export|import]]のバリエーション

| 構文 | 意味 |
| --- | --- |
| `import { a, b } from './x.js'` | 名前付き[[基礎 Part 11 ESM　Import＆Export|export]]を個別に読み込み |
| `import * as mod from './x.js'` | すべてをオブジェクトとして読み込み（`mod.a`, `mod.b`） |
| `import x from './x.js'` | [[基礎 Part 36 props バリデーション|default]] [[基礎 Part 11 ESM　Import＆Export|export]]を読み込み |
| `import x, { a, b } from './x.js'` | [[基礎 Part 36 props バリデーション|default]]と名前付き[[基礎 Part 11 ESM　Import＆Export|export]]を同時に読み込み |

---

## 参照の仕組み（ライブバインディング）

[[基礎 Part 11 ESM　Import＆Export|ESM]]の[[基礎 Part 11 ESM　Import＆Export|import]]は「コピー」ではなく「参照（ポインタ）」です。つまり、[[基礎 Part 11 ESM　Import＆Export|export]]元の値が変わると、[[基礎 Part 11 ESM　Import＆Export|import]]側も更新されます。

```javascript
// counter.js
export let count = 0
export function inc() { count++ }

```

```javascript
// main.js
import { count, inc } from './counter.js'
inc()
console.log(count) // => 1 （同期している）

```

---

## [[基礎 Part 36 props バリデーション|default]] [[基礎 Part 11 ESM　Import＆Export|export]] がある理由

| 観点 | [[基礎 Part 36 props バリデーション|default]] [[基礎 Part 11 ESM　Import＆Export|export]] | named [[基礎 Part 11 ESM　Import＆Export|export]] |
| --- | --- | --- |
| 数 | 1つだけ | 複数可 |
| [[基礎 Part 11 ESM　Import＆Export|import]]時の名前 | 自由に変更可 | [[基礎 Part 11 ESM　Import＆Export|export]]と同名のみ（またはasで変更） |
| 目的 | ファイルの主役を示す | 補助的な機能をまとめる |
| よく使う場面 | [[基礎 Part 29 コンポーネント|コンポーネント]]・[[設定ファイル言語|設定ファイル]] | ユーティリティ関数群 |

---

## まとめ

- [[基礎 Part 11 ESM　Import＆Export|ESM]]は `import / export` を使う公式モジュールシステム。
- `export` は名前付き（複数可）と `export default`（1つだけ）がある。
- `import` は値のコピーではなく参照（ライブリンク）。
- `{}` はモジュールオブジェクトから一部を取り出す構文。
- [[基礎 Part 36 props バリデーション|default]] [[基礎 Part 11 ESM　Import＆Export|export]] はファイルの主役を明確にできる。

---

## Vueにおける例

```javascript
// main.js
import { createApp } from 'vue'   // 名前付きexport
import App from './App.vue'       // デフォルトexport

createApp(App).mount('#app')

```

- Vue本体（`vue`）は複数の関数を名前付きで提供。
- `App.vue` はアプリの主役なので `export default`。