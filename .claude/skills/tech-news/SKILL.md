---
name: tech-news
description: 新技術Labメンバーが同じ材料で学び・共有・議論できるよう、Reddit・Hacker News・セキュリティブログ・はてブから技術トレンドを横断収集し、新聞風のHTMLまとめを作る
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Glob, WebFetch
argument-hint: "[ソース追加・除外の指示(省略可)]"
---

## 目的

新技術Lab――生成AIやガジェットなどの新しい技術を実際に触って検証し、ベルシステム24のビジネスへの活用可能性を探る場――において、複数の技術ニュースソースから重要な話題を横断的に収集・整理し、メンバーが同じ材料をもとに学び・共有・議論できるNotion風のまとめを作る。

## 処理フロー

全ソースのデータ取得を並列で実行し、その後に注目トピック選定 → 詳細取得 → タグ付け → HTML生成の順で進める。

---

## Source 1: Reddit（10サブレディット）

WebFetchはreddit.comをブロックするため、Bashでcurl + Node.jsを使う。

```bash
SUBS="netsec cybersecurity OpenAI ClaudeAI ChatGPT claude ClaudeCode ObsidianMD programming technology"
for sub in $SUBS; do
  echo "===SUBREDDIT:$sub==="
  curl -s -L -H "User-Agent: neta-trend-collector/1.0 (trend analysis tool)" "https://old.reddit.com/r/$sub/hot.json?t=day&limit=10"
  echo ""
done | node -e "
let buf='';process.stdin.on('data',d=>buf+=d);process.stdin.on('end',()=>{
  const parts=buf.split(/===SUBREDDIT:(\w+)===/);
  const results={};
  for(let i=1;i<parts.length;i+=2){
    const sub=parts[i],jsonStr=parts[i+1].trim();
    try{
      const json=JSON.parse(jsonStr);
      results[sub]=json.data.children.map(c=>({
        title:c.data.title,ups:c.data.ups,comments:c.data.num_comments,
        url:'https://www.reddit.com'+c.data.permalink
      }));
    }catch(e){results[sub]=[{error:e.message}];}
  }
  console.log(JSON.stringify(results,null,2));
});
"
```

Redditの注目トピック詳細取得（並列実行）:

```bash
curl -s -L -H "User-Agent: neta-trend-collector/1.0 (trend analysis tool)" \
  "https://old.reddit.com/r/{subreddit}/comments/{post_id}/{slug}.json?limit=15" | \
  node -e "
let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{
  const j=JSON.parse(d);
  const post=j[0].data.children[0].data;
  const comments=j[1].data.children.slice(0,10).map(c=>({author:c.data.author,body:c.data.body,ups:c.data.ups}));
  console.log(JSON.stringify({title:post.title,selftext:post.selftext,url:post.url,ups:post.ups,
    preview:post.preview?.images?.[0]?.source?.url,thumbnail:post.thumbnail,comments},null,2));
})"
```

---

## Source 2: Hacker News

HN Firebase APIを使用。topstoriesから上位30件のIDを取得し、各IDの詳細を取得する。

```bash
curl -s "https://hacker-news.firebaseio.com/v0/topstories.json" | node -e "
const https=require('https');
let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{
  const ids=JSON.parse(d).slice(0,30);
  let done=0;const results=[];
  ids.forEach(id=>{
    https.get('https://hacker-news.firebaseio.com/v0/item/'+id+'.json',res=>{
      let b='';res.on('data',c=>b+=c);res.on('end',()=>{
        try{
          const item=JSON.parse(b);
          results.push({title:item.title,score:item.score,comments:item.descendants||0,
            hn_url:'https://news.ycombinator.com/item?id='+item.id,
            source_url:item.url||null});
        }catch(e){}
        done++;
        if(done===ids.length){
          results.sort((a,b)=>b.score-a.score);
          console.log(JSON.stringify(results,null,2));
        }
      });
    });
  });
});
"
```

取得フィールド:
- `title`: タイトル
- `score`: ポイント数
- `descendants`: コメント数
- HNコメントページURL: `https://news.ycombinator.com/item?id={id}`（元記事URLではなくこちらを使う）

---

## Source 3: セキュリティブログ

WebFetchツールで以下の2サイトから最新記事を取得する:

### Aikido Security Blog
- URL: `https://www.aikido.dev/blog`
- WebFetchで取得し、最新5-10件の記事タイトルとURLを抽出

### Wiz Blog
- URL: `https://www.wiz.io/blog`
- WebFetchで取得し、最新5-10件の記事タイトルとURLを抽出

各記事について:
- タイトル
- 記事URL
- 公開日（取得できれば）
- 概要（取得できれば）

---

## Source 4: はてなブックマーク IT

RSS形式で取得。以下5カテゴリのRSSを全て取得し、重複を除去してマージする。

```bash
HATENA_URLS=(
  "https://b.hatena.ne.jp/hotentry/it.rss"
  "https://b.hatena.ne.jp/hotentry/it/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0.rss"
  "https://b.hatena.ne.jp/hotentry/it/AI%E3%83%BB%E6%A9%9F%E6%A2%B0%E5%AD%A6%E7%BF%92.rss"
  "https://b.hatena.ne.jp/hotentry/it/%E3%81%AF%E3%81%A6%E3%81%AA%E3%83%96%E3%83%AD%E3%82%B0%EF%BC%88%E3%83%86%E3%82%AF%E3%83%8E%E3%83%AD%E3%82%B8%E3%83%BC%EF%BC%89.rss"
  "https://b.hatena.ne.jp/hotentry/it/%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E6%8A%80%E8%A1%93.rss"
  "https://b.hatena.ne.jp/hotentry/it/%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%8B%E3%82%A2.rss"
)
for url in "${HATENA_URLS[@]}"; do
  curl -s "$url"
done | node -e "
let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{
  const items=new Map();
  const regex=/<item rdf:about=\"([^\"]+)\">\s*<title>([^<]+)<\/title>[\s\S]*?<hatena:bookmarkcount>(\d+)<\/hatena:bookmarkcount>/g;
  let m;
  while((m=regex.exec(d))!==null){
    const url=m[1],title=m[2],bookmarks=parseInt(m[3]);
    if(!items.has(url)||items.get(url).bookmarks<bookmarks){
      items.set(url,{url,title,bookmarks});
    }
  }
  const sorted=[...items.values()].sort((a,b)=>b.bookmarks-a.bookmarks);
  console.log(JSON.stringify(sorted,null,2));
});
"
```

取得フィールド:
- `title`: タイトル（HTMLエンティティをデコードすること）
- `url`: リンク先の元記事URL（はてブのエントリーページURLではない）
- `bookmarks`: ブックマーク数

---

## 注目トピック選定

全ソースを横断して以下の基準で15-20件を厳選:
- スコア/ups/ブックマーク数が高い
- コメントが活発で議論が盛り上がっている
- 技術的に興味深い・新規性がある
- 社会的インパクトが大きい
- 複数ソースで同じ話題が出ている場合は統合して取り上げる

固定投稿（pinned/stickied）やメタ投稿（hiring thread、weekly recap等）は除外する。

Redditの注目トピックについてはStep 1のコメント取得コマンドで詳細を取得する。

---

## HTML生成

出力先: `300_News/YYYY-MM-DD_テックニュースまとめ.html`（実行日の日付を使用）

### ページ構成: 全10ページ

HTMLは`@media print`でページ分割可能にしつつ、ブラウザ閲覧時は各ページを`<section class="page">`で区切り、ページ番号を表示する。
各ページ間は太い水平線+ページ番号で視覚的に分離する。

| ページ | 内容 | 記事数目安 |
|---|---|---|
| 1面 | トップ記事（全ソース中最重要の1件を全幅で大きく） | 1 |
| 2面 | テクノロジー / 政策（r/technology、HN等から） | 2-3 |
| 3面 | AI / LLM（r/OpenAI、r/ClaudeAI、r/ChatGPT、r/ClaudeCode、HN等） | 3-4 |
| 4面 | セキュリティ（r/netsec、r/cybersecurity、Aikido、Wiz） | 2-3 |
| 5面 | 開発 / プログラミング（r/programming、HN等） | 2-3 |
| 6面 | ツール / Obsidian（r/ObsidianMD等） | 2-3 |
| 7面 | 日本市場 -- AI / 機械学習（はてブ） | 3-5 |
| 8面 | 日本市場 -- プログラミング / エンジニア（はてブ） | 3-5 |
| 9面 | 日本市場 -- セキュリティ / インフラ（はてブ） | 3-5 |
| 10面 | Hacker News ダイジェスト（HN上位からピックアップ） | 5-8（短信形式） |

記事数が少ないページは隣接ページと統合してよい。逆に話題が豊富なテーマは面を増やしてよい。合計で概ね10ページ前後を目安とする。

### デザイン: クラシック新聞風

**全体レイアウト**
- 白背景（#faf9f6）、黒テキスト（#1a1a1a）
- フォント: 見出しは serif（"Noto Serif JP", "Georgia", serif）、本文は sans-serif（"Noto Sans JP", sans-serif）
- max-width: 1100px、中央寄せ
- 新聞名「TECH TIMES」をマストヘッドとして1面の最上部に大きく表示（セリフ体、letter-spacing広め）
- マストヘッド下に日付を細いラインで区切って表示

**1面（トップ記事）**
- 全ソースを通じて最も重要な記事を1件選び、全幅で大きく表示
- 見出しは大きなセリフ体（2rem以上）
- 画像があれば全幅で表示
- リード文（要約）を太字で表示
- Redditソースの場合、コメントから抜粋した議論も掲載
- ドロップキャップ（先頭文字を大きく）適用

**2面以降（各セクション）**
- ページ上部にページ番号とセクション名を表示（例: 「-- 3面 AI / LLM --」）
- CSS Grid で2カラムレイアウト
- 各記事は縦罫線（border-right: 1px solid #ddd）で区切る
- 見出しはセリフ体（1.2-1.4rem）
- 本文は小さめ（0.9rem）
- 画像は記事幅に合わせて表示

**10面（HNダイジェスト）**
- Hacker News上位記事を短信形式（見出し+1-2行の概要+スコア）で一覧
- 1カラムのコンパクトなリスト形式
- 各項目にHNコメントページへのリンクを付与

**記事の共通構成**
- 見出し（日本語。はてブソースで既に日本語のものはそのまま）
- ソース名を小さく表示（例: 「r/technology」「Hacker News」「はてなブックマーク」「Aikido Blog」）
- スコア情報を控えめに表示（ups / points / bookmarks）
- 本文の日本語要約
- Redditソースの場合: 主要コメント2-4件を引用ブロック風に表示
- ソースへのリンクを記事末に配置

**画像の扱い**
- preview URLがあれば`<img>`タグで埋め込み（`&amp;`は`&`に変換）
- 1面トップ記事: 全幅表示
- 2段組記事: カラム幅に合わせて表示

**罫線・装飾**
- ページ間は太い水平線（border-top: 3px double #1a1a1a）+ ページ番号で区切る
- セクション内の記事間は細い水平線（border-top: 1px solid #ddd）

**フッター**
- 取得日時とソース一覧を記載

---

## 出力ルール

- 全テキストは日本語（技術用語・固有名詞は原文のまま）
- 常体で記述（敬語は使わない）
- 絵文字は使用しない
- $ARGUMENTSでソースの追加・除外指示があれば調整すること
