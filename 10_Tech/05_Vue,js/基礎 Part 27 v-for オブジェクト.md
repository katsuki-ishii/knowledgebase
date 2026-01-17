---
base: "[[Vue,js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-12-07T23:13:00
aliases: [v-for オブジェクト, v-forオブジェクト, v-for オブジェクト, v-for object]
---
# [[基礎 Part 26 v-for 配列|v-for]] でオブジェクトを扱う

## 1. [[基礎 Part 26 v-for 配列|v-for]] の基本

`v-for` は配列やオブジェクトの中身を順番に取り出して、要素を繰り返し描画するための仕組み。Vue 特有の文法。

### 図解

```plain text
オブジェクト
{ a: 1, b: 2, c: 3 }
      │
      ▼
v-for
      │
      ▼
1回目 → (value=1, key="a", index=0)
2回目 → (value=2, key="b", index=1)
3回目 → (value=3, key="c", index=2)

```

---

## 2. オブジェクトを回すときの基本形

```plain text
<div v-for="(value, key, index) in obj">
  {{ index }} : {{ key }} = {{ value }}
</div>

```

### 取り出せる 3 つの要素

- value: 値
- [[基礎 Part 26 v-for 配列|key]]: キー名
- [[基礎 Part 2 index.htmlとmain.js|index]]: 0 から始まる番号

---

## 3. 具体例

### データ

```plain text
user: {
  name: "Taro",
  age: 20,
  country: "Japan"
}

```

### [[基礎 Part 26 v-for 配列|v-for]]

```plain text
<div v-for="(value, key) in user" :key="key">
  {{ key }} : {{ value }}
</div>

```

### 出力イメージ

```plain text
name : Taro
age : 20
country : Japan

```

---

## 4. [[基礎 Part 2 index.htmlとmain.js|index]] と [[基礎 Part 26 v-for 配列|key]] は省略できるか

### 結論

- [[基礎 Part 26 v-for 配列|v-for]] の引数としては省略できる
- だが、:[[基礎 Part 26 v-for 配列|key]] 属性にキー名を使う必要があるため「[[基礎 Part 26 v-for 配列|key]] を省略すると現実的に困る」

### 図解

```plain text
【良い】
(key を受け取っている)

v-for="(value, key) in user" :key="key"
             │              │
             └──使える───┘

```

```plain text
【困る】
(key を受け取っていない)

v-for="value in user" :key="???"
                        │
                        └─ key に渡すものが無い

```

---

## 5. [[基礎 Part 2 index.htmlとmain.js|index]] を [[基礎 Part 26 v-for 配列|key]] に使うべきでない理由

```plain text
:key="index"

```

これは動くが、順番が変わったときに Vue が別の要素と誤認するので再描画が多くなり非効率。

### 図解

```plain text
[before] a, b, c
 index   0  1  2

[after]  b, c, a
 index   0  1  2  ← 別物として扱われる

```

---

## 6. 結論まとめ

- オブジェクトでは `(value, key, index)` の形式で取り出せる
- [[基礎 Part 26 v-for 配列|key]] は実質必須（:[[基礎 Part 26 v-for 配列|key]] に使うため）
- [[基礎 Part 2 index.htmlとmain.js|index]] はほぼ不要
- 安定した [[基礎 Part 26 v-for 配列|key]] を付けることが重要

---

## 7. 実際のアプリ開発での使用例

オブジェクトを [[基礎 Part 26 v-for 配列|v-for]] で回す場面は実務でも頻出する。以下に代表的な例を示す。

### 例1: ユーザープロフィールの設定項目を自動生成するケース

```plain text
settings: {
  username: "ユーザー名を入力",
  email: "メールアドレスを入力",
  age: "年齢を入力"
}

```

フォーム入力欄を自動生成する。

```plain text
<div v-for="(placeholder, key) in settings" :key="key">
  <label :for="key">{{ key }}</label>
  <input :id="key" :placeholder="placeholder" />
</div>

```

### 例2: 権限管理などのフラグを一覧表示するケース

```plain text
permissions: {
  admin: true,
  edit: false,
  view: true
}

```

管理画面で権限を表示する。

```plain text
<div v-for="(enabled, name) in permissions" :key="name">
  {{ name }} : {{ enabled ? "有効" : "無効" }}
</div>

```

### 例3: API のレスポンスがオブジェクト形式の統計値のとき

```plain text
stats: {
  total_users: 1520,
  active_users: 845,
  new_signups: 32
}

```

統計ダッシュボードに表示する。

```plain text
<div v-for="(value, label) in stats" :key="label">
  {{ label }} : {{ value }}
</div>

```

### 例4: 翻訳テキスト（i18n）の一覧を確認するケース

```plain text
messages: {
  hello: "こんにちは",
  goodbye: "さようなら",
  thanks: "ありがとう"
}

```

翻訳漏れチェックのために一覧化する。

```plain text
<div v-for="(text, key) in messages" :key="key">
  {{ key }} : {{ text }}
</div>

```

以上のように、オブジェクトをそのまま並べて画面に出したい場面では [[基礎 Part 26 v-for 配列|v-for]] が自然に使われることが多い。

必要なら、このまとめに続けて「実践編」や「よくあるエラー集」も追加可能。

## 関連
- [[基礎 Part 26 v-for 配列]]
- [[基礎 Part 28 v-for 数字]]
