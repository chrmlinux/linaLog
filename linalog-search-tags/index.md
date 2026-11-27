---
image: https://chrmlinux.github.io/linaLog/assets/rina-chan.png
---

# linaLog に検索とタグを付けた件

2026年11月27日 21:10
<p class="article-tags">タグ: <a href="..?tag=GitHub">GitHub</a> <a href="..?tag=%E3%83%96%E3%83%AD%E3%82%B0">ブログ</a></p>

<p><img src="../assets/rina-chan.png" alt="リナちゃん" width="240"></p>

対象ブログ: https://chrmlinux.github.io/linaLog/ （GitHub Pages + Jekyll、テーマは Hacker）

記事が 75 本まで増えて、一覧を目で追うだけでは探しにくくなった。静的サイトなので検索もタグも標準では無い。どちらもページ側の JavaScript だけで付けた。

## 検索

トップページに検索欄を置いた。入力した語で、タイトル・タグ・日付を対象にその場で絞り込む。並べ替えとページ送りは、絞り込みと併用できる。

検索インデックスのような別ファイルは作っていない。一覧の各項目が持っているタイトルとタグと日付を、そのまま検索対象にしている。

## タグ

タグは一から考えず、移行元の elchika で各記事に付けていたものを、そのまま引き継いだ（71 記事分）。当時付けたタグが一番正確だからである。elchika に無い新規の記事には、同系統のタグを付けた。

- 各記事のタイトル下にタグを表示した。押すとトップページへ戻り、一覧がそのタグで絞り込まれる
- トップページにも、件数付きのタグボタンを並べた

## タグ表示のスイッチ

タグボタンが全部並ぶと一覧が長くなるので、表示／非表示の切り替えボタンを付けた。既定は非表示で、設定はブラウザに保存される。

## まとめ

- 検索もタグも、静的サイトのまま HTML と JavaScript だけで実現した
- タグは新規に考えず、elchika 本家のものを引き継いだ

<div class="like-wrap"><div data-lyket-type="like" data-lyket-template="twitter" data-lyket-namespace="linalog" data-lyket-id="linalog-search-tags" data-lyket-color-icon="#b5e853" data-lyket-color-primary="#b5e853" data-lyket-color-text="#eaeaea" data-lyket-color-secondary="#2b2b2b"></div></div>
