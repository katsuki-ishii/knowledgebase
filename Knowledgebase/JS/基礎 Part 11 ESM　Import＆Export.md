---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-10-13T15:22:00
---
# ES Modules（ESM）の仕組みまとめ

## 概要

ES Modules（略して ESM）は、JavaScript の公式モジュールシステムです。`import` と `export` を使って、コードをファイル単位で分割・共有できるようにする仕組みです。

---

## 基本構文

### 名前付きエクスポート（Named Export）

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
- import側の名前は export側と一致する必要がある。

---

### デフォルトエクスポート（Default Export）

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
- import時に好きな名前をつけられる。
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

## exportできるもの

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

## importのバリエーション

| 構文 | 意味 |
| --- | --- |
| `import { a, b } from './x.js'` | 名前付きexportを個別に読み込み |
| `import * as mod from './x.js'` | すべてをオブジェクトとして読み込み（`mod.a`, `mod.b`） |
| `import x from './x.js'` | default exportを読み込み |
| `import x, { a, b } from './x.js'` | defaultと名前付きexportを同時に読み込み |

---

## 参照の仕組み（ライブバインディング）

ESMのimportは「コピー」ではなく「参照（ポインタ）」です。つまり、export元の値が変わると、import側も更新されます。

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

## default export がある理由

| 観点 | default export | named export |
| --- | --- | --- |
| 数 | 1つだけ | 複数可 |
| import時の名前 | 自由に変更可 | exportと同名のみ（またはasで変更） |
| 目的 | ファイルの主役を示す | 補助的な機能をまとめる |
| よく使う場面 | コンポーネント・設定ファイル | ユーティリティ関数群 |

---

## まとめ

- ESMは `import / export` を使う公式モジュールシステム。
- `export` は名前付き（複数可）と `export default`（1つだけ）がある。
- `import` は値のコピーではなく参照（ライブリンク）。
- `{}` はモジュールオブジェクトから一部を取り出す構文。
- default export はファイルの主役を明確にできる。

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