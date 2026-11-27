---
image: https://chrmlinux.github.io/linaLog/livec/images/img01.jpg
---

# [#SPRESENSE 2026]Cインタプリタ(VM)を作ったにょ[LiveC編]

2026年8月26日 11:44

![](images/img01.jpg)

# テキストエディタを核にした、USBキーボード/マウス対応の組み込みC実行環境「liveC」を作った

## 概要

Arduino環境で動く自作の組み込みOS「liveC」を作りました。特徴はこの3つです。

- **テキストエディタがOSの核**になっていて、その場でCコードを書いて実行できる
- **USB接続の一般的なキーボード/マウス**がそのまま使える(専用の変換基板 CH9350L 経由)
- 積んでいる**Cサブセットインタプリタ(livec)がかなり本格的**で、構造体・switch・配列・マクロまで動く

対象ボードは以下の2枚です。

- Sony Spresense
- Vicharak Shrike-Lite (RP2040)

## きっかけ

「マイコン上で動く、その場でコードを書いて動かせる環境が欲しい」というのが出発点でした。PCで書いてビルドして書き込んで…のサイクルではなく、電源を入れたらすぐエディタが立ち上がり、その場でCっぽいコードを書いて`:run`一発で動く。ゲーム機やレトロPCのBASIC環境に近い体験を、令和のマイコンとCで再現するイメージです。

## テキストエディタをOSの核にする

liveCは起動すると、DOS風のシェルか、nanoスタイルのテキストエディタが立ち上がります。

- `nanoeditor.hpp` … エディタのデータモデル(行バッファ、カーソル、スクロール等)
- `editorloop.hpp` … キー入力ループ、マウス対応(クリックでカーソル移動、ホイールでスクロール)
- `shell.hpp` … DOS風のコマンドシェル(`edit`、`run`、`ls`などのコマンド)

エディタ上でCのソースを書いて `Ctrl+R` (またはシェルの `run` コマンド) を叩くと、その場でコンパイル→実行されます。ファイルはSD/LittleFSに保存でき、書いたスクリプトを蓄積していけます。

画面は LovyanGFX 経由で ILI9341 (320×240) / ILI9488 (480×320) の2種類のパネルに対応しており、ビルド時のマクロ1つで切り替えられるようにしてあります。

## USBキーボード/マウス対応

マイコンにUSBホスト機能で汎用のキーボード・マウスを繋ぐのは地味に面倒なポイントですが、**CH9350L** という「USB→UART変換」チップを使うことで解決しました。

- CH9350LにUSBキーボード・マウスを挿すと、UART経由でHIDレポートが飛んでくる
- `ch9350lib.hpp` でこのUARTフレーム(可変長: マウスは4バイトのステータスフレーム、キーボードは長さバイト付きのデータフレーム)をパースし、キー入力とマウスの相対/絶対座標に変換
- JIS106キー配列に対応(US配列だと記号がズレるので要注意)
- マウスカーソルは背景退避/復元方式のビットマップ描画(`cursor.hpp`)

さらに `input.hpp` で「USBシリアル(Arduino IDEのシリアルモニタ)」と「CH9350経由のUSBキーボード」を1本の入力ストリームに統合しています。エディタやシェル、Cインタプリタの `getch()`/`kbhit()` はどちらの入力源からのキーも区別なく受け取れます。

ハードウェア的には、CH9350の2つ目のUSB-Aポート(スタック型コネクタ)を使うことで、キーボードとマウスを同時に挿して両方使える、いわゆる「下位機モード」で動作させています。

## Cインタプリタ「livec」

このOS向けに自作したCサブセットのインタプリタで、`c4`(Robert Swierczek氏の有名な自己ホスト型ミニCコンパイラ)を土台にしたスタックマシン方式です。テキストをその場でバイトコードにコンパイルし、シンプルなVMで実行します。

対応している機能は以下の通りです。

### 基本

- `int` / `char` / ポインタ、四則演算・比較・論理演算・ビット演算(`| ^ & << >>`)、三項演算子 `?:`
- `if` / `else` / `while` / `for` (初期化子付き宣言 `for (int i = 0; ...)` にも対応)
- `switch` / `case` / `default` … 本物のC同様のフォールスルー、`break` で抜けられる(`break`は`while`/`for`でも使用可)
- 関数定義・呼び出し・再帰
- `++` / `--` (前置・後置とも、ポインタ演算のスケーリングも正しく処理)
- 変数宣言時の初期化子 (`int a = 1;`)、グローバル変数の定数初期化

### 全体の流れ

`c4`系の設計の特徴は、**構文木(AST)を作らない**ことです。字句解析したトークン列を読みながら、その場でバイトコードを直接書き出していきます。ソースを1回なめるだけでコンパイルが終わるので、マイコン上でも十分な速度が出ます。

| 処理 | 説明 |
| --- | --- |
| ソース(.c テキスト) | |
| レキサー | 1文字ずつ読んでトークン列に変換 (識別子・数値・記号...) |
| コンパイラ (構文木なし) | トークンを読みながらその場でバイトコードを emit。※ if/while/for の分岐先アドレスは「あとで埋める」プレースホルダ方式 |
| バイトコード | text_[] 配列に積まれた命令列 |
| VM | 単純なスタックマシンが1命令ずつ実行 |

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

### VMの内部

レジスタはたったの4つだけです。

| レジスタ | 役割 |
| --- | --- |
| `a` | アキュムレータ(演算結果はここに入る) |
| `pc` | 次に実行する命令のアドレス |
| `sp` | スタックポインタ(値を積む場所。下に伸びる) |
| `bp` | ベースポインタ(今の関数のローカル変数の基準点) |

メモリ領域は3つに分かれています。

```
text_[]   ... コンパイル済みバイトコード(命令列)
  data_[]   ... グローバル変数、文字列リテラル
  stack_[]  ... 関数呼び出しのスタック(ローカル変数もここに乗る)
```

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

命令セットは`LEA`(アドレス計算)`IMM`(即値ロード)`LI`/`LC`(int/charロード)`SI`/`SC`(int/charストア)`PUSH``JMP``BZ`(ゼロなら分岐)`ENT`/`LEV`(関数の出入り)`ADD`/`SUB`/...(演算)など、30個弱です。

例えば `a = a + 1;` は、こんなバイトコード列にコンパイルされます。

```
LEA -1      // a: この変数はbpから見て何番目のスロットか(アドレス計算)
PUSH        // アドレスをスタックに退避
LEA -1
LI          // a の現在値をロード
PUSH
IMM 1
ADD         // 現在値 + 1
SI          // 退避しておいたアドレスへストア
```

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

一見遠回りですが、「アドレスを先に計算してスタックに積んでおき、右辺を評価してから最後にストアする」という型は、代入・複合代入・構造体のフィールド代入・配列要素代入まで全部同じパターンで扱えるので、実装がとてもシンプルになります。

### ローカル変数のアドレス配置(構造体・配列とバグの話)

関数に入ると `ENT` 命令が「ローカル変数用に何ワード確保するか」を`sp`から差し引きます。ローカル変数は `bp` からの相対オフセット(負の数)でアクセスします。

アドレス大 ↑

| 領域 | 位置・備考 |
| --- | --- |
| 呼び出し元の情報 | 直下の境界が bp (ここが基準点) |
| ローカル変数 #1 | bp-1 |
| ローカル変数 #2 | bp-2 |
| : | |

アドレス小 ↓ (spはここまで伸びる)

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

`bp`に近いほどアドレスが**大きく**、離れるほど**小さく**なります。構造体変数や配列を1つのローカル変数として複数ワード確保するとき、「1つ目のフィールド/要素がどっち向きに並ぶか」を最初逆に実装してしまい、2番目以降のフィールドが変数の外側のメモリを書き換えるバグを作り込みました(後述)。

### 構造体

```c
struct Point { int x; int y; };
struct Point origin;              // グローバルでもローカルでもOK

int dist2(struct Point *a, struct Point *b) {
    int dx = a->x - b->x;
    int dy = a->y - b->y;
    return dx*dx + dy*dy;
}
```

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

構造体は「型ごとにフィールド名→オフセットの対応表を持つ」という素朴な実装です。`.`/`->`アクセスは、どちらも最終的に「ベースアドレス + フィールドのオフセット」を計算するだけの、ほぼ同じコードに落ちます。

| ケース | 手順 |
| --- | --- |
| s.x (構造体を値として持っている場合) | s のアドレスは、実は s を参照した時点で「値をロードせず、アドレスのまま」保持している (配列名がポインタに落ちるのと同じ考え方) |
| s.x (構造体を値として持っている場合) | アドレス + (x のオフセット) → その番地をロード |
| p->x (構造体へのポインタの場合) | p の"値"(=ポインタの指す先のアドレス)をロード |
| p->x (構造体へのポインタの場合) | アドレス + (x のオフセット) → その番地をロード |

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

`.`と`->`で最終的にやっていることはほぼ同じ(「アドレス + オフセット」を計算してロードするだけ)というのが、実装してみて分かった面白いポイントでした。

- 自己参照ポインタ(`struct Node *next;`)で連結リストも組める
- 構造体変数を関数に「裸で」渡すと自動的にアドレス渡しになる(Cの配列がポインタに落ちるのと同じ考え方)
- 値渡し・値返し、構造体のネスト、初期化子は非対応(ポインタ渡しに寄せた割り切り設計)

### 固定長配列

```c
int a[5];
char buf[64];   // char配列はバイト単位でパック(int等と同じくワード単位で確保すると8倍メモリを食うため)
```

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

配列も構造体と同じ「連続したワードを確保して、名前は先頭アドレスに対応させる」方式です。ただし`char`配列だけは特別扱いをしています。`int`は元々1個1ワード(8バイト)を使う設計なので、配列にしてもそのままですが、`char`をそのままの規則で確保すると **1文字あたり8バイト消費**してしまい、`char buf[64];`が512バイトになってしまいます。文字列バッファは実用上よく使うサイズなので、`char`配列だけはバイト単位でパック(切り上げてワード数を計算)するようにしました。

int a[5]; (1要素 = 1ワード) — 合計 = 40B

| a[0] | a[1] | a[2] | a[3] | a[4] |
| --- | --- | --- | --- | --- |
| 8B | 8B | 8B | 8B | 8B |

char buf[9]; (バイト単位でパック) — 合計 = 16B (64Bなら512→64Bまで削減)

| b[0..7] | b[8] |
| --- | --- |
| 8B | 8B |

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

- ローカル・グローバルどちらも対応、配列名は関数に渡すと自動でポインタに変換される

### `switch`のコンパイル方式

`switch`は一見複雑ですが、実は「値を一時変数に退避 → 各`case`の定数と順番に比較 → 一致したらその`case`のコード位置へジャンプ」という単純な比較チェーンにコンパイルしています。

```c
switch (x) {
  case 1: A; break;
  case 2: B; break;
  default: C;
}
```

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

は、概念的にはこう展開されます。

| 要素 | 役割・接続先 |
| --- | --- |
| 本体のコード | 先に書いてしまい、あとから「x と 1 を比較 → 一致すれば A の位置へジャンプ」という比較チェーンを本体の"後ろ"に生やす |
| [ジャンプ命令: 比較チェーンへ] | 本体の先頭に置かれ、比較チェーンへジャンプする |
| A: ...; | break→switch終端 |
| B: ...; | break→switch終端 |
| C: ... | |
| 本体の実行範囲 | 本体はここまでフォールスルーで実行される |
| 比較チェーン (x==1 ?) | Aへジャンプ : 次へ |
| 比較チェーン (x==2 ?) | Bへジャンプ : 次へ |
| 比較チェーン (どれにも一致しない) | defaultがあればCへ、なければ終端へ |

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

`case`ラベル自体は何の命令も生成せず、「今のバイトコード上の位置」を記録するだけの目印です。本体を先にコンパイルしながら位置を記録しておき、本体の後ろに比較チェーンを生やして、最初にそこへジャンプする、という2段構えになっています。これは次の`for`の`post`式と同じ「前方参照」の問題を、同じ考え方で解いています。

### `for`のpost式 ― 実行順とソース上の順番のズレ

`for (init; cond; post) { body }` の `post` 部分は、ソース上では本体より前に書かれているのに、実際に実行される順番は「本体の後」です。1回ソースをなめるだけのコンパイラだと、これは地味に厄介な問題でした。

| 位置 | ソース上の見た目 | 実際の実行順 |
| --- | --- | --- |
| 1 | init | init |
| 2 | cond | cond |
| 3 | post | { body } |
| 4 | { body } | post |

ソース順 ≠ 実行順

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

解決策は「`post`部分を一旦トークンだけ読み飛ばして位置を覚えておき、本体をコンパイルし終わってから、その位置までレキサーを巻き戻してもう一度読み直し、今度は実際にコードを生成する」という二度読み方式です。バイトコードの並び順は「本体→post→条件判定に戻るジャンプ」という、実行順どおりの並びになります。

### プリプロセッサ

- `#include "file.c"` (SDから展開)
- `#define NAME 値` / `#undef` (関数形式マクロは非対応、文字列・コメント内は展開されない)

### ハードウェア連携(syscall)

Cコードから叩ける組み込み関数:

| カテゴリ | 関数 |
| --- | --- |
| 画面 | `print` `printi` `printc` `cls` `gotoxy` `rect` `frect` `flush` |
| 時間 | `delayms` `nowms` |
| GPIO | `aread` `dread` `dwrite` `pmode` |
| 入力 | `getch` `kbhit` (Serial + CH9350キーボード両対応) |
| メモリ | `malloc` `free` `peek` `poke` |

## 実装で苦労した点

- **Ctrl+CによるプログラムAbort**: 無限ループしているCコードを外から止めるため、`kbhit`/`getch`を呼んでいなくても定期的にCtrl+Cをポーリングする仕組みを、実行速度を落とさない形で入れる必要がありました
- **構造体・配列のアドレス計算方向**(詳細は上記): ローカル変数のフィールド/要素オフセットの向きを最初逆にしてしまい、2番目以降が変数の外側のメモリを壊すバグを作り込みました(テスト中に発見・修正)
- **`for`のpost式・`switch`の分岐**(詳細は上記): どちらも「実行順とソース上の記述順がズレる」問題を、字句解析位置の巻き戻し/コード位置の記録という同じ考え方で解決しました
- **Spresenseのメモリ**: ILI9488のような大きい画面を使うと、Arduino IDEの「MainCore RAM」設定を1024KBにしないとCインタプリタが無言でハングする現象がありました(768KBだとメモリ不足)

## ファイル構成

```
config.hpp       ビルド時の切り替えマクロを集約
display.hpp      LovyanGFX / ILI9341・ILI9488両対応、描画プリミティブ
nanoeditor.hpp   エディタのデータモデル
ch9350lib.hpp    CH9350のUARTフレーム解析(マウス+キーボード)
input.hpp        USBシリアル + CH9350キーボードの統合入力
cursor.hpp       マウスカーソル描画
storage.hpp      SD/LittleFSアクセス
livec.hpp        Cサブセットインタプリタ本体 + ハードウェアsyscall
lineinput.hpp    共通のプロンプト/コマンドライン入力
editorloop.hpp   エディタのキー入力ループ
shell.hpp        起動時のDOS風シェル
main.hpp         全体の配線
```

function expandCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = codeBlock.querySelector('.triangle');
          if (!codeBlock.classList.contains('expanded')) {
            codeBlock.classList.add('expanded');
            triangle.classList.add('up');
          }
        }

        function toggleCode(event) {
          var codeBlock = event.currentTarget;
          var triangle = event.currentTarget.querySelector('.triangle');;
          if (codeBlock.classList.contains('expanded')) {
            codeBlock.classList.remove('expanded');
            triangle.classList.remove('up');
          } else {
            expandCode({ currentTarget: codeBlock });
          }
          event.stopPropagation();
        }

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
