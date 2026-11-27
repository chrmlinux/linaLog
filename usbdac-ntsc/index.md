---
image: https://chrmlinux.github.io/linaLog/usbdac-ntsc/images/img01.png
---

# [USBDAC]イヤホンジャックでNTSC映像を出してみたにょ[CVBS]

2026年6月17日 13:27

![](images/img01.png)

# はじめに

3.5mmイヤホンジャックから、NTSCアナログ映像信号を出力することに成功しました。
使ったのは **384kHz / 32bit / 2ch の PCMオーディオストリーム** だけです。

## 仕組み

NTSCビデオ信号は、本質的には「タイミングが決まったアナログ波形」です。
水平同期パルス・ブランキングレベル・ピクセル輝度値を、PCMサンプルに直接エンコードすることで、オーディオ出力から正規のNTSC信号を再現できます。

## スペック

| 項目 | 値 |
|------|----|
| サンプルレート | 384,000 Hz |
| ビット深度 | 32bit signed PCM |
| チャンネル | 2ch ステレオ（Left = 映像信号、Right = 同期リファレンス） |
| 水平ライン数 | 262ライン（NTSC標準） |
| 有効解像度 | 19 × 120ピクセル（縦2倍ストレッチ） |
| フレームレート | 約60fps |

## 流したもの

**「Bad Apple!!」**

あらゆるハードウェアで動かされてきた伝説のデモ映像。
ついにオーディオDACのイヤホンジャックでも動きました。

（X 投稿の埋め込み）
## ポイント

- ビデオ出力ハードウェア：なし
- HDMI：なし
- コンポジットビデオ回路：なし

ただのオーディオサンプルを、正確なタイミングでイヤホンジャックから出力するだけ。
HiresAudioPlayerから映像が出ます。 

**2030年代のテクノロジーを逆走していますが。それでも動きます。**

## 使用ツール

- Python（信号生成・動画エンコード）
- OpenCV（動画フレーム処理）
- AZLA オーディオデバイス（384kHz DAC出力）
- みさきフォント 8×12（日本語文字描画）

詳細は 👉[HackDay.io](https://hackaday.io/project/205934-spresense-audio-jack-as-ntsc-video-output)　

<div class="like-wrap"><div data-lyket-type="like" data-lyket-template="twitter" data-lyket-namespace="linalog" data-lyket-color-icon="#b5e853" data-lyket-color-primary="#b5e853" data-lyket-color-text="#eaeaea" data-lyket-color-secondary="#2b2b2b" data-lyket-id="usbdac-ntsc"></div></div>
