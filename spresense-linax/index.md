---
image: https://chrmlinux.github.io/linaLog/spresense-linax/images/img01.jpg
---

# [#SPRESENSE 2026]最終奥義OS作ってみたにょ[linaX]

2026年9月8日 19:25

![](images/img01.jpg)

# Spresenseで動く「純粋Cインタプリタ製Linux風OS」LinaXを作った話

〜 カーネルもシェルも、全部Cインタプリタの上で動く仮想Linux環境 〜

## はじめに

「マイコンの上で、本物のLinuxっぽいものを動かしたい」

「しかもそのOS自体を、Cで書いて、Cインタプリタで解釈して動かしたい」

そんな欲張りな欲求から生まれたのが **LinaX** です。

LinaXは、Sony Spresense上で動作する **Pure 32-bit C-Interpreter OS** です。

以前elchikaで紹介した初期バージョンから、I2C対応・フルスクリーンエディタ・コマンド履歴など、実用面でかなり肉付けが進んだので、今回はアップデート版としてあらためて詳しく解説します。

特徴は以下の通り：

- カーネル本体・標準ライブラリ相当・シェルを **すべてCサブセットで記述**
- それらをSDカード上の仮想ファイルとしてデプロイし、起動時にロードして実行
- Linux風のコマンド（`ls` `cd` `mkdir` `nano` `edit` `cat` `cp` `mv` など）が一通り動く
- コマンド履歴（↑↓キー）とTab補完に対応したシェル
- 矢印キーでカーソル移動できる**フルスクリーンテキストエディタ**を搭載
- グラフィックス・GPIO・**I2C**・RTCまでsyscallとしてバインド
- ネイティブ側は「VM起動とハードウェア抽象化」だけに徹している

要するに、**「Cインタプリタをコアにした、自己ブートストラップする仮想Linux環境」** です。

姉妹プロジェクトとして、仮想16bit CPU + アセンブリ風言語でMS-DOSの構造を再現した [LinDos](../lindos/) も作っています。LinDosが「本物のDOSの階層をアセンブリで忠実に再現する」方向なのに対し、LinaXは「OS自身をCで書いて、Cインタプリタ自身に自分自身を実行させる」という、また違った尖り方をしています。

本記事では、設計思想から実装の詳細、ブートストラップの仕組み、そして今回追加した機能群までをかなり深掘りして解説します。

---

## 全体アーキテクチャ

LinaXは明確にレイヤ分けされています。上から下に行くほどハードウェアに近くなります。

| レイヤ（上→下） | 実装言語 | 主な役割・代表的な中身 |
| --- | --- | --- |
| SH_SRC（シェル本体） | インタプリタ上のC | main() / cmd_help / do_fork_test など |
| LIBC_SRC（ユーティリティ層） | インタプリタ上のC | linux_ls / linux_nano / linux_gpio など |
| VMLINUZ_SRC（syscallラッパー層） | インタプリタ上のC | io_puts / io_cat / io_pixel / io_i2c_* など |
| CInterp（liveC.hpp） | ネイティブC++ | c4スタイルの軽量Cサブセットインタプリタ |
| kernel.hpp | ネイティブC++ | syscall実装 + ブートローダ |
| edit.hpp | ネイティブC++ | フルスクリーンエディタ |
| term.hpp | ネイティブC++ | ターミナルエンジン + 履歴管理 |
| hw.hpp | ネイティブC++ | LovyanGFX + ILI9488 |

ポイントは、**上3層がすべてインタプリタが解釈するCソース**であることです。

ネイティブ側は「仮想マシンを用意して、仮想ファイルシステムから自分自身をロードする」役割と、「フルスクリーンエディタのような、シェルのループを一時的に乗っ取る重い画面制御」だけに徹しています。

---

## ブートストラップの仕組み（詳細）

ここがLinaXで一番面白い部分です。

### 起動シーケンス

1. Arduinoの`setup()` → `osSetup()`が呼ばれる
2. ハードウェア初期化（LCD・スプライト・RTC・Serial）
3. ターミナル初期化＆タイトル表示
4. SDカードマウント
5. Tab補完コールバックを登録（`vTerm::setCompleter`）
6. **仮想システムファイルをSDに書き込む**（deploy）
7. それらを**読み戻す**
8. 3つのソースを1本の巨大なCソースに結合
9. `CInterp`に渡してコンパイル＆実行
10. `SH_SRC`の`main()`が動き出し、`i2c_begin()`でI2Cバスを初期化してからシェルプロンプトが出る

### 実際のコード（抜粋）

```cpp
void osSetup() {
  // ... ハードウェア初期化 ...

  vTerm::setCompleter(completeWord); // Tab補完（コマンド名＋ファイル名）を登録

  // 仮想システムファイルをSDにデプロイ
  writeSystemFile("boot_vmlinuz", VMLINUZ_SRC);
  writeSystemFile("bin_libc",    LIBC_SRC);
  writeSystemFile("bin_sh",      SH_SRC);

  // 読み戻す
  bool ok1 = false, ok2 = false, ok3 = false;
  std::string vmlinuz_src = loadFileFromSD("boot_vmlinuz", ok1);
  std::string libc_src    = loadFileFromSD("bin_libc", ok2);
  std::string sh_src      = loadFileFromSD("bin_sh", ok3);

  // 結合
  std::string combinedKernelImage = vmlinuz_src + "\n" + libc_src + "\n" + sh_src;

  // インタプリタ起動
  auto vm = std::unique_ptr<CInterp>(new CInterp());
  register_linux_syscalls(*vm);
  vm->run(combinedKernelImage);   // ここでシェルが動き出す
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

**なぜ一度SDに書いて読み直すのか？**

- 「仮想ファイルシステムからロードする」というOSらしい形を取っている
- 将来的に「ユーザーがSD上のファイルを書き換えて再起動できる」設計の土台になる
- `deploy`コマンドの存在意義にもつながる

---

## 仮想イメージの3層構造

### 1. VMLINUZ_SRC（最下層）

ネイティブで登録したsyscallを、Cから呼びやすい薄いラッパーにしたものです。

```c
int io_puts(int s) { prints(s); return 0; }
int io_puti(int n) { printi(n); return 0; }
int io_cls()       { cls(); return 0; }
int io_new(int path) { return fs_new(path); }
int io_cat(int path) { return fs_cat(path); }
int io_pixel(int x, int y, int color) { return gfx_pixel(x, y, color); }
int io_i2c_scan()  { return i2c_scan(); }
int io_edit(int path) { return fs_edit(path); }
int io_hist_count() { return hist_count(); }
// ... など多数
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

ここが「カーネルのシステムコールインターフェース」に相当します。

### 2. LIBC_SRC（中間層）

`io_*`を使って、より人間が使いやすいコマンド関数を実装しています。

```c
int linux_ls() { sd_dir(); return 0; }
int linux_nano(int arg) { /* 行単位の簡易エディタ実装 */ }
int linux_edit(int arg) { io_edit(arg); return 0; } // フルスクリーンエディタを起動
int linux_gpio(int arg) { /* pin番号+in/out/read/analog等のサブコマンド分岐 */ }
int linux_i2c_scan() { io_i2c_scan(); return 0; }
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

### 3. SH_SRC（最上層）

これが最終的に実行されるシェル本体です。

```c
int main() {
  io_puts("\nWelcome to Pure Linux Style Environment");
  io_nl();
  i2c_begin();   // I2Cバスを起動時に初期化
  while (1) {
    int line = io_gets(io_prompt());   // "root@spresense:/# "
    int cmd  = io_cmd(line);
    int arg  = io_arg(line);

    if (linux_strcmp(cmd, "ls") == 1)     { linux_ls(); }
    if (linux_strcmp(cmd, "edit") == 1)   { linux_edit(arg); }
    if (linux_strcmp(cmd, "scan") == 1)   { linux_i2c_scan(); }
    if (linux_strcmp(cmd, "gpio") == 1)   { linux_gpio(arg); }
    // ... 全コマンド
  }
  return 0;
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

---

## Cインタプリタ（liveC.hpp）の特徴

ベースは有名な **c4** の系譜を汲む、軽量Cサブセットインタプリタです。

対応している機能：

- `int` / `char` / ポインタ / 構造体
- `if` / `else` / `while` / `for` / `switch-case`（フォールスルー付き） / `break`
- 関数定義・呼び出し
- 配列（固定長）
- オブジェクト形式の`#define`
- 自己参照ポインタを持つ構造体

非対応：

- `float` / `union` / `enum` / `typedef`
- 多次元配列
- 構造体の値渡し・返却
- 関数形式マクロ

メモリ設定（Spresense向け）：

```cpp
#define CINTERP_POOL_SIZE  (24 * 1024)  // 命令領域
#define CINTERP_STACK_SIZE ( 4 * 1024)  // スタック
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

外部コマンド実行（`.c`ファイルの直接実行）では、親と同じ大きなプールをもう1つ確保するとRAMが足りなくなるため、子プロセス用に小さめのプール（text/data各16KB、スタック4KB）を別途確保する形にしています。子プロセスにも同じsyscallセットを登録しているので、ネストした実行（外部コマンドの中からさらに外部コマンドを呼ぶ）も可能です。

実行エンジンは **Computed Goto版** と通常のswitch版の両方を持ち、`#define COMPUTED_GOTO` で切り替え可能です。

---

## シェルの強化: コマンド履歴とTab補完

今回の更新でシェル自体もかなり実用的になりました。

### コマンド履歴（↑↓キー）

`term.hpp`側にリングバッファ（最大20件）を持たせ、Enterで確定した行を履歴に積んでいきます。↑キーで古い履歴へ、↓キーで新しい履歴（または空行）へ戻れます。

```cpp
static void pushHistory(const std::string& line) {
  if (line.empty()) return;
  if (historyCount_ > 0 && history_[historyCount_ - 1] == line) return;
  if (historyCount_ >= HISTORY_MAX) {
    for (int i = 1; i < HISTORY_MAX; i++) history_[i - 1] = history_[i];
    historyCount_ = HISTORY_MAX - 1;
  }
  history_[historyCount_++] = line;
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

この履歴は`vTerm_histCount_bridge`/`vTerm_histGet_bridge`を通じてCInterp側にも公開されており、`history`コマンドでシェルの中から一覧表示することもできます。

### Tab補完

ビルトインコマンド名と、カレントディレクトリのファイル/ディレクトリ名の両方から前方一致で候補を探します。候補が複数ある場合は共通する先頭部分まで補完する、シェルではおなじみの挙動です。

---

## フルスクリーンテキストエディタ（edit.hpp）

もともとLinaXには「`.`だけの行を入力すると保存終了する」という、かなり原始的な`nano`コマンドしかありませんでした。今回、姉妹プロジェクトのliveOS2で作っていたフルスクリーンエディタを移植し、`edit`コマンドとして追加しました（`nano`は互換性のためそのまま残しています）。

### できること

- 矢印キーでのカーソル移動（行末・行頭をまたぐ移動にも対応）
- Ctrl+S: ファイル名を指定して保存
- Ctrl+L: SDカード上のファイル一覧から選んで読み込み（Deleteキーでその場削除も可能）
- Ctrl+C / Ctrl+V: 行コピー・貼り付け
- Ctrl+D: 行複製
- Ctrl+K: 行削除
- Ctrl+N: 行番号表示のON/OFF
- Ctrl+F: 画面下部の機能バー表示のON/OFF
- Ctrl+Q: 全消去（確認あり）
- Ctrl+X: 終了（未保存なら確認あり）

### 実装のポイント

liveOS2版はUSBキーボード/マウスブリッジ（CH9350L）にも対応していましたが、LinaXはシリアル端末（TeraTerm等）からの入力のみなので、マウス関連の機能は移植時にすべて削除しています。矢印キーの検出はANSIエスケープシーケンス（`\x1b[A`など）をそのまま流用できたので、この部分はほぼそのまま使えました。

```cpp
if (escState == 2) {
  switch (c) {
    case 'A': moveCursorFast([] { editor.moveUp();    }); escState = 0; break;
    case 'B': moveCursorFast([] { editor.moveDown();  }); escState = 0; break;
    case 'C': moveCursorFast([] { editor.moveRight(); }); escState = 0; break;
    case 'D': moveCursorFast([] { editor.moveLeft();  }); escState = 0; break;
  }
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

カーソル移動や1文字編集のような小さな変更では、画面全体を再描画せずに変化したセルだけを`immediate`フラグ付きで直接LCDに書き込む（`drawScreenLight`のフル再描画とは別に`moveCursorFast`/`afterCharEditFast`で差分更新する）ことで、SPI転送量を抑えて操作感を軽くしています。

エディタの起動はシェル側から見るとただのsyscall1つです。

```c
int io_edit(int path) { return fs_edit(path); }
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

```cpp
vm.registerSyscall("fs_edit", [](CInterp&, long* a, int) -> long {
  extern void runEditorOnFile(const char* name);
  std::string name = (a && a[0]) ? std::string((const char*)a[0]) : std::string("EDIT.TXT");
  runEditorOnFile(name.c_str());
  return 1;
});
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

`runEditorOnFile`がファイルロード→編集ループ→（必要なら）保存までを一体で処理し、抜けるときにフルフレームをシェル画面へ書き戻します。CInterp側は「エディタを起動して結果が返ってくるのを待つ」だけで済むので、シェルのコード自体はとてもシンプルなままです。

---

## I2C対応

センサー類を繋いで遊べるように、I2Cもsyscallとして生やしました。

```cpp
vm.registerSyscall("i2c_write", [](CInterp&, long* a, int) -> long {
  uint8_t addr = (uint8_t)a[0];
  uint8_t reg  = (uint8_t)a[1];
  uint8_t val  = (uint8_t)a[2];
  Wire.beginTransmission(addr);
  Wire.write(reg);
  Wire.write(val);
  uint8_t err = Wire.endTransmission();
  return (err == 0) ? 1 : 0;
});
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

`i2c_begin`/`i2c_write(addr, reg, val)`/`i2c_read(addr, reg)`というレジスタ単位の低レベルAPIに加えて、`scan`コマンドから呼べる`i2c_scan()`も実装しています。こちらは0x03〜0x77をスキャンし、応答のあったアドレスをその場でシェル画面にhex表示するところまでsyscall内で完結させています（CInterp側にアドレス配列を返す必要がなく、シェル側の実装が1行で済むのが利点です）。

```
root@spresense:/# scan
Scanning I2C bus...
  0x3C
  0x68
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

`i2c_begin()`はシェルの`main()`起動時に自動で呼ばれるので、ユーザーが明示的に初期化コマンドを打つ必要はありません。

---

## システムコール一覧（抜粋）

`register_linux_syscalls()` で登録している主なもの：

| カテゴリ | 主なsyscall |
| --- | --- |
| 出力 | prints / printi / printc / cls / flush |
| 入力 | readin |
| 文字列/数値変換 | str_eq / arg_cmd / arg_rest / atoi |
| ファイルシステム | fs_new / fs_put / fs_end / fs_cat / fs_del / fs_pwd / fs_chdir / fs_mkdir / fs_rmdir / fs_touch / fs_copy / fs_move / sd_dir / fs_edit |
| メモリ | fs_mem |
| 時刻 | time_year / time_month / ... / time_now |
| グラフィックス | gfx_pixel / gfx_line / gfx_rect / gfx_circle / gfx_text / gfx_clear / gfx_flush |
| GPIO | dmode / dwrite / dread / aread |
| I2C | i2c_begin / i2c_write / i2c_read / i2c_scan |
| 履歴 | hist_count / hist_get |
| その他 | sys_reboot / fork / delay / proc_exec |

`fork`は現在スタブ（常に1を返す）ですが、枠組みは用意してあります。

---

## 実装されている主なコマンド

| コマンド | 説明 |
| --- | --- |
| ls / ll / dir | ファイル一覧（すべて同じ動作のエイリアス） |
| pwd | カレントディレクトリ表示 |
| cd <dir> | ディレクトリ移動（引数なしで/へ） |
| mkdir / rmdir | ディレクトリ作成・削除 |
| touch | 空ファイル作成 |
| cat | ファイル内容表示 |
| rm | ファイル削除 |
| cp / mv | コピー・移動（成功時にメッセージ表示） |
| echo | 文字列表示 |
| nano | 行単位の簡易エディタ（.行で保存終了） |
| edit | フルスクリーンエディタ（矢印キー対応） |
| mem | 空きメモリ表示 |
| clear | 画面クリア |
| deploy | 仮想システムファイルの再デプロイ |
| scan | I2Cバスをスキャンして接続機器を一覧表示 |
| history | コマンド履歴を一覧表示 |
| date | RTCから現在日時を表示 |
| gpio <pin> <mode> | GPIO制御（in/out/pullup/high/low/read/analog） |
| fork_test | バックグラウンドプロセスのデモ |
| reboot | 再起動 |
| help [cmd] | ヘルプ表示 |

上記に一致しないコマンド名は、`<name>.c`としてSDカードから探して実行されます（minDosのEXEC相当）。Tab補完も、これらビルトインコマンドとカレントディレクトリのファイル名の両方に対応しています。

---

## 技術的な工夫・こだわりポイント

1. **完全な自己記述性**
   
   OSの主要部分が「自分自身を解釈する言語」で書かれている。
2. **仮想ファイルシステムからのブート**
   
   実在のファイルとしてデプロイ→ロードする形を取っている。
3. **レイヤの分離**
   
   ネイティブ側とインタプリタ側の責任範囲が非常に綺麗。フルスクリーンエディタのような「重い画面制御」だけはネイティブ側に置き、CInterp側は薄いsyscall呼び出しで済ませている。
4. **差分描画によるSPI転送量の削減**
   
   フルスクリーンエディタのカーソル移動や1文字編集は、変更のあったセルだけを直接LCDへ書き込む差分更新にすることで、フル再描画より軽くしている。
5. **構造体サポート**
   
   自己参照ポインタも可能なので、リスト構造なども書ける。

---

## 現時点の制限事項

- マルチプロセスは未実装（`fork`はスタブ）
- メモリ保護やユーザー権限の概念なし
- 浮動小数点非対応
- シェルのコマンドパーサがかなり単純
- I2Cはレジスタ単位のread/writeのみで、複数バイトのバースト転送は未対応
- フルスクリーンエディタはマウス非対応（liveOS2版から移植した際にCH9350関連機能はすべて削除）
- エラー処理が甘い部分がある
- 巨大なソースを毎回コンパイルするため、起動に少し時間がかかる

---

## 今後やりたいこと

- 本格的なマルチタスク（ラウンドロビン or 優先度付き）
- ユーザープログラムのロード＆実行機能のさらなる強化
- I2Cの複数バイトread/write（バースト転送）対応
- ネットワーク対応（SpresenseのLTEアドオンを活用）
- メモリ使用量の可視化ツール
- より本格的なファイルシステム抽象化

---

## おわりに

LinaXは **ただ動くもの**を作るのではなく、**Cインタプリタを核にした、自己記述的な仮想OS**という、かなり尖った方向性で作っています。今回のアップデートで、コマンド履歴・Tab補完・フルスクリーンエディタ・I2Cが揃い、実際に**触って使える**レベルにだいぶ近づいてきました。

公開されている類似プロジェクトを調べた限り、この組み合わせ（c4系インタプリタ + 仮想FSからの自己ブート + Linux風ユーザーランド）は非常に珍しく、ほぼ独自の位置にいると感じています。

まだまだ荒削りですが **マイコンの上でOSっぽいものを、しかも自分で書いたCで動かす**という楽しさは十分に味わえると思います。

興味を持った方は、ぜひコードを触ってみてください。

改善案や「ここもっとこうすれば」という指摘も大歓迎です。

---

### 動作環境

- ボード: Sony Spresense メインボード
- ディスプレイ: ILI9488（480×320）+ LovyanGFX
- ストレージ: microSDカード
- ライブラリ: LovyanGFX, SDHCI, Wire

---

**関連ファイル構成**

```
LinaX.ino
main.hpp
hw.hpp
liveC.hpp      ← Cインタプリタ本体
term.hpp       ← ターミナルエンジン（履歴・Tab補完含む）
kernel.hpp     ← ブートローダ + syscall
edit.hpp       ← フルスクリーンテキストエディタ
fs_images.hpp  ← 仮想システムイメージ（VMLINUZ / LIBC / SH）
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

### 実機

（X 投稿の埋め込み）

（X 投稿の埋め込み）

（X 投稿の埋め込み）

### 移植開始

- ESP32-2432S028 (Cheap Yellow Display)
  
  
  
  （X 投稿の埋め込み）
- M5StackS3
  
  
  
  （X 投稿の埋め込み）

### vm 比較

- liveC vs c4
  
  
  
  （X 投稿の埋め込み）

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

以上です。

<div class="like-wrap"><div data-lyket-type="like" data-lyket-template="twitter" data-lyket-namespace="linalog" data-lyket-color-icon="#b5e853" data-lyket-color-primary="#b5e853" data-lyket-color-text="#eaeaea" data-lyket-color-secondary="#2b2b2b" data-lyket-id="spresense-linax"></div></div>
