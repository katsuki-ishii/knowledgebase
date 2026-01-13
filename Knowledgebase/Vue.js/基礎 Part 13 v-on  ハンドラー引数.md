---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-03T11:38:00
aliases: [v-on ハンドラー引数, v-onハンドラー引数, v-on, $event, ハンドラー引数]
---
## [[基礎 Part 13 v-on  ハンドラー引数|v-on]]でハンドラーに引数を渡す方法まとめ

### 基本

`v-on`（または`@`）で[[基礎 Part 14 イベントオブジェクト|イベント]]ハンドラーを設定する際、引数を渡すにはメソッド呼び出しの形式で記述する。

```plain text
<button @click="sayHello('太郎')">クリック</button>

```

```javascript
methods: {
  sayHello(name) {
    console.log(`こんにちは、${name}`)
  }
}

```

### `$event` の利用

[[基礎 Part 14 イベントオブジェクト|イベントオブジェクト]]を同時に受け取りたい場合は、特別変数`$event`を使用する。

```plain text
<button @click="sayHello('太郎', $event)">クリック</button>

```

```javascript
methods: {
  sayHello(name, event) {
    console.log(name)       // '太郎'
    console.log(event.type) // 'click'
  }
}

```

### `$event` の正体

`$event`はVueが提供するテンプレート専用の特別変数で、DOM[[基礎 Part 14 イベントオブジェクト|イベントオブジェクト]]（`MouseEvent`など）を指す。テンプレート内では通常の変数として`event`を参照できないため、`$event`を使う必要がある。

Vue内部では以下のように展開される。

```plain text
@click="handler($event)"

```

↓ 内部的には

```javascript
element.addEventListener('click', event => handler(event))

```

### よく使う例

### 入力値を取得する

```plain text
<input @input="updateMessage($event)">

```

```javascript
methods: {
  updateMessage(event) {
    this.message = event.target.value
  }
}

```

### マウス座標を取得する

```plain text
<div @mousemove="trackMouse($event)">マウスを動かす</div>

```

```javascript
methods: {
  trackMouse(event) {
    console.log(event.clientX, event.clientY)
  }
}

```

### まとめ

| 目的 | 書き方 | 備考 |
| --- | --- | --- |
| 引数なし | `@click="doSomething"` | 基本形 |
| 引数あり | `@click="doSomething('値')"` | 即時評価されるが安全 |
| [[基礎 Part 14 イベントオブジェクト|イベントオブジェクト]]も欲しい | `@click="doSomething('値', $event)"` | `$event`でDOM[[基礎 Part 14 イベントオブジェクト|イベント]]を参照 |
| 無名関数で複数引数 | `@click="() => doSomething(a, b)"` | 明示的に関数で包む |

## 関連
- [[基礎 Part 13 v-on]]
- [[基礎 Part 14 イベントオブジェクト]]
