---
image: https://chrmlinux.github.io/linaLog/spresense-audiohack-kanji/images/img01.png
---

# [SPRESENSE 2026]AudioHack漢字に対応したにょ[CVBS]

2026年6月16日 16:51

![](images/img01.png)



（X 投稿の埋め込み）
```
import struct
import sys
import os
from misaki_font8x12L import get_glyph as _get_glyph

# horizontal punctuation correction (move to bottom of glyph)
HORI_PUNCT = {
    '。': [0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x60, 0xA0, 0xA0, 0x60],
    '、': [0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x80, 0x40, 0x40, 0x00],
    '・': [0x00, 0x00, 0x00, 0x00, 0x00, 0x60, 0x60, 0x00, 0x00, 0x00, 0x00, 0x00],
}

def get_glyph(char):
    if char in HORI_PUNCT:
        return HORI_PUNCT[char]
    return _get_glyph(char)

# constants
SAMPLE_RATE   = 188811
NTSC_H_FREQ   = 15734.26
NTSC_H        = 262
NTSC_W        = round(SAMPLE_RATE / NTSC_H_FREQ)  # 12

SYNC_W        = 2
BLANK_W       = 3
VIDEO_W       = NTSC_W - SYNC_W - BLANK_W          # 9 dots (1 sample = 1 dot)

FONT_W        = 8                                   # font width
FONT_H        = 12                                  # font height (8x12L)
CHAR_SPACE    = 1                                   # space between characters
LINE_REPEAT   = 16                                  # scanlines per font row

PCM_SYNC      = 0x000000                            # sync level
PCM_BLACK     = 0x100000                            # black level
PCM_WHITE     = 0x7FFFFF                            # white level

SCROLL_SPEED  = 6                                   # frames per 1 dot scroll (lower = faster)

def build_scroll_buffer(string):
    """Render full string into a wide scroll buffer"""
    padding = VIDEO_W                               # leading/trailing blank
    total_w = padding + len(string) * (FONT_W + CHAR_SPACE) + padding
    buf     = [[0] * total_w for _ in range(FONT_H)]

    x = padding
    for char in string:
        glyph = get_glyph(char)
        for row in range(FONT_H):
            for col in range(FONT_W):
                bit = (glyph[row] >> (7 - col)) & 1
                if bit and (x + col) < total_w:
                    buf[row][x + col] = 1
        x += FONT_W + CHAR_SPACE

    return buf, total_w

def generate_ntsc_scroll(string, scroll_speed=SCROLL_SPEED):
    fname = "kanji.raw"
    fps   = SAMPLE_RATE / (NTSC_W * NTSC_H)

    buf, total_w = build_scroll_buffer(string)
    scroll_steps = total_w - VIDEO_W
    total_frames = scroll_steps * scroll_speed

    print(f"String   : {string}")
    print(f"VIDEO_W  : {VIDEO_W} dots")
    print(f"FONT     : {FONT_W}x{FONT_H}L")
    print(f"FPS      : {fps:.3f}")
    print(f"Frames   : {total_frames}")
    print(f"Duration : {total_frames / fps:.1f} sec")
    print(f"Generating {fname}...")

    with open(fname, "wb") as f:
        for frame in range(total_frames):
            if frame % 100 == 0:
                pct = frame / total_frames * 100
                sys.stdout.write(f"\r{pct:.1f}% ({frame}/{total_frames})")
                sys.stdout.flush()

            offset = frame // scroll_speed          # current scroll position (dots)

            for y in range(NTSC_H):
                if y < 9:
                    # vertical sync period
                    for s in range(NTSC_W):
                        f.write(struct.pack("<i", PCM_SYNC)[:3])   # L: sync
                        f.write(struct.pack("<i", PCM_SYNC)[:3])   # R: sync
                    continue
                elif y >= 240:
                    # blanking area
                    for s in range(NTSC_W):
                        f.write(struct.pack("<i", PCM_BLACK)[:3])  # L: black
                        f.write(struct.pack("<i", PCM_BLACK)[:3])  # R: black
                    continue

                start_y = (240 - LINE_REPEAT * FONT_H) // 2 + 9  # vertical centering
                v_row   = (y - start_y) // LINE_REPEAT  # font row index (0~11)

                for s in range(NTSC_W):
                    if s < SYNC_W:
                        l = PCM_SYNC                # L: horizontal sync
                        r = PCM_BLACK               # R: black
                    elif s < SYNC_W + BLANK_W:
                        l = PCM_BLACK               # L: blanking
                        r = PCM_BLACK               # R: black
                    else:
                        v_col = s - (SYNC_W + BLANK_W)  # 0~8
                        dot_x = offset + v_col           # position in scroll buffer

                        if 0 <= v_row < FONT_H and 0 <= dot_x < total_w:
                            pixel = buf[v_row][dot_x]
                        else:
                            pixel = 0

                        l = PCM_WHITE if pixel else PCM_BLACK  # L: video
                        r = PCM_BLACK                          # R: black

                    f.write(struct.pack("<i", l)[:3])
                    f.write(struct.pack("<i", r)[:3])

    size = os.path.getsize(fname)
    sys.stdout.write(f"\r100.0% ({total_frames}/{total_frames})\n")
    print(f"Done: {fname} ({size:,} bytes)")

if __name__ == "__main__":
    user_input = input("Enter string (hiragana/katakana/kanji OK): ")
    if user_input:
        generate_ntsc_scroll(user_input)
    else:
        print("No input provided.")
```
