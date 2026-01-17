---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-03T10:52:00
aliases: [v-on, von, v on, @, アットマーク]
---
### 1. [[基礎 Part 13 v-on|v-on]]と[[基礎 Part 14 イベントオブジェクト|イベント]]とは

[[基礎 Part 14 イベントオブジェクト|イベント]]とは、ユーザーの操作やブラウザ上で発生する「出来事」のこと。たとえばクリック、キー入力、マウス移動、スクロール、ページ読み込みなどがある。Vue.jsでは、`v-on`[[基礎 Part 16 ディレクティブ|ディレクティブ]]（または省略形の`@`）を使うことで、これらの出来事を監視し、発生した際に特定の処理を実行できる。

VueはDOM（Document Object Model）に対して内部的に`addEventListener`を自動で設定する。したがって、`v-on:click="handleClick"`は、ネイティブJavaScriptの

```javascript
document.querySelector('button').addEventListener('click', handleClick)

```

と同じ意味になる。

この仕組みによって、開発者は直接DOM操作を記述せずに、テンプレート内で[[基礎 Part 14 イベントオブジェクト|イベント]]を[[基礎 Part 8 宣言・代入・再代入|宣言]]的に扱うことができる。

### 2. ハンドラーとは

ハンドラー（[[基礎 Part 14 イベントオブジェクト|イベント]]ハンドラー）とは、特定の[[基礎 Part 14 イベントオブジェクト|イベント]]が発生したときに実行される関数のこと。Vueでは、テンプレート内で指定したメソッド名が[[基礎 Part 29 コンポーネント|コンポーネント]]の`methods`オブジェクト内に定義されていれば、自動的にその関数が呼び出される。

### 例：基本的なクリックハンドラー

```plain text
<template>
  <button @click="sayHello">クリック</button>
</template>

<script>
export default {
  methods: {
    sayHello() {
      alert('こんにちは！');
    }
  }
}
</script>

```

この場合：

- `@click` はクリック[[基礎 Part 14 イベントオブジェクト|イベント]]を監視している。
- `sayHello` はクリック時に実行されるハンドラー。
- 関数は`methods`オブジェクト内で定義されており、Vueが自動的に関連付ける。

### 複数の[[基礎 Part 14 イベントオブジェクト|イベント]]を同じ要素にバインドすることも可能

```plain text
<button @mouseenter="onEnter" @mouseleave="onLeave">ホバー</button>

```

### 3. メソッドとインラインの違い

Vueでは[[基礎 Part 14 イベントオブジェクト|イベント]]発生時の処理を「メソッド形式」または「インライン形式」で書ける。

### （1）メソッド形式

メソッドとして定義した関数を呼び出す形式。複雑な処理、再利用が必要な処理、または複数行になる場合に適している。

```plain text
<template>
  <button @click="increment">カウント: {{ count }}</button>
</template>

<script>
export default {
  data() {
    return { count: 0 }
  },
  methods: {
    increment() {
      this.count++
      console.log('カウントが増加しました:', this.count)
    }
  }
}
</script>

```

この方法は関数に名前を与えられるため、コードの意味が明確になり、保守性が高くなる。また、他の要素から同じ関数を再利用することもできる。

### （2）インライン形式

短い処理をテンプレート内に直接記述する形式。即時的で単純な操作に向いている。

```plain text
<template>
  <button @click="count++">クリック: {{ count }}</button>
</template>

<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

```

この場合、クリック[[基礎 Part 14 イベントオブジェクト|イベント]]が発生するたびに`count`が1増える。テンプレート内に処理を直接書いているためシンプルだが、処理が増えると可読性が落ちる。一般的に、1行以内の簡単な操作にのみ使うのが望ましい。

### （3）実務的な使い分け

| 処理内容 | 書き方 | 理由 |
| --- | --- | --- |
| 1行で終わる（カウンターなど） | インライン | 短く書けて可読性を保てる |
| 条件分岐や[[基礎 Part 20 副作用と純粋関数|副作用]]を含む | メソッド | 複雑化しても管理しやすい |
| チーム開発・長期保守 | メソッド | 意図を明確化できる |

### 4. [[基礎 Part 14 イベントオブジェクト|イベントオブジェクト]]

[[基礎 Part 14 イベントオブジェクト|イベント]]が発生した際、Vueは自動的に「[[基礎 Part 14 イベントオブジェクト|イベントオブジェクト]]」をハンドラーに渡すことができる。このオブジェクトには[[基礎 Part 14 イベントオブジェクト|イベント]]に関する情報（発生元、[[基礎 Part 14 イベントオブジェクト|イベント]]の種類、押されたキーなど）が含まれる。

```plain text
<button @click="handleClick($event)">クリック</button>

<script>
export default {
  methods: {
    handleClick(event) {
      console.log(event.type)   // 'click'
      console.log(event.target) // クリックされたDOM要素
    }
  }
}
</script>

```

`$event`を引数に指定することで、VueはネイティブのDOM[[基礎 Part 14 イベントオブジェクト|イベントオブジェクト]]をハンドラーに渡す。

このオブジェクトを使うと、クリック位置（`event.clientX`や`event.clientY`）や押されたキー（`event.key`）などを取得できる。

### 5. Native DOM Events

Vueで指定できる[[基礎 Part 14 イベントオブジェクト|イベント]]名は、基本的にブラウザが提供するネイティブDOM[[基礎 Part 14 イベントオブジェクト|イベント]]と同じ。これらはMDN（Mozilla Developer Network）の「Document Object Model (DOM) Events」ページで確認できる。

主なカテゴリは以下の通り。

| カテゴリ | [[基礎 Part 14 イベントオブジェクト|イベント]]例 | 説明 |
| --- | --- | --- |
| マウス操作 | `click`, `dblclick`, `mousedown`, `mouseup`, `mousemove`, `contextmenu` | クリックやマウス移動などの操作 |
| キーボード | `keydown`, `keyup`, `keypress` | キーが押された・離されたなどの操作 |
| フォーム | `input`, `change`, `submit`, `focus`, `blur`, `reset` | 入力やフォーム送信関連 |
| ウィンドウ | `load`, `resize`, `scroll`, `beforeunload` | ページ読み込みやリサイズなど |
| タッチ | `touchstart`, `touchmove`, `touchend`, `touchcancel` | スマホやタブレットの操作 |
| ドラッグ | `drag`, `dragover`, `drop`, `dragend` | 要素のドラッグ＆ドロップ操作 |
| メディア | `play`, `pause`, `ended`, `volumechange` | 動画・音声の再生状態の変化 |

これらはVueでもそのまま使用でき、`@イベント名="ハンドラー"`として記述するだけで反応する。

例：

```plain text
<div @dragover.prevent @drop="handleDrop">ここにファイルをドロップ</div>

```

### 6. まとめ

| 概念 | 内容 |
| --- | --- |
| [[基礎 Part 14 イベントオブジェクト|イベント]] | ブラウザやユーザーの操作によって発生する出来事 |
| ハンドラー | [[基礎 Part 14 イベントオブジェクト|イベント]]発生時に実行される関数 |
| メソッド | [[基礎 Part 29 コンポーネント|コンポーネント]]内で定義した再利用可能な関数 |
| インライン | 処理をその場で直接記述するスタイル |
| [[基礎 Part 14 イベントオブジェクト|イベントオブジェクト]] | 発生した[[基礎 Part 14 イベントオブジェクト|イベント]]に関する詳細情報（`$event`で取得） |
| Native DOM Events | ブラウザが提供する標準[[基礎 Part 14 イベントオブジェクト|イベント]]（MDNに一覧あり） |

## 関連
- [[基礎 Part 13 v-on  ハンドラー引数]]
- [[基礎 Part 14 イベントオブジェクト]]
