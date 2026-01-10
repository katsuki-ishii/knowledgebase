---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-03T10:52:00
---
### 1. v-onとイベントとは

イベントとは、ユーザーの操作やブラウザ上で発生する「出来事」のこと。たとえばクリック、キー入力、マウス移動、スクロール、ページ読み込みなどがある。Vue.jsでは、`v-on`ディレクティブ（または省略形の`@`）を使うことで、これらの出来事を監視し、発生した際に特定の処理を実行できる。

VueはDOM（Document Object Model）に対して内部的に`addEventListener`を自動で設定する。したがって、`v-on:click="handleClick"`は、ネイティブJavaScriptの

```javascript
document.querySelector('button').addEventListener('click', handleClick)

```

と同じ意味になる。

この仕組みによって、開発者は直接DOM操作を記述せずに、テンプレート内でイベントを宣言的に扱うことができる。

### 2. ハンドラーとは

ハンドラー（イベントハンドラー）とは、特定のイベントが発生したときに実行される関数のこと。Vueでは、テンプレート内で指定したメソッド名がコンポーネントの`methods`オブジェクト内に定義されていれば、自動的にその関数が呼び出される。

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

- `@click` はクリックイベントを監視している。
- `sayHello` はクリック時に実行されるハンドラー。
- 関数は`methods`オブジェクト内で定義されており、Vueが自動的に関連付ける。

### 複数のイベントを同じ要素にバインドすることも可能

```plain text
<button @mouseenter="onEnter" @mouseleave="onLeave">ホバー</button>

```

### 3. メソッドとインラインの違い

Vueではイベント発生時の処理を「メソッド形式」または「インライン形式」で書ける。

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

この場合、クリックイベントが発生するたびに`count`が1増える。テンプレート内に処理を直接書いているためシンプルだが、処理が増えると可読性が落ちる。一般的に、1行以内の簡単な操作にのみ使うのが望ましい。

### （3）実務的な使い分け

| 処理内容 | 書き方 | 理由 |
| --- | --- | --- |
| 1行で終わる（カウンターなど） | インライン | 短く書けて可読性を保てる |
| 条件分岐や副作用を含む | メソッド | 複雑化しても管理しやすい |
| チーム開発・長期保守 | メソッド | 意図を明確化できる |

### 4. イベントオブジェクト

イベントが発生した際、Vueは自動的に「イベントオブジェクト」をハンドラーに渡すことができる。このオブジェクトにはイベントに関する情報（発生元、イベントの種類、押されたキーなど）が含まれる。

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

`$event`を引数に指定することで、VueはネイティブのDOMイベントオブジェクトをハンドラーに渡す。

このオブジェクトを使うと、クリック位置（`event.clientX`や`event.clientY`）や押されたキー（`event.key`）などを取得できる。

### 5. Native DOM Events

Vueで指定できるイベント名は、基本的にブラウザが提供するネイティブDOMイベントと同じ。これらはMDN（Mozilla Developer Network）の「Document Object Model (DOM) Events」ページで確認できる。

主なカテゴリは以下の通り。

| カテゴリ | イベント例 | 説明 |
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
| イベント | ブラウザやユーザーの操作によって発生する出来事 |
| ハンドラー | イベント発生時に実行される関数 |
| メソッド | コンポーネント内で定義した再利用可能な関数 |
| インライン | 処理をその場で直接記述するスタイル |
| イベントオブジェクト | 発生したイベントに関する詳細情報（`$event`で取得） |
| Native DOM Events | ブラウザが提供する標準イベント（MDNに一覧あり） |

## 関連
- [[基礎 Part 13 v-on  ハンドラー引数]]
- [[基礎 Part 14 イベントオブジェクト]]
