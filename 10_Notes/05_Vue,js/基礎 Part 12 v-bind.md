---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-01T21:26:00
aliases: [v-bind, vbind, v bind, :, コロン]
---
## [[基礎 Part 12 v-bind|v-bind]]とは

[[基礎 Part 12 v-bind|v-bind]]は、Vue.jsにおいてHTMLの属性にJavaScriptの値を動的に結びつける[[基礎 Part 16 ディレクティブ|ディレクティブ]]。テンプレート内で変数の値を属性へ反映させるために使用する。

### 基本構文

```javascript
<img v-bind:src="imageUrl" alt="サンプル画像">

```

省略形：

```javascript
<img :src="imageUrl" alt="サンプル画像">

```

### 主な用途

- 属性に変数を[[基礎 Part 8 宣言・代入・再代入|代入]]
```javascript
<a :href="profileUrl">プロフィールへ</a>

```
- クラスやスタイルを動的に制御
```javascript
<div :class="{ active: isActive }"></div>
<div :style="{ color: textColor, fontSize: fontSize + 'px' }"></div>

```
- 複数属性をオブジェクトでまとめて適用
```javascript
<button v-bind="buttonAttrs">クリック</button>

```

### 同名属性の省略記法

属性名と変数名が同じ場合、値を省略できる。

```javascript
<img :src>

```

これは次と同義：

```javascript
<img v-bind:src="src">

```

### 他の例

```javascript
<button :disabled></button>
<MyComponent :title :content />

```

### 省略記法の扱い方

省略記法は便利だが、チーム開発では可読性や保守性の観点から使い方を統一すべき。

### メリット

- コードが短くなる
- Vueに慣れた人には見やすい

### デメリット

- 初心者が理解しづらい
- 検索性が下がる
- バインドミスを誘発する可能性

### チームでの推奨ルール例

| ケース | 推奨書き方 | 理由 |
| --- | --- | --- |
| 属性名と変数名が同じ | 省略可（`:src`） | 短く明快 |
| 属性名と変数名が異なる | 明示（`:src="imageUrl"`） | 可読性重視 |
| 公開用・共通[[基礎 Part 29 コンポーネント|コンポーネント]] | 明示 | 外部の理解を助ける |
| 学習・教育用コード | 明示 | 教育的観点から推奨 |

### Linterで統一する例

```javascript
// .eslintrc.cjs
'rules': {
  'vue/v-bind-style': ['error', 'longform'] // 省略禁止
}

```

### まとめ

- [[基礎 Part 12 v-bind|v-bind]]はVueで属性とデータを結びつける基本機能
- 省略記法は便利だが、チーム開発では使い方を統一するべき
- LinterやFormatterによるルール化が有効

## 関連
- [[基礎 Part 6 マスタッシュ構文]]
- [[基礎 Part 17 v-model]]
- [[基礎 Part 23 class指定]]
