---
layout: default
title: linaLog
image: https://chrmlinux.github.io/linaLog/assets/rina-chan.png
---

# linaLog

<p align="center">
  <img src="assets/rina-chan.png" alt="リナちゃん" width="360">
</p>

製作記事置き場です。Markdown のまま読めます。

Project write-ups, in Markdown.

<p>X: <a href="https://x.com/chrmlinux03">@chrmlinux03</a></p>

<p><a href="feed.xml">RSS フィード</a></p>

## 記事 / Articles

<style>
#sortBtn, #tagToggle {
  background: #151515;
  color: #b5e853;
  border: 1px solid #b5e853;
  padding: 4px 14px;
  font-family: Monaco, "Bitstream Vera Sans Mono", "Lucida Console", monospace;
  font-size: 13px;
  cursor: pointer;
  margin-bottom: 10px;
}
#sortBtn:hover, #tagToggle:hover { background: #b5e853; color: #151515; }
#pager { margin-top: 12px; }
.pager-btn {
  background: #151515;
  color: #b5e853;
  border: 1px solid #b5e853;
  padding: 4px 10px;
  font-family: Monaco, "Bitstream Vera Sans Mono", "Lucida Console", monospace;
  font-size: 13px;
  cursor: pointer;
  margin-right: 6px;
}
.pager-btn:hover { background: #b5e853; color: #151515; }
.pager-btn.current { background: #b5e853; color: #151515; }
#searchBox { background: #151515; color: #eaeaea; border: 1px solid #b5e853; padding: 5px 10px; font-family: Monaco, "Bitstream Vera Sans Mono", "Lucida Console", monospace; font-size: 14px; width: 100%; max-width: 420px; }
#tagBar { margin-bottom: 4px; }
.tagbtn { background: #151515; color: #b5e853; border: 1px solid #b5e853; padding: 2px 10px; font-family: Monaco, "Bitstream Vera Sans Mono", "Lucida Console", monospace; font-size: 12px; cursor: pointer; margin: 0 6px 6px 0; border-radius: 10px; }
.tagbtn:hover { background: #b5e853; color: #151515; }
.tagbtn.active { background: #b5e853; color: #151515; }
#articleList .tag { display: inline-block; border: 1px solid #666; color: #aaa; padding: 0 7px; margin-left: 6px; font-size: 11px; border-radius: 9px; text-decoration: none; }
#articleList .tag:hover { border-color: #b5e853; color: #b5e853; }
#resultInfo { color: #aaa; font-size: 13px; }
body.tags-off #tagBar { display: none; }
body.tags-off #articleList .tags { display: none; }
.thumb {
  width: 78px;
  height: 44px;
  object-fit: cover;
  vertical-align: middle;
  margin-right: 10px;
}
</style>

<p><input id="searchBox" type="search" placeholder="検索（タイトル・タグ・日付）" aria-label="記事を検索"></p>
<div id="tagBar"></div>
<script>try { if (localStorage.getItem('linalog-tags-on') !== '1') document.body.classList.add('tags-off'); } catch (e) { document.body.classList.add('tags-off'); }</script>
<p><button id="sortBtn" type="button">古い順に並べ替える</button> <button id="tagToggle" type="button">タグを表示</button></p>
<p id="resultInfo"></p>

<ul id="articleList">
  <li data-tags="GitHub|ブログ" data-date="2026-11-27 21:10"><img class="thumb" src="assets/rina-chan.png" alt="">2026-11-27 21:10 ― <a href="linalog-search-tags/">linaLog に検索とタグを付けた件</a> <span class="tags"><a class="tag" href="?tag=GitHub">GitHub</a> <a class="tag" href="?tag=%E3%83%96%E3%83%AD%E3%82%B0">ブログ</a></span></li>
  <li data-tags="GitHub|ブログ" data-date="2026-11-27 15:31"><img class="thumb" src="assets/rina-chan.png" alt="">2026-11-27 15:31 ― <a href="linalog-counter-like/">linaLog にアクセスカウンターと「いいね」を付けた件</a> <span class="tags"><a class="tag" href="?tag=GitHub">GitHub</a> <a class="tag" href="?tag=%E3%83%96%E3%83%AD%E3%82%B0">ブログ</a></span></li>
  <li data-tags="GitHub|ブログ" data-date="2026-11-27 13:44"><img class="thumb" src="assets/rina-chan.png" alt="">2026-11-27 13:44 ― <a href="blog-migration/">elchika から GitHub Pages へ、ブログ71記事を引っ越した手順</a> <span class="tags"><a class="tag" href="?tag=GitHub">GitHub</a> <a class="tag" href="?tag=%E3%83%96%E3%83%AD%E3%82%B0">ブログ</a></span></li>
  <li data-tags="linux|Arduino|Unix" data-date="2026-11-26 20:17"><img class="thumb" src="assets/rina-chan.png" alt="">2026-11-26 20:17 ― <a href="linax-uno/">linaX(uno) ― Arduino Uno（ATmega328P）に載せた Unix 系 OS</a> <span class="tags"><a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=Unix">Unix</a></span></li>
  <li data-tags="Junction|Windows11" data-date="2026-11-23 13:54"><img class="thumb" src="assets/rina-chan.png" alt="">2026-11-23 13:54 ― <a href="windows-disk/">[Junction] Windows の C ドライブを空ける話 [目から鱗]</a> <span class="tags"><a class="tag" href="?tag=Junction">Junction</a> <a class="tag" href="?tag=Windows11">Windows11</a></span></li>
  <li data-tags="linux|ESP32|CYD|LinaX" data-date="2026-11-21 21:41"><img class="thumb" src="assets/rina-chan.png" alt="">2026-11-21 21:41 ― <a href="linax-cyd/">LinaX(cyd) ― PSRAM なしの ESP32（CYD）に載せた Linux 6.11</a> <span class="tags"><a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=ESP32">ESP32</a> <a class="tag" href="?tag=CYD">CYD</a> <a class="tag" href="?tag=LinaX">LinaX</a></span></li>
  <li data-tags="CYD|ESP32|LinaX|linux" data-date="2026-11-21 14:52"><img class="thumb" src="cyd-linux611/images/img01.png" alt="">2026-11-21 14:52 ― <a href="cyd-linux611/">[CYD]内蔵 DRAM 320KB で Linux6.11を動かしたにょ</a> <span class="tags"><a class="tag" href="?tag=CYD">CYD</a> <a class="tag" href="?tag=ESP32">ESP32</a> <a class="tag" href="?tag=LinaX">LinaX</a> <a class="tag" href="?tag=linux">linux</a></span></li>
  <li data-tags="CYD|ESP32|x11" data-date="2026-11-19 10:46"><img class="thumb" src="tinyx11-cyd/images/img01.jpg" alt="">2026-11-19 10:46 ― <a href="tinyx11-cyd/">[cyd] $7の液晶付きマイコンに x11 を載せてみた[3分ハッキング別冊]</a> <span class="tags"><a class="tag" href="?tag=CYD">CYD</a> <a class="tag" href="?tag=ESP32">ESP32</a> <a class="tag" href="?tag=x11">x11</a></span></li>
  <li data-tags="KB2|M5Stack|SSH" data-date="2026-11-18 11:01"><img class="thumb" src="cardkb2-ssh/images/img01.jpg" alt="">2026-11-18 11:01 ― <a href="cardkb2-ssh/">[M5Stack]CardKB2に SSH クライアント実装した[3分ハッキング付録]</a> <span class="tags"><a class="tag" href="?tag=KB2">KB2</a> <a class="tag" href="?tag=M5Stack">M5Stack</a> <a class="tag" href="?tag=SSH">SSH</a></span></li>
  <li data-tags="CARDKB2|CNT+C|M5Stack" data-date="2026-11-16 16:49"><img class="thumb" src="cardkb2-cntc/images/img01.jpg" alt="">2026-11-16 16:49 ― <a href="cardkb2-cntc/">[M5Stack]CardKB2にCNT+Cを実装する[3分ハッキング最終回]</a> <span class="tags"><a class="tag" href="?tag=CARDKB2">CARDKB2</a> <a class="tag" href="?tag=CNT%2BC">CNT+C</a> <a class="tag" href="?tag=M5Stack">M5Stack</a></span></li>
  <li data-tags="FEP|IME|SPRESENSE|日本語入力|自作OS" data-date="2026-09-19 13:51"><img class="thumb" src="spresense-nihongo/images/img01.jpg" alt="">2026-09-19 13:51 ― <a href="spresense-nihongo/">[#SPRESENSE 2026]簡単だけど奥が深いにょ[日本語入力]</a> <span class="tags"><a class="tag" href="?tag=FEP">FEP</a> <a class="tag" href="?tag=IME">IME</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E6%97%A5%E6%9C%AC%E8%AA%9E%E5%85%A5%E5%8A%9B">日本語入力</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="CH32V003|I2C|ILI9341|SPI|cardputer|sop8" data-date="2026-09-16 11:06"><img class="thumb" src="ch32v003-cardputer-lcd/images/img01.jpg" alt="">2026-09-16 11:06 ― <a href="ch32v003-cardputer-lcd/">[ch32v003]ほぼどんなマイコンでも液晶付けられるにょ[cardputer]</a> <span class="tags"><a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=I2C">I2C</a> <a class="tag" href="?tag=ILI9341">ILI9341</a> <a class="tag" href="?tag=SPI">SPI</a> <a class="tag" href="?tag=cardputer">cardputer</a> <a class="tag" href="?tag=sop8">sop8</a></span></li>
  <li data-tags="Arduino|LIVEC|M5Stack|VM|linux|stickS3|tab5|自作OS" data-date="2026-09-12 13:06"><img class="thumb" src="spresense-linax-port/images/img01.jpg" alt="">2026-09-12 13:06 ― <a href="spresense-linax-port/">[#SPRESENSE 2026]移植してみたにょ[linaX]</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=LIVEC">LIVEC</a> <a class="tag" href="?tag=M5Stack">M5Stack</a> <a class="tag" href="?tag=VM">VM</a> <a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=stickS3">stickS3</a> <a class="tag" href="?tag=tab5">tab5</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="Arduino|LIVEC|SPRESENSE|VM|linux|自作OS" data-date="2026-09-08 19:25"><img class="thumb" src="spresense-linax/images/img01.jpg" alt="">2026-09-08 19:25 ― <a href="spresense-linax/">[#SPRESENSE 2026]最終奥義OS作ってみたにょ[linaX]</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=LIVEC">LIVEC</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=VM">VM</a> <a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="Arduino|LIVEC|SPRESENSE|VM|自作OS" data-date="2026-09-07 17:13"><img class="thumb" src="lindos/images/img01.jpg" alt="">2026-09-07 17:13 ― <a href="lindos/">[#SPRESENSE 2026]OS作ってみたにょ[LinDOS]</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=LIVEC">LIVEC</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=VM">VM</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="Arduino|FORTRAN|LIVEFO|SPRESENSE|VM|自作OS" data-date="2026-09-03 13:06"><img class="thumb" src="livefo/images/img01.jpg" alt="">2026-09-03 13:06 ― <a href="livefo/">[#SPRESENSE 2026] FORTRAN(VM)を作ったにょ [LiveFO編]</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=FORTRAN">FORTRAN</a> <a class="tag" href="?tag=LIVEFO">LIVEFO</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=VM">VM</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="Arduino|COBOL|LIVECO|SPRESENSE|VM|自作OS" data-date="2026-09-02 13:20"><img class="thumb" src="liveco/images/img01.jpg" alt="">2026-09-02 13:20 ― <a href="liveco/">[#SPRESENSE 2026] COBOLインタプリタ(VM)を作ったにょ [LiveCO編]</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=COBOL">COBOL</a> <a class="tag" href="?tag=LIVECO">LIVECO</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=VM">VM</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="Arduino|BASIC|LIVEB|LIVEP|Python|SPRESENSE|VM|自作OS" data-date="2026-08-31 16:56"><img class="thumb" src="liveb-livep/images/img01.jpg" alt="">2026-08-31 16:56 ― <a href="liveb-livep/">[#SPRESENSE 2026]Cインタプリタ(VM)を作ったにょ[LiveB/LiveP編]</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=BASIC">BASIC</a> <a class="tag" href="?tag=LIVEB">LIVEB</a> <a class="tag" href="?tag=LIVEP">LIVEP</a> <a class="tag" href="?tag=Python">Python</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=VM">VM</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="Arduino|C言語|LIVEC|SPRESENSE|VM|自作OS" data-date="2026-08-26 13:28"><img class="thumb" src="livec-editor/images/img01.jpg" alt="">2026-08-26 13:28 ― <a href="livec-editor/">[#SPRESENSE 2026]Cインタプリタ(VM)を作ったにょ[エディタ編]</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=C%E8%A8%80%E8%AA%9E">C言語</a> <a class="tag" href="?tag=LIVEC">LIVEC</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=VM">VM</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="Arduino|C言語|LIVEC|SPRESENSE|VM|ch9350|キーボード|マウス|自作OS" data-date="2026-08-26 13:10"><img class="thumb" src="livec-ch9350/images/img01.jpg" alt="">2026-08-26 13:10 ― <a href="livec-ch9350/">[#SPRESENSE 2026]Cインタプリタ(VM)を作ったにょ[ch9350編]</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=C%E8%A8%80%E8%AA%9E">C言語</a> <a class="tag" href="?tag=LIVEC">LIVEC</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=VM">VM</a> <a class="tag" href="?tag=ch9350">ch9350</a> <a class="tag" href="?tag=%E3%82%AD%E3%83%BC%E3%83%9C%E3%83%BC%E3%83%89">キーボード</a> <a class="tag" href="?tag=%E3%83%9E%E3%82%A6%E3%82%B9">マウス</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="Arduino|C言語|LIVEC|SPRESENSE|VM|自作OS" data-date="2026-08-26 11:44"><img class="thumb" src="livec/images/img01.jpg" alt="">2026-08-26 11:44 ― <a href="livec/">[#SPRESENSE 2026]Cインタプリタ(VM)を作ったにょ[LiveC編]</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=C%E8%A8%80%E8%AA%9E">C言語</a> <a class="tag" href="?tag=LIVEC">LIVEC</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=VM">VM</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></span></li>
  <li data-tags="Audio|CVBS|USBDAC" data-date="2026-06-18 11:41"><img class="thumb" src="usbdac-cvbs-small/images/img01.png" alt="">2026-06-18 11:41 ― <a href="usbdac-cvbs-small/">[USBDAC]基板が小さくなったにょ[CVBS]</a> <span class="tags"><a class="tag" href="?tag=Audio">Audio</a> <a class="tag" href="?tag=CVBS">CVBS</a> <a class="tag" href="?tag=USBDAC">USBDAC</a></span></li>
  <li data-tags="Audio|CVBS|HIRES" data-date="2026-06-17 13:27"><img class="thumb" src="usbdac-ntsc/images/img01.png" alt="">2026-06-17 13:27 ― <a href="usbdac-ntsc/">[USBDAC]イヤホンジャックでNTSC映像を出してみたにょ[CVBS]</a> <span class="tags"><a class="tag" href="?tag=Audio">Audio</a> <a class="tag" href="?tag=CVBS">CVBS</a> <a class="tag" href="?tag=HIRES">HIRES</a></span></li>
  <li data-tags="Audio|CVBS|HIRES|SPRESENSE|misakifont|漢字" data-date="2026-06-16 16:51"><img class="thumb" src="spresense-audiohack-kanji/images/img01.png" alt="">2026-06-16 16:51 ― <a href="spresense-audiohack-kanji/">[SPRESENSE 2026]AudioHack漢字に対応したにょ[CVBS]</a> <span class="tags"><a class="tag" href="?tag=Audio">Audio</a> <a class="tag" href="?tag=CVBS">CVBS</a> <a class="tag" href="?tag=HIRES">HIRES</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=misakifont">misakifont</a> <a class="tag" href="?tag=%E6%BC%A2%E5%AD%97">漢字</a></span></li>
  <li data-tags="Audio|CVBS|HIRES|SPRESENSE" data-date="2026-06-15 16:52"><img class="thumb" src="spresense-audiojack-hack/images/img01.png" alt="">2026-06-15 16:52 ― <a href="spresense-audiojack-hack/">[SPRESENSE 2026]AudioJackをHackしたにょ[CVBS]</a> <span class="tags"><a class="tag" href="?tag=Audio">Audio</a> <a class="tag" href="?tag=CVBS">CVBS</a> <a class="tag" href="?tag=HIRES">HIRES</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a></span></li>
  <li data-tags="CVBS|SPRESENSE" data-date="2026-06-10 12:22"><img class="thumb" src="spresense-tv-cvbs/images/img01.png" alt="">2026-06-10 12:22 ― <a href="spresense-tv-cvbs/">[SPRESENSE 2026]TVに出力したにょ[CVBS]</a> <span class="tags"><a class="tag" href="?tag=CVBS">CVBS</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a></span></li>
  <li data-tags="CH552E|CH552G|MSOP10|SOP10|USB-Audio|linaAudio" data-date="2026-06-05 10:50"><img class="thumb" src="linaaudio-2/images/img01.png" alt="">2026-06-05 10:50 ― <a href="linaaudio-2/">【linaAudio】45円CPUで作るUSB-Audioだにょ②【動作編】</a> <span class="tags"><a class="tag" href="?tag=CH552E">CH552E</a> <a class="tag" href="?tag=CH552G">CH552G</a> <a class="tag" href="?tag=MSOP10">MSOP10</a> <a class="tag" href="?tag=SOP10">SOP10</a> <a class="tag" href="?tag=USB-Audio">USB-Audio</a> <a class="tag" href="?tag=linaAudio">linaAudio</a></span></li>
  <li data-tags="CH32V003|USB-Audio|linaAudio|linaduino|sop8" data-date="2026-05-27 10:18"><img class="thumb" src="linaaudio-1/images/img01.png" alt="">2026-05-27 10:18 ― <a href="linaaudio-1/">【linaAudio】25円CPUで作るUSB-Audioだにょ①【認識編】</a> <span class="tags"><a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=USB-Audio">USB-Audio</a> <a class="tag" href="?tag=linaAudio">linaAudio</a> <a class="tag" href="?tag=linaduino">linaduino</a> <a class="tag" href="?tag=sop8">sop8</a></span></li>
  <li data-tags="IRFi|WiFi" data-date="2026-05-20 13:58"><img class="thumb" src="irfi/images/img01.png" alt="">2026-05-20 13:58 ― <a href="irfi/">【IRFi】え？IRDAでWiFi?【無線光通信】</a> <span class="tags"><a class="tag" href="?tag=IRFi">IRFi</a> <a class="tag" href="?tag=WiFi">WiFi</a></span></li>
  <li data-tags="CH32V003|Grove|USB-HID|linaduino|sop8" data-date="2026-05-15 08:31"><img class="thumb" src="linaduino-usbgrove/images/img01.jpg" alt="">2026-05-15 08:31 ― <a href="linaduino-usbgrove/">【linaDuino】25円で作るUSBGroveブリッジだにょ【USBGrove】</a> <span class="tags"><a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=Grove">Grove</a> <a class="tag" href="?tag=USB-HID">USB-HID</a> <a class="tag" href="?tag=linaduino">linaduino</a> <a class="tag" href="?tag=sop8">sop8</a></span></li>
  <li data-tags="CH32V003|ILI9341|USB-HID|linaduino|sop8" data-date="2026-05-14 20:30"><img class="thumb" src="linaduino-usbmonitor/images/img01.jpg" alt="">2026-05-14 20:30 ― <a href="linaduino-usbmonitor/">【linaDuino】25円で作るUSBモニターだにょ【USBMonitor】</a> <span class="tags"><a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=ILI9341">ILI9341</a> <a class="tag" href="?tag=USB-HID">USB-HID</a> <a class="tag" href="?tag=linaduino">linaduino</a> <a class="tag" href="?tag=sop8">sop8</a></span></li>
  <li data-tags="CH32V003|USB-HID|gamepad|linaduino|sop8" data-date="2026-05-14 14:27"><img class="thumb" src="linaduino-4btnpad/images/img01.jpg" alt="">2026-05-14 14:27 ― <a href="linaduino-4btnpad/">【linaDuino】25円で作るUSB-HIDだにょ【4btnPad】</a> <span class="tags"><a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=USB-HID">USB-HID</a> <a class="tag" href="?tag=gamepad">gamepad</a> <a class="tag" href="?tag=linaduino">linaduino</a> <a class="tag" href="?tag=sop8">sop8</a></span></li>
  <li data-tags="USB" data-date="2026-05-10 23:13"><img class="thumb" src="usb-a-probability/images/img01.jpg" alt="">2026-05-10 23:13 ― <a href="usb-a-probability/">【USB-Aの成功率】50%のはずが75%の確率で間違える理由【なぜなぜ？】</a> <span class="tags"><a class="tag" href="?tag=USB">USB</a></span></li>
  <li data-tags="CH32V003|j4m6|linaduino|sop8" data-date="2026-05-07 14:09"><img class="thumb" src="linaduino/images/img01.png" alt="">2026-05-07 14:09 ― <a href="linaduino/">【linaDuino】リナディーノ(てじなーにゃのアクセントで)【世界最小Arduino互換】</a> <span class="tags"><a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=j4m6">j4m6</a> <a class="tag" href="?tag=linaduino">linaduino</a> <a class="tag" href="?tag=sop8">sop8</a></span></li>
  <li data-tags="64bit|CH32V003|RISC-V|クラスタ|リナゴスティーニ" data-date="2026-04-08 11:21"><img class="thumb" src="linagostini-1/images/img01.png" alt="">2026-04-08 11:21 ― <a href="linagostini-1/">【リナゴスティーニ①創刊号】64bitI2Cクラスタを27円RISC-Vで作るにょ【思想】</a> <span class="tags"><a class="tag" href="?tag=64bit">64bit</a> <a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=RISC-V">RISC-V</a> <a class="tag" href="?tag=%E3%82%AF%E3%83%A9%E3%82%B9%E3%82%BF">クラスタ</a> <a class="tag" href="?tag=%E3%83%AA%E3%83%8A%E3%82%B4%E3%82%B9%E3%83%86%E3%82%A3%E3%83%BC%E3%83%8B">リナゴスティーニ</a></span></li>
  <li data-tags="40円マイコン|Arduino-IDE|CH32RTA|CH32V003" data-date="2026-04-01 11:06"><img class="thumb" src="dual-ch32v003/images/img01.png" alt="">2026-04-01 11:06 ― <a href="dual-ch32v003/">【dualCH32V003】92ByteでPD6から高速矩形波を出してみたにょ【40円マイコン】</a> <span class="tags"><a class="tag" href="?tag=40%E5%86%86%E3%83%9E%E3%82%A4%E3%82%B3%E3%83%B3">40円マイコン</a> <a class="tag" href="?tag=Arduino-IDE">Arduino-IDE</a> <a class="tag" href="?tag=CH32RTA">CH32RTA</a> <a class="tag" href="?tag=CH32V003">CH32V003</a></span></li>
  <li data-tags="290円マイコン|CH32V003|UIAPduino" data-date="2026-03-25 12:41"><img class="thumb" src="uiapduino-video/images/img01.png" alt="">2026-03-25 12:41 ― <a href="uiapduino-video/">【UIAPduino】VIDEO信号を出したにょ①【290円マイコン基板】</a> <span class="tags"><a class="tag" href="?tag=290%E5%86%86%E3%83%9E%E3%82%A4%E3%82%B3%E3%83%B3">290円マイコン</a> <a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=UIAPduino">UIAPduino</a></span></li>
  <li data-tags="290円マイコン|CH32V003|UIAPduino" data-date="2026-03-18 13:17"><img class="thumb" src="uiapduino-printf/images/img01.png" alt="">2026-03-18 13:17 ― <a href="uiapduino-printf/">【UIAPduino】printf を実装するにょ【290円マイコン基板】</a> <span class="tags"><a class="tag" href="?tag=290%E5%86%86%E3%83%9E%E3%82%A4%E3%82%B3%E3%83%B3">290円マイコン</a> <a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=UIAPduino">UIAPduino</a></span></li>
  <li data-tags="290円マイコン|CH32V003|UIAPduino" data-date="2026-03-07 17:56"><img class="thumb" src="uiapduino/images/img01.png" alt="">2026-03-07 17:56 ― <a href="uiapduino/">【UIAPduino】普通に使えるっぽいにょ【290円マイコン基板】</a> <span class="tags"><a class="tag" href="?tag=290%E5%86%86%E3%83%9E%E3%82%A4%E3%82%B3%E3%83%B3">290円マイコン</a> <a class="tag" href="?tag=CH32V003">CH32V003</a> <a class="tag" href="?tag=UIAPduino">UIAPduino</a></span></li>
  <li data-tags="Arduino|linux|unoQ" data-date="2026-03-05 18:23"><img class="thumb" src="unoq-matrix-ip/images/img01.png" alt="">2026-03-05 18:23 ― <a href="unoq-matrix-ip/">【unoQ】起動時にIPアドレスと温度をMatrixに表示するにょ【小さなLinux機】</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=unoQ">unoQ</a></span></li>
  <li data-tags="Arduino|linaQ|linux|unoQ" data-date="2026-03-02 06:44"><img class="thumb" src="unoq-freedos/images/img01.png" alt="">2026-03-02 06:44 ― <a href="unoq-freedos/">【unoQ】tiny386でFreeDos動かしたにょ【小さなLinux機】</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=linaQ">linaQ</a> <a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=unoQ">unoQ</a></span></li>
  <li data-tags="Arduino|linux|unoQ" data-date="2026-02-25 10:06"><img class="thumb" src="unoq-windows/images/img01.png" alt="">2026-02-25 10:06 ― <a href="unoq-windows/">【unoQ】Windowsを動かしたにょ【小さなLinux機】</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=unoQ">unoQ</a></span></li>
  <li data-tags="Arduino|linux|unoQ" data-date="2026-02-18 09:42"><img class="thumb" src="unoq-ros2/images/img01.png" alt="">2026-02-18 09:42 ― <a href="unoq-ros2/">【unoQ】ROS2を無理やり入れたにょ【小さなLinux機】</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=unoQ">unoQ</a></span></li>
  <li data-tags="Arduino|linux|unoQ" data-date="2026-02-11 16:00"><img class="thumb" src="unoq-x11-lcd/images/img01.png" alt="">2026-02-11 16:00 ― <a href="unoq-x11-lcd/">【unoQ】QRB側X11をSTM32のLCDに表示出来たにょ【小さなLinux機】</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=unoQ">unoQ</a></span></li>
  <li data-tags="Arduino|linux|unoQ" data-date="2026-02-09 17:57"><img class="thumb" src="unoq-qrb-stm32/images/img01.png" alt="">2026-02-09 17:57 ― <a href="unoq-qrb-stm32/">【unoQ】さっそく簡単にQRBとSTM32U5通信させたにょ【小さなLinux機】</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=unoQ">unoQ</a></span></li>
  <li data-tags="Arduino|linux|unoQ" data-date="2026-02-06 10:54"><img class="thumb" src="unoq-setup/images/img01.jpg" alt="">2026-02-06 10:54 ― <a href="unoq-setup/">【unoQ】Arduino UNO Q 簡単初期セットアップ手順【小さなLinux機】</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=linux">linux</a> <a class="tag" href="?tag=unoQ">unoQ</a></span></li>
  <li data-tags="GNSS|GPS|SDカード|SPRESENSE|VGA|WiFi" data-date="2026-01-28 01:47"><img class="thumb" src="spresense-sprepad/images/img01.png" alt="">2026-01-28 01:47 ― <a href="spresense-sprepad/">【SPRESENSE2025】VGA液晶を搭載しGPS地図を表示したにょ【sprePAD?】</a> <span class="tags"><a class="tag" href="?tag=GNSS">GNSS</a> <a class="tag" href="?tag=GPS">GPS</a> <a class="tag" href="?tag=SD%E3%82%AB%E3%83%BC%E3%83%89">SDカード</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=VGA">VGA</a> <a class="tag" href="?tag=WiFi">WiFi</a></span></li>
  <li data-tags="NNC|SPRESENSE|YOLO|dnnRt|ロボット|追従エンジン" data-date="2026-01-20 18:59"><img class="thumb" src="spresense-dnn/images/img01.jpg" alt="">2026-01-20 18:59 ― <a href="spresense-dnn/">【SPRESENSE2025】dnnPlathomeを作ったょ【YOLOっぽい】</a> <span class="tags"><a class="tag" href="?tag=NNC">NNC</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=YOLO">YOLO</a> <a class="tag" href="?tag=dnnRt">dnnRt</a> <a class="tag" href="?tag=%E3%83%AD%E3%83%9C%E3%83%83%E3%83%88">ロボット</a> <a class="tag" href="?tag=%E8%BF%BD%E5%BE%93%E3%82%A8%E3%83%B3%E3%82%B8%E3%83%B3">追従エンジン</a></span></li>
  <li data-tags="I2C|RDA5807|SPRESENSE" data-date="2026-01-09 16:25"><img class="thumb" src="spresense-radio/images/img01.png" alt="">2026-01-09 16:25 ― <a href="spresense-radio/">【SPRESENSE2025】spresense拡張基板に搭載できるラジオモジュールを作ったよっ</a> <span class="tags"><a class="tag" href="?tag=I2C">I2C</a> <a class="tag" href="?tag=RDA5807">RDA5807</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a></span></li>
  <li data-tags="Arduino|ESPNOW|SPRESENSE|テオヤンセン" data-date="2025-01-31 20:05"><img class="thumb" src="spresense-jansen/images/img01.jpg" alt="">2025-01-31 20:05 ― <a href="spresense-jansen/">【SPRESENSE2024】動くテオ・ヤンセン機構【eNord通信】</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=ESPNOW">ESPNOW</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E3%83%86%E3%82%AA%E3%83%A4%E3%83%B3%E3%82%BB%E3%83%B3">テオヤンセン</a></span></li>
  <li data-tags="3Dプリンタ|Arduino|SPRESENSE|mp3|長岡式" data-date="2025-01-30 17:09"><img class="thumb" src="spresense-matrix-speaker/images/img01.jpg" alt="">2025-01-30 17:09 ― <a href="spresense-matrix-speaker/">【SPRESENSE2024】3Dプリンターで長岡式マトリクススピーカー【夢のオーディオ体験】</a> <span class="tags"><a class="tag" href="?tag=3D%E3%83%97%E3%83%AA%E3%83%B3%E3%82%BF">3Dプリンタ</a> <a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=mp3">mp3</a> <a class="tag" href="?tag=%E9%95%B7%E5%B2%A1%E5%BC%8F">長岡式</a></span></li>
  <li data-tags="GNSS|LoRa|SPRESENSE" data-date="2025-01-29 16:24"><img class="thumb" src="spresense-compass/images/img01.jpg" alt="">2025-01-29 16:24 ― <a href="spresense-compass/">【SPRESENSE2024】今年の恵方巻きの方向は？【サテライトコンパス】</a> <span class="tags"><a class="tag" href="?tag=GNSS">GNSS</a> <a class="tag" href="?tag=LoRa">LoRa</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a></span></li>
  <li data-tags="SPRESENSE|タッチパネル|マルチコア|マルチスレッド|マンデルブロ集合|リアルタイム" data-date="2025-01-28 23:13"><img class="thumb" src="spresense-mandelbrot/images/img01.jpg" alt="">2025-01-28 23:13 ― <a href="spresense-mandelbrot/">【SPRESENSE2024】仮想30コアでマンデルブロ集合【リアルタイム描画】</a> <span class="tags"><a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E3%82%BF%E3%83%83%E3%83%81%E3%83%91%E3%83%8D%E3%83%AB">タッチパネル</a> <a class="tag" href="?tag=%E3%83%9E%E3%83%AB%E3%83%81%E3%82%B3%E3%82%A2">マルチコア</a> <a class="tag" href="?tag=%E3%83%9E%E3%83%AB%E3%83%81%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89">マルチスレッド</a> <a class="tag" href="?tag=%E3%83%9E%E3%83%B3%E3%83%87%E3%83%AB%E3%83%96%E3%83%AD%E9%9B%86%E5%90%88">マンデルブロ集合</a> <a class="tag" href="?tag=%E3%83%AA%E3%82%A2%E3%83%AB%E3%82%BF%E3%82%A4%E3%83%A0">リアルタイム</a></span></li>
  <li data-tags="SPRESENSE|mp3|スピーカー|パッシブジェネレータ|自作オーディオ" data-date="2024-12-23 16:19"><img class="thumb" src="spresense-mp3-cub/images/img01.jpg" alt="">2024-12-23 16:19 ― <a href="spresense-mp3-cub/">【SPRESENSE2024】mp3綺麗に鳴らす【CuB】</a> <span class="tags"><a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=mp3">mp3</a> <a class="tag" href="?tag=%E3%82%B9%E3%83%94%E3%83%BC%E3%82%AB%E3%83%BC">スピーカー</a> <a class="tag" href="?tag=%E3%83%91%E3%83%83%E3%82%B7%E3%83%96%E3%82%B8%E3%82%A7%E3%83%8D%E3%83%AC%E3%83%BC%E3%82%BF">パッシブジェネレータ</a> <a class="tag" href="?tag=%E8%87%AA%E4%BD%9C%E3%82%AA%E3%83%BC%E3%83%87%E3%82%A3%E3%82%AA">自作オーディオ</a></span></li>
  <li data-tags="ESP32|ESPNOW|I2C|SPRESENSE|Wire|eNord" data-date="2024-12-12 13:14"><img class="thumb" src="spresense-enord/images/img01.png" alt="">2024-12-12 13:14 ― <a href="spresense-enord/">【SPRESENSE2024】esp32とSpresenseによるモデム機能制御"eNord"の構築</a> <span class="tags"><a class="tag" href="?tag=ESP32">ESP32</a> <a class="tag" href="?tag=ESPNOW">ESPNOW</a> <a class="tag" href="?tag=I2C">I2C</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=Wire">Wire</a> <a class="tag" href="?tag=eNord">eNord</a></span></li>
  <li data-tags="LCD|LovyanGFX|SPRESENSE|ST7735S" data-date="2024-10-31 13:30"><img class="thumb" src="spresense-multi-lcd/images/img01.jpg" alt="">2024-10-31 13:30 ― <a href="spresense-multi-lcd/">【SPRESENSE】複数LCD制御のためのSpresenseとLovyanGFXに関する実験的研究</a> <span class="tags"><a class="tag" href="?tag=LCD">LCD</a> <a class="tag" href="?tag=LovyanGFX">LovyanGFX</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=ST7735S">ST7735S</a></span></li>
  <li data-tags="CAMERA|SPRESENSE|パラパラ漫画|写真|動画" data-date="2024-02-06 21:29"><img class="thumb" src="spresense-camera-lcd/images/img01.png" alt="">2024-02-06 21:29 ― <a href="spresense-camera-lcd/">【SPRESENSE】液晶にカメラ画像の動画再生に成功したよっ</a> <span class="tags"><a class="tag" href="?tag=CAMERA">CAMERA</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E3%83%91%E3%83%A9%E3%83%91%E3%83%A9%E6%BC%AB%E7%94%BB">パラパラ漫画</a> <a class="tag" href="?tag=%E5%86%99%E7%9C%9F">写真</a> <a class="tag" href="?tag=%E5%8B%95%E7%94%BB">動画</a></span></li>
  <li data-tags="Audio|SPRESENSE|改造|環境音" data-date="2024-01-26 09:52"><img class="thumb" src="spresense-soundserenity/images/img01.jpg" alt="">2024-01-26 09:52 ― <a href="spresense-soundserenity/">【SPRESENSE2023】SoundSerenityを作ったよっ【環境音を8Trackで】</a> <span class="tags"><a class="tag" href="?tag=Audio">Audio</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E6%94%B9%E9%80%A0">改造</a> <a class="tag" href="?tag=%E7%92%B0%E5%A2%83%E9%9F%B3">環境音</a></span></li>
  <li data-tags="Library|LoRa|Module|SPRESENSE|通信" data-date="2024-01-26 09:49"><img class="thumb" src="spresense-lorapager/images/img01.jpg" alt="">2024-01-26 09:49 ― <a href="spresense-lorapager/">【SPRESENSE2023】LoRaPagerを作ったよっ【簡易メッセンジャー】</a> <span class="tags"><a class="tag" href="?tag=Library">Library</a> <a class="tag" href="?tag=LoRa">LoRa</a> <a class="tag" href="?tag=Module">Module</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E9%80%9A%E4%BF%A1">通信</a></span></li>
  <li data-tags="SPRESENSE|タッチパネル|改造" data-date="2024-01-26 09:47"><img class="thumb" src="spresense-bstem-touch/images/img01.jpg" alt="">2024-01-26 09:47 ― <a href="spresense-bstem-touch/">【SPRESENSE2023】B-Stem4CM01Displayにタッチパネルの改造を行うよっ</a> <span class="tags"><a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E3%82%BF%E3%83%83%E3%83%81%E3%83%91%E3%83%8D%E3%83%AB">タッチパネル</a> <a class="tag" href="?tag=%E6%94%B9%E9%80%A0">改造</a></span></li>
  <li data-tags="Mixer|SPRESENSE|fader" data-date="2024-01-26 09:45"><img class="thumb" src="spresense-tinymixer/images/img01.jpg" alt="">2024-01-26 09:45 ― <a href="spresense-tinymixer/">【SPRESENSE2023】tinyMixer【簡易ミキサー】</a> <span class="tags"><a class="tag" href="?tag=Mixer">Mixer</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=fader">fader</a></span></li>
  <li data-tags="Module|SPRESENSE|タッチパネル" data-date="2024-01-26 09:44"><img class="thumb" src="spresense-ili9341-touch/images/img01.png" alt="">2024-01-26 09:44 ― <a href="spresense-ili9341-touch/">【SPRESENSE2023】LCD液晶基板ili9341でタッチパネルを使うよっ</a> <span class="tags"><a class="tag" href="?tag=Module">Module</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E3%82%BF%E3%83%83%E3%83%81%E3%83%91%E3%83%8D%E3%83%AB">タッチパネル</a></span></li>
  <li data-tags="SPRESENSE|fader|mixer" data-date="2024-01-26 09:41"><img class="thumb" src="spresense-virtual-fader/images/img01.jpg" alt="">2024-01-26 09:41 ― <a href="spresense-virtual-fader/">【SPRESENSE2023】タッチパネルにバーチャルフェーダーを載せたよっ</a> <span class="tags"><a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=fader">fader</a> <a class="tag" href="?tag=mixer">mixer</a></span></li>
  <li data-tags="Module|SPRESENSE|キーボード|タッチパネル" data-date="2024-01-26 09:40"><img class="thumb" src="spresense-virtual-keyboard/images/img01.jpg" alt="">2024-01-26 09:40 ― <a href="spresense-virtual-keyboard/">【SPRESENSE2023】タッチパネルにバーチャルキーボードを載せたよっ</a> <span class="tags"><a class="tag" href="?tag=Module">Module</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E3%82%AD%E3%83%BC%E3%83%9C%E3%83%BC%E3%83%89">キーボード</a> <a class="tag" href="?tag=%E3%82%BF%E3%83%83%E3%83%81%E3%83%91%E3%83%8D%E3%83%AB">タッチパネル</a></span></li>
  <li data-tags="Library|LoRa|Module|SPRESENSE" data-date="2024-01-26 09:38"><img class="thumb" src="spresense-lora-lib/images/img01.png" alt="">2024-01-26 09:38 ― <a href="spresense-lora-lib/">【SPRESENSE2023】LoRaのライブラリを使いやすくしたよっ</a> <span class="tags"><a class="tag" href="?tag=Library">Library</a> <a class="tag" href="?tag=LoRa">LoRa</a> <a class="tag" href="?tag=Module">Module</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a></span></li>
  <li data-tags="8Track再生|ArduinoIDE|Audio|Module|SPRESENSE|ライブラリ" data-date="2024-01-26 09:36"><img class="thumb" src="spresense-audiodsp-jissen/images/img01.jpg" alt="">2024-01-26 09:36 ― <a href="spresense-audiodsp-jissen/">【SPRESENSE2023】AudioDSPを絶妙にハックしたよっ【実践編】</a> <span class="tags"><a class="tag" href="?tag=8Track%E5%86%8D%E7%94%9F">8Track再生</a> <a class="tag" href="?tag=ArduinoIDE">ArduinoIDE</a> <a class="tag" href="?tag=Audio">Audio</a> <a class="tag" href="?tag=Module">Module</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA">ライブラリ</a></span></li>
  <li data-tags="Module|SPRESENSE|WS2812B|組込み" data-date="2024-01-26 09:34"><img class="thumb" src="spresense-module/images/img01.jpg" alt="">2024-01-26 09:34 ― <a href="spresense-module/">【SPRESENSE2023】組込用のSPRESENSEモジュールを作ったよ</a> <span class="tags"><a class="tag" href="?tag=Module">Module</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=WS2812B">WS2812B</a> <a class="tag" href="?tag=%E7%B5%84%E8%BE%BC%E3%81%BF">組込み</a></span></li>
  <li data-tags="3Dプリンタ|Module|SPRESENSE|WS2812B" data-date="2024-01-26 09:31"><img class="thumb" src="spresense-ws2812b/images/img01.png" alt="">2024-01-26 09:31 ― <a href="spresense-ws2812b/">【SPRESENSE2023】WS2812B で流れるような光効果を～</a> <span class="tags"><a class="tag" href="?tag=3D%E3%83%97%E3%83%AA%E3%83%B3%E3%82%BF">3Dプリンタ</a> <a class="tag" href="?tag=Module">Module</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=WS2812B">WS2812B</a></span></li>
  <li data-tags="3Dプリンタ|Arduino|Audio|ILI9341|SPRESENSE|WS2812B|タッチパネル" data-date="2024-01-24 21:08"><img class="thumb" src="spresense-2023-matome/images/img01.jpg" alt="">2024-01-24 21:08 ― <a href="spresense-2023-matome/">【SPRESENSE2023】開発まとめ</a> <span class="tags"><a class="tag" href="?tag=3D%E3%83%97%E3%83%AA%E3%83%B3%E3%82%BF">3Dプリンタ</a> <a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=Audio">Audio</a> <a class="tag" href="?tag=ILI9341">ILI9341</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=WS2812B">WS2812B</a> <a class="tag" href="?tag=%E3%82%BF%E3%83%83%E3%83%81%E3%83%91%E3%83%8D%E3%83%AB">タッチパネル</a></span></li>
  <li data-tags="Arduino|Audio|DSP|simpleFIFO|spresense" data-date="2023-12-14 19:51"><img class="thumb" src="spresense-dsp-riron/images/img01.jpg" alt="">2023-12-14 19:51 ― <a href="spresense-dsp-riron/">SPRESENSE の DSPを微妙にHackするよっ【理論編】</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=Audio">Audio</a> <a class="tag" href="?tag=DSP">DSP</a> <a class="tag" href="?tag=simpleFIFO">simpleFIFO</a> <a class="tag" href="?tag=spresense">spresense</a></span></li>
  <li data-tags="Arduino|E220|LoRa|SPRESENSE" data-date="2023-12-12 08:05"><img class="thumb" src="lorae220-fix/images/img01.jpg" alt="">2023-12-12 08:05 ― <a href="lorae220-fix/">LoRaE220_900T22S_JPのサンプルが間違ってたので修正したよっ</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=E220">E220</a> <a class="tag" href="?tag=LoRa">LoRa</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a></span></li>
  <li data-tags="Arduino|Library|SPRESENSE|multicore|thread" data-date="2023-12-01 16:06"><img class="thumb" src="spresense-setup1loop1/images/img01.jpg" alt="">2023-12-01 16:06 ― <a href="spresense-setup1loop1/">RP2040のsetup1/loop1が素敵なので SPRESENSE にも実装してみたよっ</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=Library">Library</a> <a class="tag" href="?tag=SPRESENSE">SPRESENSE</a> <a class="tag" href="?tag=multicore">multicore</a> <a class="tag" href="?tag=thread">thread</a></span></li>
  <li data-tags="IMU|M5Atom|SLAM|easyLiDAR|spresense" data-date="2022-09-15 14:30"><img class="thumb" src="easylidar/images/img01.jpg" alt="">2022-09-15 14:30 ― <a href="easylidar/">Spresensを使って自律移動出来るよっ(easyLiDARの製作と運用)</a> <span class="tags"><a class="tag" href="?tag=IMU">IMU</a> <a class="tag" href="?tag=M5Atom">M5Atom</a> <a class="tag" href="?tag=SLAM">SLAM</a> <a class="tag" href="?tag=easyLiDAR">easyLiDAR</a> <a class="tag" href="?tag=spresense">spresense</a></span></li>
  <li data-tags="Arduino|ESP32|M5Atom|WiFi|esp32Modem|spresense" data-date="2022-08-24 21:06"><img class="thumb" src="spresense-blynk/images/img01.jpg" alt="">2022-08-24 21:06 ― <a href="spresense-blynk/">SpresenseをBlynkに接続出来たよっ</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=ESP32">ESP32</a> <a class="tag" href="?tag=M5Atom">M5Atom</a> <a class="tag" href="?tag=WiFi">WiFi</a> <a class="tag" href="?tag=esp32Modem">esp32Modem</a> <a class="tag" href="?tag=spresense">spresense</a></span></li>
  <li data-tags="Arduino|Arduino_IDE|Library|疑似マルチタスク" data-date="2022-06-23 15:41"><img class="thumb" src="arduino-setup1loop1/images/img01.jpg" alt="">2022-06-23 15:41 ― <a href="arduino-setup1loop1/">RP2040のsetup1/loop1が素敵なので Arduino でも使えるように実装してみたよっ</a> <span class="tags"><a class="tag" href="?tag=Arduino">Arduino</a> <a class="tag" href="?tag=Arduino_IDE">Arduino_IDE</a> <a class="tag" href="?tag=Library">Library</a> <a class="tag" href="?tag=%E7%96%91%E4%BC%BC%E3%83%9E%E3%83%AB%E3%83%81%E3%82%BF%E3%82%B9%E3%82%AF">疑似マルチタスク</a></span></li>
  <li data-tags="Arduino_IDE|ESP32|Library|M5Atom|multicore" data-date="2022-06-20 16:12"><img class="thumb" src="m5stack-setup1loop1/images/img01.jpg" alt="">2022-06-20 16:12 ― <a href="m5stack-setup1loop1/">RP2040のsetup1/loop1が素敵なので M5Stack/ESP32 にも実装してみたよっ</a> <span class="tags"><a class="tag" href="?tag=Arduino_IDE">Arduino_IDE</a> <a class="tag" href="?tag=ESP32">ESP32</a> <a class="tag" href="?tag=Library">Library</a> <a class="tag" href="?tag=M5Atom">M5Atom</a> <a class="tag" href="?tag=multicore">multicore</a></span></li>
</ul>

<div id="pager"></div>

<script>
(function () {
  var list = document.getElementById('articleList');
  var btn = document.getElementById('sortBtn');
  var pager = document.getElementById('pager');
  var searchBox = document.getElementById('searchBox');
  var tagBar = document.getElementById('tagBar');
  var info = document.getElementById('resultInfo');
  var desc = true;
  var page = 1;
  var PER = 20;
  var curTag = '';
  var curQ = '';
  var params = new URLSearchParams(location.search);
  if (params.get('tag')) curTag = params.get('tag');
  if (params.get('q')) { curQ = params.get('q'); searchBox.value = curQ; }
  var allItems = Array.prototype.slice.call(list.querySelectorAll('li'));
  var counts = {};
  allItems.forEach(function (li) {
    (li.getAttribute('data-tags') || '').split('|').forEach(function (t) { if (t) counts[t] = (counts[t] || 0) + 1; });
  });
  var tagNames = Object.keys(counts).sort(function (a, b) { return counts[b] - counts[a] || (a < b ? -1 : 1); });
  function addTagBtn(label, tag) {
    var b = document.createElement('button');
    b.type = 'button';
    b.className = 'tagbtn';
    b.textContent = label;
    b.setAttribute('data-tag', tag);
    b.addEventListener('click', function () { curTag = (curTag === tag) ? '' : tag; page = 1; render(); });
    tagBar.appendChild(b);
  }
  addTagBtn('すべて', '');
  tagNames.forEach(function (t) { addTagBtn(t + ' (' + counts[t] + ')', t); });
  function visibleItems() {
    var q = curQ.replace(/^\s+|\s+$/g, '').toLowerCase();
    return allItems.filter(function (li) {
      var tgs = (li.getAttribute('data-tags') || '').split('|');
      if (curTag && tgs.indexOf(curTag) < 0) return false;
      if (q && (li.textContent + ' ' + tgs.join(' ')).toLowerCase().indexOf(q) < 0) return false;
      return true;
    });
  }
  function render() {
    Array.prototype.forEach.call(tagBar.children, function (b) {
      b.className = 'tagbtn' + (b.getAttribute('data-tag') === curTag ? ' active' : '');
    });
    var items = visibleItems();
    items.sort(function (a, b) {
      var x = a.getAttribute('data-date'), y = b.getAttribute('data-date');
      if (x === y) return 0;
      return desc ? (x < y ? 1 : -1) : (x < y ? -1 : 1);
    });
    allItems.forEach(function (li) { li.style.display = 'none'; });
    items.forEach(function (li) { list.appendChild(li); });
    var pages = Math.max(1, Math.ceil(items.length / PER));
    if (page > pages) page = pages;
    items.forEach(function (li, i) {
      li.style.display = (i >= (page - 1) * PER && i < page * PER) ? '' : 'none';
    });
    info.textContent = '全 ' + allItems.length + ' 件中 ' + items.length + ' 件' + (curTag ? '（タグ: ' + curTag + '）' : '') + (curQ ? '（検索: ' + curQ + '）' : '');
    pager.innerHTML = '';
    var prev = document.createElement('button');
    prev.type = 'button';
    prev.className = 'pager-btn';
    prev.textContent = '前へ';
    prev.disabled = (page <= 1);
    if (page <= 1) prev.style.opacity = '0.4';
    prev.addEventListener('click', function () { if (page > 1) { page--; render(); } });
    pager.appendChild(prev);
    for (var n = 1; n <= pages; n++) {
      (function (n) {
        var b = document.createElement('button');
        b.type = 'button';
        b.className = 'pager-btn' + (n === page ? ' current' : '');
        b.textContent = n;
        b.addEventListener('click', function () { page = n; render(); });
        pager.appendChild(b);
      })(n);
    }
    var next = document.createElement('button');
    next.type = 'button';
    next.className = 'pager-btn';
    next.textContent = '次へ';
    next.disabled = (page >= pages);
    if (page >= pages) next.style.opacity = '0.4';
    next.addEventListener('click', function () { if (page < pages) { page++; render(); } });
    pager.appendChild(next);
  }
  btn.addEventListener('click', function () {
    desc = !desc;
    btn.textContent = desc ? '古い順に並べ替える' : '新しい順に並べ替える';
    page = 1;
    render();
  });
  searchBox.addEventListener('input', function () { curQ = searchBox.value; page = 1; render(); });
  var tagToggle = document.getElementById('tagToggle');
  function applyTagVis() {
    var on = false;
    try { on = localStorage.getItem('linalog-tags-on') === '1'; } catch (e) {}
    document.body.classList.toggle('tags-off', !on);
    tagToggle.textContent = on ? 'タグを隠す' : 'タグを表示';
  }
  tagToggle.addEventListener('click', function () {
    var willShow = document.body.classList.contains('tags-off');
    try { localStorage.setItem('linalog-tags-on', willShow ? '1' : '0'); } catch (e) {}
    applyTagVis();
  });
  applyTagVis();
  render();
})();
</script>
<div class="like-wrap"><div data-lyket-type="like" data-lyket-template="twitter" data-lyket-namespace="linalog" data-lyket-color-icon="#b5e853" data-lyket-color-primary="#b5e853" data-lyket-color-text="#eaeaea" data-lyket-color-secondary="#2b2b2b" data-lyket-id="top"></div></div>
