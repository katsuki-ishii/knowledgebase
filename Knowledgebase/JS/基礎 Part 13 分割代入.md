---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - JavaScript
作成日時: 2025-12-07T23:01:00
---
# JavaScriptの分割代入まとめ

## 1. 分割代入とは

配列やオブジェクトの中にある値を、短く明確な書き方で変数として取り出すための文法。通常の代入よりもコード量を減らし、目的が読み取りやすくなる。

## 2. 配列の分割代入

配列は「順番」で値を取り出す。

```javascript
const arr = [10, 20, 30];
const [a, b, c] = arr;

```

必要のない要素はカンマで飛ばせる。

```javascript
const [, second] = [1, 2, 3];

```

残りの要素をまとめて受け取るには、JavaScript文法である rest 構文を使う。

```javascript
const [first, ...rest] = [1, 2, 3, 4];

```

## 3. オブジェクトの分割代入

オブジェクトは「キー（名前）」を指定して取り出す。

```javascript
const user = { name: "Alice", age: 20 };
const { name, age } = user;

```

### 別名をつける（alias）

左側がオブジェクトのキー、右側が受け取る変数名になる。

```javascript
const { name: userName, age: userAge } = user;

```

この書き方は、同じキーを持つ複数のオブジェクトを扱うときや、より意味のある変数名にしたいときに使う。

### デフォルト値

該当するキーが無い場合に備えて初期値を設定できる。

```javascript
const { country = "Japan" } = user;

```

## 4. 配列とオブジェクトの組み合わせ

実務上よく見られるパターンとして、配列の中にオブジェクトが入るケースがある。

```javascript
const users = [
  { name: "Alice", age: 20 },
  { name: "Bob", age: 30 }
];

const [{ name: firstUserName }] = users;

```

## 5. 関数引数での分割代入

必要なプロパティだけを直接受け取れるため、実務で頻繁に使用される。Reactでも多用される書き方。

```javascript
function printUser({ name, age }) {
  console.log(name, age);
}

printUser({ name: "Carol", age: 25 });

```

## 6. 実務上の具体例

### APIレスポンスの処理

サーバーから返ってくるデータのうち、必要な項目だけを抜き出す。

```javascript
const response = {
  status: 200,
  data: {
    id: 42,
    title: "Task",
    completed: false,
    createdAt: "2025-01-01"
  }
};

const { data: { id, title } } = response;

```

### Reactのpropsでの利用

コンポーネントが受け取るオブジェクトから必要な部分だけを抜き出す。

```javascript
function UserCard({ name, age, role }) {
  return `${name} (${age}) - ${role}`;
}

```

### 設定オブジェクトからの抽出

オプションを受け取る関数で、デフォルト値と組み合わせて使う場合が多い。

```javascript
function createUser({ name, role = "user" }) {
  return { name, role };
}

createUser({ name: "Dave" });

```

## 7. よくあるつまずき

- 配列は順番、オブジェクトはキー名で取り出す。
- オブジェクト分割代入でキー名を間違えると undefined になる。
- 別名構文では「左がキー、右が変数名」であることを混同しやすい。

## 8. まとめ

- 配列は `[a, b]` のように順番で取り出す。
- オブジェクトは `{ a, b }` のようにキー名で取り出す。
- 別名は `{ a: x }` のように書く。
- デフォルト値は `{ a = 10 }` のように書