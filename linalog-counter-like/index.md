---
image: https://chrmlinux.github.io/linaLog/assets/rina-chan.png
---

# linaLog にアクセスカウンターと「いいね」を付けた件

2026年11月27日 15:31

<p><img src="../assets/rina-chan.png" alt="リナちゃん" width="240"></p>

対象ブログ: https://chrmlinux.github.io/linaLog/ （GitHub Pages + Jekyll、テーマは Hacker）

GitHub Pages は静的サイトなので、アクセス計測も「いいね」の保存も、サイト側だけではできない。どちらも外部サービスに保存先を借りる形にした。

## アクセスカウンター: GoatCounter

選んだ理由は、無料で、Cookie を使わず、個人情報を集めない設計だったため。PV と参照元が見られれば十分だった。

### 登録

- https://www.goatcounter.com/signup で登録
- Account name は `linalog` だけを入れる。`linalog.goatcounter.com` と入れると「`.` を含む」というエラーになる（後ろは自動で付く）
- Site domain には `chrmlinux.github.io` を入れた
- 人間確認の欄（「Fill in 9 here」のような指示）には、指示された数字をそのまま入れる

### 組み込み

登録後に表示されるスニペットを、全ページの `<head>` に入る `_includes/head-custom.html` の末尾へ追加した。

```html
<script data-goatcounter="https://linalog.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>
```

集計画面は https://linalog.goatcounter.com （ログインが必要）。

## いいね: Lyket

記事ごとに「いいね」ボタンを置けて、押す側はログイン不要、カウントは Lyket 側に保存される。

### 登録

- https://lyket.dev で登録（GitHub アカウントでも可）
- ダッシュボードで公開 API キー（`pt_` で始まる）を取得する。このキーはページに埋め込む前提の公開キー
- Allowed websites に `chrmlinux.github.io` を設定し、キーが使えるサイトを制限する
- カテゴリは Tech を選んだ

### 組み込み

スクリプトは `_includes/head-custom.html` に静的に置く。

```html
<script src="https://unpkg.com/@lyket/widget@latest/dist/lyket.js?apiKey=（公開キー）"></script>
```

ボタン本体は、各記事の `index.md` の末尾に静的な div として置く。`data-lyket-id` は記事のスラッグで、記事ごとに別のカウントになる。名前空間は `linalog` で統一した。

```html
<div class="like-wrap"><div data-lyket-type="like" data-lyket-template="twitter" data-lyket-namespace="linalog" data-lyket-id="blog-migration" data-lyket-color-icon="#b5e853" data-lyket-color-primary="#b5e853" data-lyket-color-text="#eaeaea" data-lyket-color-secondary="#2b2b2b"></div></div>
```

動作確認は API でもできる（ボタンが作られ、カウントが返る）。

```
curl -H 'Authorization: Bearer （公開キー）' 'https://api.lyket.dev/v1/like-buttons/linalog/blog-migration'
```

### ハマった点

- ボタンの div を JavaScript で後から挿入する方法は動かなかった。ウィジェットは読み込み時に DOM を走査するので、スクリプトも div も静的に置くのが正解だった
- 既定の色は黒背景に濃いグレーで、表示されているのに見えなかった。`data-lyket-color-icon` などでテーマに合わせて明示する必要がある（アイコンと「いいね済み」の色は緑 `#b5e853`、数字は `#eaeaea` にした）
- テンプレート名は種類ごとに決まっている。「いいね」用は Default / Twitter / Chevron / Heart で、`simple` は「拍手」用なので、いいねに指定すると既定の円形デザインに戻る。既定デザインは押すと大きな緑の円になるため、ハート＋数字の `twitter` テンプレートにした
- 「いいね」はトグル式で、もう一度押すと取り消せる

## 結果

- 全74記事とトップページに、アクセス計測（GoatCounter）といいねボタン（Lyket）が入った
- どちらもサイト側にデータを持たないので、記事の Git 管理は今まで通りで変わらない

<div class="like-wrap"><div data-lyket-type="like" data-lyket-template="twitter" data-lyket-namespace="linalog" data-lyket-id="linalog-counter-like" data-lyket-color-icon="#b5e853" data-lyket-color-primary="#b5e853" data-lyket-color-text="#eaeaea" data-lyket-color-secondary="#2b2b2b"></div></div>
