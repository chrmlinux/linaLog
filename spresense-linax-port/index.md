---
image: https://chrmlinux.github.io/linaLog/spresense-linax-port/images/img01.jpg
---

# [#SPRESENSE 2026]移植してみたにょ[linaX]

2026年9月12日 13:06
<p class="article-tags">タグ: <a href="..?tag=Arduino">Arduino</a> <a href="..?tag=LIVEC">LIVEC</a> <a href="..?tag=M5Stack">M5Stack</a> <a href="..?tag=VM">VM</a> <a href="..?tag=linux">linux</a> <a href="..?tag=stickS3">stickS3</a> <a href="..?tag=tab5">tab5</a> <a href="..?tag=%E8%87%AA%E4%BD%9COS">自作OS</a></p>

![](images/img01.jpg)

# M5Stack Tab5 や stickS3 に「LinaX」(自作 Pure C 仮想Linux風OS) を移植した話

## LinaXとは

**LinaX** は、ESP32/Spresense系マイコン上で動く自作の「疑似Linuxブート」ミニOSです。

- コアは **CInterp** という自前の C言語サブセット・インタプリタ(c4スタイルのミニコンパイラ+VM)。int/char/ポインタ/構造体/配列/if・while・for・switch/関数定義などをサポートし、union・enum・typedef・float・gotoなどは非対応という、組み込み向けに絞った軽量Cサブセットです。
- 起動時にSDカード(またはSPIFFS)へ3本の仮想ソースファイル `boot_vmlinuz` / `bin_libc` / `bin_sh` を書き出し、それらを読み戻して1つのソースに連結、CInterp上でコンパイル・実行します。
  - `boot_vmlinuz`(カーネル層): `io_puts` / `io_cat` / `io_mkdir` など、ネイティブ syscall を1:1でラップした薄い関数群
  - `bin_libc`(libc層): `linux_ls` / `linux_cp` / `linux_gpio` など、使い勝手の良い高レベルコマンドをカーネル層の上に構築
  - `bin_sh`(シェル層): `main()` のコマンドループ、`help` の実装
- つまり **カーネル → libc → シェル** という本物のLinuxブートに寄せた3層構造を、全部自作Cサブセットのソースコードとして持っている、というのがLinaXの一番の特徴です。SDカード上のこれらのファイルはシェルの `edit` コマンドで実行中に直接編集でき、次回起動時から反映されます。

対応ボードは `config.hpp` の1行のdefineで切り替える設計になっていて、現在は以下をサポートしています。

| ボード | パネル | 備考 |
|---|---|---|
| Spresense | ILI9488 320x480 | SDHCI / RTC内蔵 |
| CYD (ESP32-2432S028) | ILI9341 240x320 | 素のESP32、SRAM 520KBとタイト |
| M5StickS3 | ST7789V2 240x135 | M5GFX自動検出、SPIFFS |
| **M5Stack Tab5(今回)** | 1280x720 IPS | ESP32-P4、M5GFX自動検出、SPIFFS |

## なぜTab5への移植が面白いか

これまでのターゲットはどれも「物理キーボードをUSBシリアル経由でホストPCから叩く」小型ディスプレイ機でした。ところがTab5は

- 画面が **1280x720の5インチIPS** と圧倒的に大きい
- **物理キーボードが存在しない**(タッチのみ)
- ESP32-P4でSRAM 768KB + PSRAM 32MBとメモリに余裕がある

という、これまでのボードと性質がかなり異なるマシンです。この差分を吸収するために、主に3つの手を入れました。

### 1. 画面サイズに応じたテキストスケール切り替え

```cpp
// config.hpp
#if defined(LINAX_BOARD_TAB5)
  #define LINAX_TEXT_SCALE 2
#else
  #define LINAX_TEXT_SCALE 1
#endif
```

小型パネルでは等倍(6x8px)の標準フォントで十分読めますが、Tab5の5インチ・1280x720では小さすぎるため2倍(12x16px)に拡大しています。ターミナル(`term.hpp`)・エディタ(`edit.hpp`)ともにこの `LINAX_TEXT_SCALE` を基準にセル幅/高さを計算する作りにしてあるので、ボード追加時は基本この1行を調整するだけで済みます。

### 2. オンスクリーン仮想キーボード(vkbd.hpp)

物理キーボードがないため、画面下部にドッキングする**タッチ式の仮想キーボード**を新規に実装しました。

- 画面下 `LINAX_KBD_H`(300px)を常時キーボード領域として確保し、ターミナルの描画領域 `LINAX_TERM_H` はその残り(720-300=420px)に自動調整
- `SERIAL_PT` というマクロで「シリアル入力ストリーム」を抽象化しており、Tab5だけは通常の `Serial` ではなく仮想キーボードのストリームクラスに差し替え

```cpp
// config.hpp (Tab5)
#define SERIAL_PT vkbdStream
```

- ただし USB経由のホスト端末からの入力も引き続き使えるよう、`vkbdStream` 内部で「タッチ入力キュー→なければ `Serial` にフォールバック」という二段構えにしてあり、タッチでもホストのキー入力でもシームレスに動きます。
- 3本指トリプルタップ、または `Ctrl+T` でキーボードの表示/非表示をトグル可能。トグル時はターミナル用スプライトのサイズをキーボード分だけ増減させ、`vTerm::init()` で行/列数を再計算しています。

### 3. 実機テストで見つかった細かいバグ

移植中に実際に触ってみて初めて分かった不具合がいくつかありました。

- **エディタ終了後にキーボードを開閉すると表示が崩れる**:
  フルスクリーンエディタ (`edit.hpp`) が `lcd.setTextSize(LINAX_TEXT_SCALE)`(Tab5では2倍)をセットしたまま戻ってきてしまい、それを引きずったまま `vkbd.hpp` のキーボード再描画がラベルを2倍サイズで描いてしまう、というフォント/サイズの状態管理漏れでした。キーボード描画の最初に強制的に等倍へ戻し、末尾で基準サイズに戻す形で解消しました。
- **物理/USBキーボードからの入力を仮想キーボードにも反映**:
  ホスト端末から打鍵した文字にも、タップ時と同じ「該当キーが黄色く光る」フィードバックを出せるようにしました。ただし高速タイピング時にチラつきが気になったため、`config.hpp` に

  ```cpp
  #define LINAX_VKBD_KEY_FLASH 0   // 1でフラッシュ点灯を有効化
  ```

  というフラグを設け、タップ/シリアル入力どちらのフラッシュもこの1つのdefineでON/OFFできるようにしました。

## 使ってみる

`config.hpp` の先頭付近、ボード切り替えマクロを1つだけ有効にしてビルドします。

```cpp
// #define LINAX_TARGET_CYD
// #define LINAX_TARGET_M5STICKS3
#define LINAX_TARGET_TAB5
```

起動すると `LinaX` のASCIIアートバナーとともに `bin_sh` のシェルが立ち上がり、`ls` / `cd` / `edit` / `gpio` / `linaxconfig` などのビルトインコマンドや、SDカード上の `<name>.c` を直接実行できる簡易プロセス起動もそのまま使えます。

## まとめ

- 自作Cサブセットインタプリタの上に、カーネル/libc/シェルの3層を素直に積んだミニ疑似Linux環境「LinaX」
- Tab5移植では「大画面前提のテキストスケール」「物理キーボード非搭載を補う仮想キーボード」「実機で踏んだ表示バグの解消」が主な作業
- ボードごとの差分は基本 `config.hpp` に閉じ込める設計にしてあるので、次のボード追加もそれほど大変ではなさそうです

質問・ツッコミ歓迎です。

## 実機
- <blockquote class="twitter-tweet" data-theme="dark"><a href="https://twitter.com/chrmlinux03/status/2098621417835442632"></a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
- <blockquote class="twitter-tweet" data-theme="dark"><a href="https://twitter.com/chrmlinux03/status/2098291277456667017"></a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

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

<div class="like-wrap"><div data-lyket-type="like" data-lyket-template="twitter" data-lyket-namespace="linalog" data-lyket-color-icon="#b5e853" data-lyket-color-primary="#b5e853" data-lyket-color-text="#eaeaea" data-lyket-color-secondary="#2b2b2b" data-lyket-id="spresense-linax-port"></div></div>
