---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-06T17:26:00
aliases: [v-model, vmodel, v model, 双方向バインディング]
---
# [[基礎 Part 18 v-model|v-model]]とは

[[基礎 Part 18 v-model|v-model]]はVueが提供する「入力欄とデータを自動で結びつける仕組み」。

本来であれば、入力欄の値を取得してデータを書き換える処理と、データが変わったときに画面へ反映させる処理を手動で書く必要があるが、[[基礎 Part 18 v-model|v-model]]を使うことでこの2つが自動的に行われる。

### なぜ必要なのか

Webアプリでは、ユーザーが入力した内容をプログラム側で使うことが多い。例えば、名前入力欄に打ち込んだ文字を保存したり、検索欄の文字でフィルタリングしたりする。

通常であれば以下の2つを自分で書く必要がある。

1. 入力のたびにデータを書き換える
2. データが変わったときに画面へ反映する

[[基礎 Part 18 v-model|v-model]]はこの両方を1行でまとめてくれるため、コードが大幅に簡潔になる。

[[基礎 Part 18 v-model|v-model]]は、入力欄などのUI要素とアプリ内のデータを自動的に同期させるためのVue特有の仕組み。入力が変わればデータが変わり、データが変われば入力も変わるという双方向のつながりを提供する。

## [[基礎 Part 18 v-model|v-model]]がよく使われる場面

### 1. テキスト入力

最も基本的な使い方。ユーザーの入力した文字をそのまま使いたいときに使う。

### 具体例

ユーザー名を入力して画面に表示する。

```javascript
<script setup>
import { ref } from 'vue'
const username = ref('')
</script>

<template>
  <input v-model="username" />
  <p>あなたの名前は {{ username }} です</p>
</template>

```

この例では、入力欄の文字がそのまま username に入り、同時に画面にも反映される。

ユーザー名、メールアドレス、検索欄などの入力内容をそのまま状態として扱いたいときに使用する。

```javascript
<input v-model="username" />

```

### 2. チェックボックス

オンかオフかを扱うUIに適している。[[基礎 Part 18 v-model|v-model]]は自動的にtrue/falseを扱う。

### 具体例

```javascript
<script setup>
import { ref } from 'vue'
const isAgreed = ref(false)
</script>

<template>
  <label>
    <input type="checkbox" v-model="isAgreed" />
    利用規約に同意する
  </label>
  <p>同意状況: {{ isAgreed }}</p>
</template>

```

チェックすると true、外すと false が入る。

オンオフの状態をデータとして管理したいときに用いる。Boolean値が自動で同期される。

```javascript
<input type="checkbox" v-model="isAgreed" />

```

### 3. セレクトボックス

選択肢から選ばれた値をそのまま使いたいときに便利。

### 具体例

```javascript
<script setup>
import { ref } from 'vue'
const prefecture = ref('tokyo')
</script>

<template>
  <select v-model="prefecture">
    <option value="tokyo">東京</option>
    <option value="osaka">大阪</option>
    <option value="fukuoka">福岡</option>
  </select>
  <p>選択中: {{ prefecture }}</p>
</template>

```

ドロップダウンの選択結果を直接データとして扱いたい場合に便利。

```javascript
<select v-model="prefecture">
  <option value="tokyo">東京</option>
  <option value="osaka">大阪</option>
</select>

```

### 4. トグルスイッチ

カスタム[[基礎 Part 30 コンポーネント|コンポーネント]]でON/OFFを扱いたいときによく使う。[[基礎 Part 18 v-model|v-model]]を使うと状態管理が簡単になる。

### 具体例

```javascript
<ToggleSwitch v-model="darkMode" />
<p>ダークモード: {{ darkMode }}</p>

```

オンオフ切り替えのUIで内部状態を管理するときによく使う。

```javascript
<ToggleSwitch v-model="darkMode" />

```

### 5. モーダルの開閉状態管理

モーダルが開いているか閉じているかを親側で管理したいときに使う。

### 具体例

親[[基礎 Part 30 コンポーネント|コンポーネント]]:

```javascript
<EventModal v-model="isOpen" />
<button @click="isOpen = true">開く</button>

```

これにより、モーダル内部で閉じた際にも自動的にisOpenが更新される。

親[[基礎 Part 30 コンポーネント|コンポーネント]]と子[[基礎 Part 30 コンポーネント|コンポーネント]]間で開閉状態を同期したいときに使用する。

```javascript
<EventModal v-model="isOpen" />

```

## [[基礎 Part 18 v-model|v-model]]が多用される理由（基礎的な視点）

3. 入力欄とデータを同期させるためのコードを書く手間がなくなる
4. 値の変化を追う処理を自分で書かなくてよい
5. データとUIの一致が保証される
6. 特にフォームの多いアプリでは利用頻度が高い

初心者でも扱いやすく、Vueの利便性を強く感じられる部分でもある。

- 入力値と内部データの結びつきを自動化できるため、コード量を削減できる。
- UIの変化と状態の変化を一致させる必要がある場面で特に有効。
- 余計な[[基礎 Part 15 イベントオブジェクト|イベント]]監視や手動反映を書く必要がなくなるため、実装が単純になる。

## Mementoプロジェクトとの関連

Mementoでは、誕生日の入力、[[基礎 Part 15 イベントオブジェクト|イベント]]内容の入力、モーダルの開閉などが存在するため、[[基礎 Part 18 v-model|v-model]]が活躍しやすい。
