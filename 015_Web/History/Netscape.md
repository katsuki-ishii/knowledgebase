---
base: "[[Web.History.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Web
作成日時: 2026-02-15T12:00:00
tags:
  - Netscape
aliases:
  - Netscape
  - netscape
  - Netscape Navigator
---

# [[Netscape]]

---

## 1. 前提：Webの初期状態

1990年代前半のWebは、研究者向けの文書共有システムに近い存在だった。

- テキスト中心
- 静的ページのみ
- セキュリティほぼなし
- 状態管理なし
- 商用利用ほぼなし

この段階では、Webは「読むもの」であって「使うもの」ではなかった。

---

## 2. Mosaicから[[Netscape]]へ

### 2.1 NCSA Mosaic

1993年、イリノイ大学で開発。

特徴：

- GUIベース
- 画像をページ内表示
- 一般人でも利用可能

ただし：

- 大学プロジェクト
- 商用戦略なし
- サポート体制なし

### 2.2 [[Netscape]]の誕生

Mosaic開発者のマーク・アンドリーセンらが1994年に[[Netscape]]を設立。

Mosaicの思想を引き継ぎ、商用プロダクトとして再設計。

図：

```
Mosaic（大学研究）
    ↓
Netscape Navigator（商用ブラウザ）
```

---

## 3. [[Netscape]]が生み出した主要技術

### 3.1 JavaScript（1995）

- ブレンダン・アイクが開発
- 当初はLiveScript
- 約10日で原型完成

目的：Webページを動的にすること

影響：

- フォーム入力チェック
- 動的UI
- 後のSPA基盤

現在：JavaScript → ECMAScriptとして標準化

---

### 3.2 [[Cookie|クッキー]]（1994）

課題：HTTPはステートレス（状態を持たない）

解決：ブラウザに小さなデータを保存

用途：

- ログイン維持
- セッション管理
- カート保持

図：

```
サーバー → Set-Cookie
ブラウザ保存
次回リクエスト → Cookie送信
```

---

### 3.3 SSL（Secure Sockets Layer）

目的：通信の暗号化

背景：[[HTMLのclass, id, style|ID]]やクレジット情報が平文送信されていた

進化：SSL → TLS

現在のHTTPSの基盤。

図：

```
通常HTTP
クライアント → データ（平文）→ サーバー

SSL/TLS
クライアント → 暗号化データ → サーバー
```

---

### 3.4 Same-Origin Policy（1995頃）

JavaScript導入により発生したセキュリティ課題への対応。

定義：同一オリジン（プロトコル・ドメイン・ポートが一致）以外へのアクセスを制限。

なければ：他サイトの情報を読み取り可能になり、Webは崩壊していた。

---

### 3.5 DOMの原型

HTMLを「文書」ではなく「オブジェクト」として扱う発想。

例：

- window
- document
- forms

これが後にDOMとして標準化。

VueやReactも最終的にはDOMを操作している。

---

### 3.6 プラグインモデル（NPAPI）

外部機能をブラウザに拡張可能にした。

例：

- Flash
- Java Applet

拡張性の概念を確立。

---

### 3.7 Webサーバー事業

[[Netscape]]はブラウザだけでなく、企業向けWebサーバーも提供。

思想：「Webを企業インフラにする」

図：

```
ブラウザ（クライアント）
        ↓
Netscape Server（サーバー）
```

Webエコシステム全体を構築しようとしていた。

---

## 4. ブラウザ戦争

### 4.1 Microsoftの参入

- Spyglass Mosaicをライセンス
- Internet Explorerを開発
- Windowsに無料同梱

戦略差：

```
Netscape：ブラウザ販売
Microsoft：OS防衛のため無料配布
```

### 4.2 結果

- IEが市場を独占
- [[Netscape]]衰退
- 反トラスト法問題へ発展

---

## 5. [[Netscape]]の最大の遺産

[[Netscape]]は会社としては敗北したが、技術思想は残った。

流れ：

```
Netscape
    ↓（コード公開）
Mozilla
    ↓
Firefox
    ↓
Chromeにも影響
```

また、以下が現在も存続：

- JavaScript
- [[Cookie|クッキー]]
- HTTPS
- DOM
- セキュリティモデル

---

## 6. 本質的意義

[[Netscape]]が行ったことは単なるブラウザ開発ではない。

文書共有システムだったWebを、

「安全で、状態を持ち、プログラムが動く実行環境」

へ変化させた。

図：

```
旧Web（静的文書）
    ↓
Netscapeによる拡張
    ↓
現代Web（アプリケーション基盤）
```

---

## 7. まとめ

[[Netscape]]は以下を同時に生み出した。

- 動的実行環境（JavaScript）
- 状態管理（[[Cookie]]）
- 暗号通信（SSL）
- セキュリティ制御（SOP）
- DOMの基礎
- 拡張モデル
- 商用Web基盤

Webの設計図を描いた存在と言える。

会社は消滅したが、思想は今もWebに残っている。
