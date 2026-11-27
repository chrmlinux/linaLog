---
image: https://chrmlinux.github.io/linaLog/lindos/images/img01.jpg
---

# [#SPRESENSE 2026]OS作ってみたにょ[LinDOS]

2026年9月7日 17:13

![](images/img01.jpg)

# マイコンで動く「仮想16bit CPU + MS-DOS風OS」LinDosを作った話
〜 アセンブリで書いたシェルが、本物のDOSみたいに動く 〜

## はじめに

「マイコンの上で、昔のMS-DOSみたいなものを動かしたい」  
「しかもシステム自体をアセンブリ風の言語で書いて、仮想CPUで実行したい」

そんな想いから生まれたのが **LinDos** (スペイン語で可愛い等々)です。

LinDosは、Sony Spresense上で動作する **仮想16bit CPUベースのMS-DOS風OS** です。  
特徴は以下の通り：

- 独自の仮想16bit CPU（vCPU）を実装
- システムを **IO.SYS / DOS.SYS / CMD.SYS** の3層構造で記述（本物のDOSを模している）
- システムファイルはすべてアセンブリ風言語で書かれている
- シェルから `.asm` と `.c` の両方のユーザープログラムを実行可能
- ファイル操作・ディレクトリ操作・ワイルドカード対応など、かなり実用的なコマンドを実装
- ネイティブ側はハードウェア抽象化と仮想CPUの実行に徹している

要するに、**「仮想CPUの上で動く、自己ブートストラップするMS-DOS風環境」** です。

本記事では、設計思想から実装の詳細、ブートストラップの仕組みまでをかなり深掘りして解説します。

---

## 全体アーキテクチャ

LinDosは明確にレイヤ分けされています。

| 層 | 説明 | 実装 |
| --- | --- | --- |
| CMD.SYS（シェル層） | ユーザー入力を受け取り、コマンドを解釈 | アセンブリ |
| DOS.SYS（カーネル層） | 論理コードをIO.SYS向けコードに変換 | アセンブリ |
| IO.SYS（ハードウェア抽象化層） | SEROUT / SERNL / GFXCLR / SERIN などを呼ぶ | アセンブリ |
| vCPU（仮想16bit CPU） | 独自の命令セットを実行 | ネイティブC++ |
| LiveC（Cインタプリタ） | .c ファイルを実行するときに使用 | ネイティブC++ |
| kernel.hpp（ブートローダ + ビルトインコマンド） | | ネイティブC++ |
| term.hpp（ターミナルエンジン） | | ネイティブC++ |
| hw.hpp（LovyanGFX + ILI9488） | | ネイティブC++ |

ポイントは、**上3層がすべて仮想CPUが実行するアセンブリソース**であることです。  
ネイティブ側は「仮想CPUを用意して、仮想ファイルシステムからシステムをロードする」役割に徹しています。

---

## ブートストラップの仕組み（詳細）

ここがLinDosで一番面白い部分です。

### 起動シーケンス

1. Arduinoの`setup()` → `osSetup()`が呼ばれる
2. ハードウェア初期化（LCD・スプライト・RTC・Serial）
3. ターミナル初期化＆タイトル表示
4. SDカード（またはSPIFFS）をマウント
5. **システムファイルをSDに書き込む**（deploy）
6. `/IO.SYS` `/DOS.SYS` `/CMD.SYS` を読み込む
7. 3つのソースを結合し、先頭に `JMP SHELL_START` を付ける
8. 仮想CPU（vCPU）に渡して実行開始
9. `CMD.SYS` の `SHELL_START` からシェルが動き出す

### 実際のコード（抜粋）

```cpp
void osSetup() {
  // ... ハードウェア初期化 ...

  deploySystemFiles();  // /IO.SYS /DOS.SYS /CMD.SYS を書き込む

  std::string ioSys  = loadFileFromSD("/IO.SYS", okIO);
  std::string dosSys = loadFileFromSD("/DOS.SYS", okDos);
  std::string cmdSys = loadFileFromSD("/CMD.SYS", okCmd);

  // 結合して仮想メモリイメージを作る
  std::string coreMemoryImage =
    std::string("JMP SHELL_START\n") +
    "; --- IO LAYER ---\n" + ioSys + "\n" +
    "; --- KERNEL LAYER ---\n" + dosSys + "\n" +
    "; --- SHELL LAYER ---\n" + cmdSys + "\n";

  vCpu.onHwOp = handleHwOp;
  vCpu.run(coreMemoryImage);   // ここでシェルが動き出す
}
```

**なぜ一度SDに書いて読み直すのか？**

- 「仮想ファイルシステムからロードする」というOSらしい形を取っている
- 将来的にユーザーがシステムファイルを書き換えて再起動できる設計の土台になる
- 本物のDOSが起動時にシステムファイルをロードする流れを再現している

---

## 仮想CPU（vCPU）の設計

### レジスタ構成

シンプルに4本だけです。

- `AX` / `BX` / `CX` / `DX`（すべて32bit整数として実装）

### 主な命令セット

| 命令 | 説明 |
|------|------|
| `MOV reg, val` | レジスタに値を代入 |
| `ADD reg, val` | 加算 |
| `SUB reg, val` | 減算 |
| `CMP val1, val2` | 比較（結果をcmpFlagに保存） |
| `JMP label` | 無条件ジャンプ |
| `JEQ / JZ label` | 等しい場合ジャンプ |
| `JNE / JNZ label` | 等しくない場合ジャンプ |
| `CALL label` | サブルーチン呼び出し |
| `RET` | サブルーチンから戻る |
| `INT n` | ソフトウェア割り込み（拡張用） |
| `CMPS str` | 入力バッファと文字列を比較 |
| `SEROUT val` | 数値を画面に出力 |
| `SERNL` | 改行 |
| `GFXCLR` | 画面クリア |
| `SERIN` | 1行入力 |
| `DIRLIST` | ディレクトリ一覧 |
| `EXEC` | コマンド実行（ビルトイン or .asm/.c） |
| `HLT / HALT` | 停止 |

命令数を極限まで絞っているので、実装が軽く、デバッグもしやすいのが特徴です。

### ラベルとコールスタック

- ラベルは行末の `:` で定義
- `CALL` / `RET` でサブルーチンを実現（コールスタックを内部で管理）

---

## システムファイルの3層構造

### 1. IO.SYS（ハードウェア抽象化層）

ハードウェアに直接触れる唯一の層です。

```asm
DRV_ENTRY:
    CMP AX, 11
    JEQ DRV_PUTNUM      ; 11 = 数値表示
    CMP AX, 12
    JEQ DRV_NEWLINE     ; 12 = 改行
    CMP AX, 13
    JEQ DRV_CLS         ; 13 = 画面クリア
    CMP AX, 14
    JEQ DRV_READLINE    ; 14 = 1行入力
    CMP AX, 15
    JEQ DRV_DIRLIST     ; 15 = ディレクトリ一覧
    RET
```

### 2. DOS.SYS（カーネル層）

論理コード（1〜5）をIO.SYS向けのコード（11〜15）に変換します。

```asm
SYS_ENTRY:
    CMP AX, 1
    JEQ KRN_PUTNUM
    CMP AX, 2
    JEQ KRN_NEWLINE
    ; ...
KRN_PUTNUM:
    MOV AX, 11
    CALL DRV_ENTRY
    RET
```

この変換層があることで、シェル側はハードウェアの詳細を知らなくて済みます。

### 3. CMD.SYS（シェル層）

ユーザーと対話する最上位層です。

```asm
SHELL_START:
SHELL_LOOP:
    MOV AX, 4           ; 1行入力
    CALL SYS_ENTRY

    CMPS CLS
    JEQ CMD_CLS
    CMPS VER
    JEQ CMD_VER
    EXEC                ; ビルトインでなければ .asm / .c を探す
    JMP SHELL_LOOP

CMD_CLS:
    MOV AX, 3
    CALL SYS_ENTRY
    JMP SHELL_LOOP
```

非常にシンプルですが、「入力 → 判定 → 実行」の基本的なシェルの流れがアセンブリだけで実現されています。

---

## コマンド実行の仕組み（EXEC）

ユーザーがコマンドを入力すると、以下の順番で処理されます。

1. **ビルトインコマンド**を試す（`tryBuiltinCommand`）
   - `ls` / `dir` / `cd` / `mkdir` / `cp` / `del` / `cat` / `echo` / `mem` / `date` / `reboot` など
2. 見つからなければ **`/NAME.asm`** を探して vCPU で実行
3. それでもなければ **`/NAME.c`** を探して LiveC で実行
4. どれもなければ `Bad command or file name`

この二段構え（アセンブリとC）がLinDosの大きな特徴です。

---

## 実装されている主なコマンド

| コマンド | 説明 |
|----------|------|
| `ls` / `dir [pattern]` | ファイル一覧（`*` `?` ワイルドカード対応） |
| `cd <dir>` | ディレクトリ移動（`..` や `/` も可） |
| `mkdir` / `rmdir` | ディレクトリ作成・削除 |
| `cp` / `mv` / `ren` | コピー・移動・リネーム |
| `del` | ファイル削除（ワイルドカード可） |
| `cat` / `type` | ファイル内容表示 |
| `echo` | 文字列表示 |
| `mem` | 空きメモリ表示 |
| `date` / `time` | RTCから日時表示 |
| `cls` | 画面クリア |
| `ver` | バージョン表示 |
| `reboot` | 再起動 |
| `pause` | キー入力待ち |
| `help [cmd]` | ヘルプ表示 |

タブ補完も実装されています。

---

## 技術的な工夫・こだわりポイント

1. **本物のDOSの階層を忠実に再現**  
   IO.SYS → DOS.SYS → CMD.SYS という構造をアセンブリで実現している。

2. **仮想CPUの命令を極限まで絞った**  
   必要最低限の命令だけでシェルが動くことを証明している。

3. **.asm と .c の両対応**  
   軽い処理はアセンブリ、複雑な処理はCで書けるハイブリッド構成。

4. **ワイルドカード対応のファイル操作**  
   `del *.txt` や `ls *.c` などが普通に使える。

5. **スタックに優しい設計**  
   vCPUやCInterpをヒープに確保するなど、深い呼び出しでもスタックオーバーフローしにくい工夫をしている。

---

## 現時点の制限事項

- vCPUの命令セットが少ないため、複雑なアセンブリプログラムは書きにくい
- マルチタスクは未実装
- メモリ保護やユーザー権限の概念なし
- アセンブリの1行が64文字制限（コメントは短く英語で書く必要あり）
- エラー処理がまだ甘い部分がある

---

## 今後やりたいこと

- vCPUの命令セット拡張（特にメモリ操作系）
- 本格的なマルチタスク
- より高機能なアセンブリプログラムのサンプル追加
- ネットワーク対応（SpresenseのLTEアドオン活用）
- 簡易テキストエディタの実装
- システムファイルのホットリロード

---

## おわりに

LinDosは「ただ動くもの」を作るのではなく、**「仮想CPUの上で本物のDOSの構造を再現する」**という、かなり尖った方向性で作っています。

アセンブリでシェルを書き、それを仮想CPUで動かし、さらにCプログラムも実行できるというハイブリッドな構成は、公開されているプロジェクトの中でもかなり珍しいと思います。

まだまだ荒削りですが、「マイコンの上で昔のPCみたいなものを動かす」という楽しさは十分に味わえるはずです。

興味を持った方は、ぜひコードを触ってみてください。  
改善案や「ここもっとこうすれば」という指摘も大歓迎です。

---

### 動作環境

- ボード: Sony Spresense メインボード
- ディスプレイ: ILI9488（480×320）+ LovyanGFX
- ストレージ: microSDカード（またはSPIFFS）
- ライブラリ: LovyanGFX, SDHCI

---

**関連ファイル構成**

```
linDos.ino
main.hpp
hw.hpp
cpu.hpp          ← 仮想16bit CPU本体
term.hpp         ← ターミナルエンジン
liveC.hpp        ← Cインタプリタ（.c実行用）
kernel.hpp       ← ブートローダ + ビルトインコマンド
writeFile.hpp    ← システムファイルソース（IO.SYS / DOS.SYS / CMD.SYS）
```
## 実機
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

