---
layout: default
title: linaLog
---

# linaLog

<p align="center">
  <img src="assets/rina-chan.png" alt="リナちゃん" width="360">
</p>

製作記事置き場です。Markdown のまま読めます。

Project write-ups, in Markdown.

## 記事 / Articles

<style>
#sortBtn {
  background: #151515;
  color: #b5e853;
  border: 1px solid #b5e853;
  padding: 4px 14px;
  font-family: Monaco, "Bitstream Vera Sans Mono", "Lucida Console", monospace;
  font-size: 13px;
  cursor: pointer;
  margin-bottom: 10px;
}
#sortBtn:hover { background: #b5e853; color: #151515; }
</style>

<p><button id="sortBtn" type="button">古い順に並べ替える</button></p>

<ul id="articleList">
  <li data-date="2026-11-26 20:17">2026-11-26 20:17 ― <a href="linax_uno.html">linaX(uno) ― Arduino Uno（ATmega328P）に載せた Unix 系 OS</a></li>
  <li data-date="2026-11-21 21:41">2026-11-21 21:41 ― <a href="linax_cyd.html">LinaX(cyd) ― PSRAM なしの ESP32（CYD）に載せた Linux 6.11</a></li>
</ul>

<script>
(function () {
  var list = document.getElementById('articleList');
  var btn = document.getElementById('sortBtn');
  var desc = true;
  function apply() {
    var items = Array.prototype.slice.call(list.querySelectorAll('li'));
    items.sort(function (a, b) {
      var x = a.getAttribute('data-date'), y = b.getAttribute('data-date');
      if (x === y) return 0;
      return desc ? (x < y ? 1 : -1) : (x < y ? -1 : 1);
    });
    items.forEach(function (li) { list.appendChild(li); });
    btn.textContent = desc ? '古い順に並べ替える' : '新しい順に並べ替える';
  }
  btn.addEventListener('click', function () { desc = !desc; apply(); });
})();
</script>
