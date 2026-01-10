---
base: "[[ナレッジベース.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - CS
作成日時: 2025-12-06T14:17:00
---
# npmの基礎まとめ

## npm (Node Package Manager)とは何か

npmはJavaScript/Node.jsにおけるパッケージ管理システムで、依存関係の取得、配置、更新、削除を自動で管理する仕組みである。ユーザーはpackage.jsonに依存を宣言するだけで、npmが依存グラフの解析と整合性の維持を行う。

## package.jsonと責任スコープ

package.jsonは、そのディレクトリ単位でのプロジェクト設定を表すファイルであり、以下を定義する。

- 使用する依存パッケージ
- スクリプトコマンド
- モジュール形式（commonjs/module）
- プロジェクトのメタ情報

package.jsonのスコープはディレクトリ単位であり、親のpackage.jsonを自動継承することはない。

## npm installの動作

npm installは、実行したディレクトリのpackage.jsonを参照して依存関係を解決する。解決されたパッケージはnode_modulesに配置される。node_modulesには依存パッケージの実体が格納される。

## node_modulesの構造

node_modulesには次の内容が含まれる。

- 依存パッケージの実体コード
- パッケージごとのpackage.json
- バイナリファイル
- 依存の依存（transitive dependencies）

昔のnpmは依存の入れ子構造を採用していたが、現在は可能な限りトップ階層に配置するフラットモデルを採用している。ただし、バージョン衝突がある場合のみネストする。

## フラット配置とネスト配置

npmは依存関係を次のルールで配置する。

1. プロジェクト自身が要求する依存はトップ階層に配置される。
2. 同じパッケージでバージョンが衝突する場合、依存元パッケージの内部にネストされる。
3. 複数バージョンをトップ階層に同時配置することはできない。

例:

```plain text
project/node_modules/lodash@5  ← プロジェクトが要求
project/node_modules/vue/node_modules/lodash@4  ← Vueが要求

```

## npmの目的

npmの目的は、人間が手動では管理できない複雑な依存関係を安全かつ自動的に解決することにある。具体的には以下を自動化している。

- 依存関係解析
- バージョン整合性の維持
- 衝突時のネスト処理
- node_modulesの構築
- package-lock.jsonによる再現性の保証

ユーザーは依存を宣言するだけでよく、内部の複雑な処理はnpmが管理する。

## 具体例: Vueとlodashの依存関係

Vueと別のライブラリが異なるバージョンのlodashを要求する例を示す。

プロジェクトのpackage.json:

```plain text
"dependencies": {
  "lodash": "5.0.0",
  "vue": "^3.0.0"
}

```

Vue内部のpackage.json（例）:

```plain text
"dependencies": {
  "lodash": "4.17.0"
}

```

npm installの結果:

```plain text
project/
  node_modules/
    lodash/ (5.0.0)
    vue/
      node_modules/
        lodash/ (4.17.0)

```

このように、バージョンが一致すればトップ階層にまとめられるが、衝突すれば依存元の内部にネストされる。

## 図: npmの依存解決モデル（フラット + 条件付きネスト）

```plain text
プロジェクト (root)
│
├─ node_modules/
│    ├─ A (rootが要求)
│    ├─ B (rootが要求)
│    ├─ C (rootが要求)
│    └─ vue/
│         └─ node_modules/
│              └─ C (vueが要求、バージョン衝突のため別配置)

```

この図が示す通り、npmは可能な限りフラットに配置し、衝突時のみネストする。このモデルにより、依存の重複を避けつつ整合性を保つことができる。

## npm install時のダウンロード元

npmは依存パッケージを次の場所から取得する。

- デフォルトレジストリ: `https://registry.npmjs.org/`

npm installを実行すると、必要なパッケージのメタデータをレジストリに問い合わせ、対応する`.tgz`ファイル（圧縮パッケージ）をダウンロードし、node_modulesに展開する。

## パッケージ配布形式

npmのパッケージはすべて`.tgz`形式で配布されている。例として、Vueの配布URLは次のようになる。

```plain text
https://registry.npmjs.org/vue/-/vue-3.4.0.tgz

```

これがダウンロードされ、ローカルで解凍されることでnode_modulesにパッケージが配置される。

## npmのキャッシュ

npmはダウンロードしたパッケージを次の場所にキャッシュする。

```plain text
~/.npm/_cacache/

```

一度ダウンロードしたパッケージはキャッシュに保存されるため、再度同じバージョンのinstallを行う際にはネットワークアクセスなしで復元されることがある。

## オフラインでのnpm install

キャッシュ内に必要なバージョンのパッケージが存在し、かつpackage-lock.jsonとの整合性が取れている場合、ネットワーク接続なしでもnpm installが可能である。

## npm registryとその運営

npm registryはnpm, Inc.によって管理されており、現在はGitHub（Microsoft傘下）が運営を担う。世界中のパッケージがここに公開され、npm install時に参照される。

## カスタムレジストリの利用

企業環境やセキュリティ要件に応じて、独自レジストリを利用することがある。設定例は以下の通り。

```plain text
npm config set registry https://example.com/my-registry/

```

代表的なプライベートレジストリには以下がある。

- Verdaccio
- GitHub Packages
- AWS CodeArtifact
- Azure Artifacts

## npm installの動作フロー

```plain text
ユーザーのPC
   │ package.jsonを読み込む
   ▼
npmが依存グラフを構築
   │
   ├─ npm registryへHTTPリクエスト
   │       │
   │       └─ .tgzパッケージを取得
   │
   ├─ ローカルキャッシュへ保存
   │
   └─ node_modulesへ展開して配置

```

## 関連
- [[コンパイルとビルド]]
- [[設定ファイル言語]]
