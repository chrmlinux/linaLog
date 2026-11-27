---
image: https://chrmlinux.github.io/linaLog/livec-editor/images/img01.jpg
---

# [#SPRESENSE 2026]Cインタプリタ(VM)を作ったにょ[エディタ編]

2026年8月26日 13:28

![](images/img01.jpg)

# liveOSの中身 ― テキストエディタとシェルはどう作られているか

`livec`(Cインタプリタ)の話は[別記事](./liveOS_elchika.md)にまとめたので、ここでは liveOS のもう半分、**画面に触れる部分**(エディタ・シェル・マウス/キーボード入力)の実装を詳しく書きます。

## 全体像 ― どのファイルが何をしているか

| モジュール | 説明 | 依存先 |
| --- | --- | --- |
| display.hpp | 画面描画の一番下の層 (LovyanGFX、セル単位の描画プリミティブ) | ― |
| nanoEditor.hpp | エディタのデータモデル (テキストの中身) | display.hpp |
| cursor.hpp | マウスカーソルの描画 | display.hpp |
| lineInput.hpp | 1行入力の共通部品 (プロンプト/確認/コマンドライン) | display.hpp |
| editorLoop.hpp | エディタのキー入力ループ + UI | display.hpp |
| shell.hpp | 起動時のDOS風シェル | display.hpp |

`lineInput.hpp`が`editorLoop.hpp`と`shell.hpp`の両方から使われている、というのがポイントです。「1行テキストを入力させる」という処理は、エディタの保存ファイル名入力もシェルのコマンドライン入力も本質的に同じなので、共通化しています。

## 画面描画: 二重バッファとimmediateフラグ

liveOSの画面描画は、オフスクリーンの`spr`(スプライト、RAM上のバッファ)に描いてから、まとめて実画面`lcd`へ転送する**二重バッファ**方式が基本です。

| 要素 | 説明 |
| --- | --- |
| spr (RAMのオフスクリーンバッファ) | pushSprite() で lcd へ転送 |
| lcd (実画面/SPI経由) | spr から pushSprite() で受け取る |

ただ、キー入力のたびに毎回「全画面書いてpushSprite」をやっているとSPI転送がボトルネックになります。そこで`display.hpp`の描画プリミティブ(`EditorDisplay::charCell`など)は、すべて`immediate`という引数を持っています。

```cpp
static void charCell(int x, int y, char c, uint16_t fg, uint16_t bg, bool immediate = false) {
  spr.setTextColor(fg, bg);
  spr.drawChar(c, x * CELL_W, y * CELL_H);
  if (immediate) {
    lcd.setTextColor(fg, bg);
    lcd.drawChar(c, x * CELL_W, y * CELL_H);  // spr に加えて lcd にも直接描く
  }
}
```

- `immediate = false`(デフォルト): `spr`だけに描く。あとでまとめて`pushSprite`する全画面更新用
- `immediate = true`: `spr`と`lcd`の**両方**に直接描く。1文字やカーソル1マスだけ動かすときに、全画面分のSPI転送をせず「変わったセルだけ」を実画面に送る

これによって、「1文字打った」「カーソルを動かした」程度の操作はほぼ瞬時に画面に反映されます。

## エディタのデータモデル(nanoeditor.hpp)

テキストは`NanoLine`(固定長バッファ+長さ)の配列として単純に持っています。

```cpp
struct NanoLine {
  char buf[MAX_COLS];   // 1行分の文字バッファ(最大100文字)
  int len = 0;
};

struct NanoEditor {
  NanoLine lines[TOTAL_ROWS];  // 最大100行
  int cursorX, cursorY;         // カーソル位置(ファイル座標)
  int scrollRow, scrollCol;     // スクロール位置
  ...
};
```

画面の行数・列数はパネルの解像度によって変わるので、ビルド時に切り替えています。

```cpp
#ifdef USE_PANEL_ILI9488
const int EDIT_ROWS = 39;   // 480x320パネル
const int EDIT_COLS = 80;
#else
const int EDIT_ROWS = 29;   // 320x240パネル
const int EDIT_COLS = 53;
#endif
```

画面とファイルの座標変換は、行番号ガター(左端の3桁の行番号表示)の分だけオフセットさせて計算しています。

| ファイル座標 | 画面座標 |
| --- | --- |
| 001 int main() { | int main() { |
| 002 int x = 1; | int x = 1; |
| 003 ... | ... |

注記: scrollRow (縦スクロール)、行番号ガター (3桁+空白1) のため gutterの分だけ右にずれる、scrollCol分だけ横にもずれる

`ensureVisible()`が、カーソルが画面外に出そうになったら`scrollRow`/`scrollCol`を調整して追従させる役割を持っています。

## 全画面描画 vs 部分描画の使い分け

キー入力1回ごとに全画面を再描画していては遅いので、editorLoop.hppでは**操作の種類によって描画方法を使い分けて**います。

| 操作の種類 | 使う描画関数 | コスト |
| --- | --- | --- |
| ファイル読み込み・Ctrl+N (行番号切替) など | drawScreenLight() | 画面全体を spr→lcd転送 |
| カーソル移動のみ (行/桁が変わらない範囲) | moveCursorFast() | 古いカーソル位置と新しい位置、2セルだけ |
| 文字入力・same-line delete (行数が変わらない範囲) | afterCharEditFast() | その行だけ (immediate=trueで直接lcdへ) |

`moveCursorFast()`/`afterCharEditFast()`は、もし「行数が変わった」「スクロールが発生した」など高速化の前提が崩れるケースを検知したら、自動的に`drawScreenLight()`(全画面描画)にフォールバックします。

```cpp
void afterCharEditFast(int oldLineCount, int oldScrollRow, int oldScrollCol) {
  if (editor.lineCount != oldLineCount) {   // 行の分割/結合が起きた
    drawScreenLight();                       // → 前提が崩れたので全画面描画にフォールバック
    return;
  }
  if (editor.scrollRow != oldScrollRow || ...) {  // スクロールが発生した
    if (!scrollFast(...)) drawScreenLight();
    return;
  }
  // ここまで来たら「同じ行の中で文字が増減しただけ」→ その行だけ再描画
  ...
}
```

### ハードウェアで踏んだ罠: `scrollFast()`

「スクロールが1行だけなら、`lcd.scroll()`でピクセルをずらして、はみ出た1行だけ描き直せば速いはず」という最適化を試したのですが、実機でハングする現象が起きました。

```cpp
bool scrollFast(int oldScrollRow, int oldScrollCol) {
  return false;   // 常にfalseを返して無効化している

#if 0
  // 実装はここに残してあるが、有効化しない
  ...
#endif
}
```

パネルの`offset_rotation`設定と`lcd.scroll()`の相性の問題らしく、最初の2回は表示が乱れる程度でしたが、3回目に**大きいファイルで実際にスクロールを使うと画面が完全にフリーズ**する現象が発生しました。原因調査よりも安定動作を優先し、実装は`#if 0`で残しつつ`return false`で無効化して、確実に動く`drawScreenLight()`の全画面描画にフォールバックする形にしています。実機デバッグでしか踏めない類のバグで、教訓として関数のコメントに詳しく経緯を残してあります。

## キー入力ループとESCシーケンス

矢印キーなどは `ESC [ A` のような複数バイトのシーケンスとして届くので、状態機械でパースしています。

| 状態 | 入力 | 遷移先・動作 |
| --- | --- | --- |
| state0 | ESC受信 | state1 へ |
| state1 | '[' 受信 | state2 へ |
| state2 | 'A'/'B'/'C'/'D' | 対応する移動処理を実行、state0に戻る |
| state2 | '0'-'9' 受信 (拡張シーケンス、例: ESC[3~ = Delete) | state3 へ |
| state3 | '~' 受信 | 対応する処理を実行、state0に戻る |

これはエディタのメインループ(`runEditorLoop`)、Ctrl+Lのファイル選択ダイアログ(`pickFileFromSD`)、そして`lineInput.hpp`の共通入力処理、3箇所で同じパターンが出てきます(微妙に対応するキーの種類が違うので、今のところ関数化せず個別に書いています)。

### キーのディスパッチ

制御キーは`namespace Key`に列挙体としてまとめてあり、`if (c == Key::Ctrl_R) ...`という素朴な条件分岐チェーンでディスパッチしています。

| キー | 動作 |
|---|---|
| `Ctrl+R` | liveCで実行(`editor.onRun`) |
| `Ctrl+S` | SDへ保存 |
| `Ctrl+L` | ファイル選択して読み込み |
| `Ctrl+U` | liveCのメモリ使用状況ウィンドウを表示 |
| `Ctrl+N` | 行番号表示の切り替え |
| `Ctrl+C` / `Ctrl+V` | 行コピー/ペースト |
| `Ctrl+D` | 行複製 |
| `Ctrl+K` | 行削除 |
| `Ctrl+X` | 終了(保存確認あり) |
| `Ctrl+Q` | 全消去(確認あり) |

## マウス統合 ― キー入力を待たずに毎回ポーリング

マウスの動きは「キー入力があった時だけ」ではなく、**ループの毎周回**チェックしています。理由はコメントの通りで、キーボード入力が連続するとマウスの処理が飢餓状態になるのを防ぐためです。

```cpp
while (true) {
  bool hasKey = kbInputAvailable();  // これがCH9350 UARTの読み出しも兼ねている

  // ---- マウス処理は hasKey に関係なく毎回実行 ----
  クリック検出 → カーソル位置に応じて編集カーソルを移動
  ホイール検出 → スクロール
  updateCursor();  // マウスポインタ自体の描画

  if (hasKey) {
    // ここでようやくキー処理
  }
}
```

マウスカーソル自体の描画(`cursor.hpp`)は、**背景退避/復元方式**のシンプルなビットマップ描画です。

```
  1. カーソルを消す前に、今の位置の下に何が描かれていたか(_cursorBg)を使って復元
  2. 移動先の位置の「今の内容」をspr(オフスクリーンバッファ = 本当の画面内容)から読み取って退避
  3. その上に矢印ビットマップを透過色付きで描画
```

`spr`から読み取っているのがポイントで、「前回カーソル自身が描いた絵」を参照するとカーソルの残像がカーソルの背景として保存されてしまう、という事故を避けています。

## 共通の1行入力(lineInput.hpp)

保存ファイル名の入力、Y/N確認、シェルのコマンドライン ― これらは元々別々に実装されていて、カーソル位置がズレるバグをそれぞれ個別に踏んで直す、という非効率なことをしていました。今のバージョンは1つの`readLine()`に統合されています。

設計のポイントは「**キー入力のたびに、入力欄を丸ごと再描画する**」ことです。1文字ずつ差分更新する方式だと、カーソル位置の計算をどこか間違えるたびに表示が崩れるバグを踏みやすいのですが、丸ごと再描画なら常に正しい状態から描き直すので、その手のバグが原理的に起きません。

```cpp
auto redraw = [&]() {
  for (int i = 0; i < maxChars; i++) {
    char ch = (i < len) ? buf[i] : ' ';
    EditorDisplay::charCell(inputCol0 + i, row0, ch, TFT_WHITE, TFT_BLACK, true);
  }
  EditorDisplay::drawCursorBlock(inputCol0 + len, row0, ' ', TFT_WHITE, TFT_BLACK, true);
};
```

Tab補完・Up/Down履歴・`hotkey`(特定のキーで呼び出し元に処理を委譲する仕組み、例えばシェルのTab補完候補一覧など)もすべてこの1つの`readLine()`のオプション(`LineInputOptions`)として実装されており、エディタの保存ダイアログとシェルのコマンドラインで完全に同じコードパスを通っています。

## シェル(shell.hpp): セル座標の仮想端末

シェルは「セル座標(列・行)のカーソル位置を持つ、小さな仮想端末」として実装されています。

```cpp
static int g_shellCol = 0, g_shellRow = 0;

static void shellPutChar(char c, ...) {
  if (c == '\n') { shellNewline(); return; }
  EditorDisplay::charCell(g_shellCol, g_shellRow, c, fg, bg);
  Serial.write(c);           // Serialモニタにも同時出力
  g_shellCol++;
  if (g_shellCol >= g_shellCols) shellNewline();
}
```

画面下端まで来たら、`spr.scroll()`(RAM上のバッファをずらすだけ、`lcd.scroll()`は使わない)でスクロールします。これは前述の`scrollFast()`のハング事故を踏まえた選択で、パネルハードウェアに触れない「安全側」のスクロール実装です。

```cpp
static void shellScrollIfNeeded() {
  if (g_shellRow < g_shellRows) return;
  int dyPx = (g_shellRow - g_shellRows + 1) * EditorDisplay::CELL_H;
  spr.scroll(0, -dyPx);       // RAM上でずらすだけ、lcd.scroll()は使わない
  spr.fillRect(...);          // 空いた場所を消す
}
```

### コマンドの補完・履歴・ワイルドカード

DOSライクな体験を目指して、以下を実装しています。

- **Tab補完**: 先頭の単語ならコマンド名(`dir`/`edit`/`run`...)、2つ目以降ならカレントディレクトリのファイル名から補完。候補が1つに絞れる時だけ補完する(曖昧なら何もしない)
- **Up/Down履歴**: 直近8件をリングバッファで保持、連続する同一コマンドは記録しない
- **ワイルドカード**(`del *.c`、`copy *.txt backup/`など): `*`は任意長、`?`は1文字にマッチする、古典的な反復法でのマッチャーを自前実装

```cpp
static bool globMatch(const char* pattern, const char* text) {
  // pがワイルドカード'*'に当たったら、そこを起点(starP/starT)にバックトラック
  const char *p = pattern, *t = text;
  const char *starP = nullptr, *starT = nullptr;
  while (*t) {
    if (*p == '?' || tolower(*p) == tolower(*t)) { p++; t++; }
    else if (*p == '*') { starP = p++; starT = t; }
    else if (starP) { p = starP + 1; t = ++starT; }  // マッチ失敗→直前の'*'まで戻ってやり直す
    else return false;
  }
  while (*p == '*') p++;
  return *p == '\0';
}
```

コマンドは `dir`/`ls`/`cd`/`mkdir`/`rmdir`/`type`/`cat`/`del`/`copy`/`ren`/`move`/`run`/`edit`/`cls`/`help` あたりを実装していて、`copy`/`move`/`del`はワイルドカード一括処理にも対応しています。

## まとめ

エディタ・シェルまわりで一貫しているのは、「**土台となる描画/入力の部品を1つに共通化して、その上に個別のUIを組む**」という方針です。

- 描画は`EditorDisplay`の`immediate`フラグ付きプリミティブ1種類に集約
- 1行入力は`lineInput.hpp`の`readLine()`1つに集約(プロンプト/確認/コマンドラインすべてがこれを使う)
- キー入力は`input.hpp`でSerial+CH9350キーボードを1本のストリームに統合(これは[livec編](./liveOS_elchika.md)にも書いた通り)

個別に書いた方が最初は速く進むのですが、後から「あのバグ、前にも直したのに別の場所でまた出た」を繰り返すことになりがちだったので、今の形に寄せています。ハードウェアがらみ(`scrollFast`)は逆に「共通化・最適化を諦めて、確実に動く方に倒す」判断をした例です。


## 関連コンテンツ
・[liveOS liveC](../livec/)
・[liveOS エディタ編](../livec-editor/)
・[liveOS ch9350編](../livec-ch9350/)
・[liveOS LiveB/LiveP](../liveb-livep/)
・[liveOS LiveCO](../liveco/)
・[liveOS LiveFO](../livefo/)
・[liveOS LinDOS](../lindos/)
・[liveOS linaX](../spresense-linax/)
・[liveOS porting m5](../spresense-linax-port/) ←いまここ

*このプロジェクトはArduinoフレームワーク上で開発しています。ボードはVicharak Shrike-Lite (RP2040) / Sony Spresense / cyd / m5tab5 / m5sticks3 5機種に対応。*

<div class="like-wrap"><div data-lyket-type="like" data-lyket-namespace="linalog" data-lyket-id="livec-editor"></div></div>
