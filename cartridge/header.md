# [カートリッジヘッダ](https://problemkaputt.de/fullsnes.htm#snescartridgeromheader)

カートリッジのヘッダは、SNESのメモリでは00FFxxh(例外ベクタの近く)にマッピングされています。

ROMイメージでは、ROMヘッダは、オフセット 007Fxxh (LoROM)、00FFxxh (HiROM)、または 40FFxxh (ExHiROM) にあります。

もし`(imagesize AND 3FFh)=200h`の場合、つまり SWC/UFO/その他のコピー機からの追加ヘッダがある場合は、このオフセットに +200h を加えます。

## カートリッジヘッダ (Area FFC0h..FFCFh)

```
  FFC0h  カートリッジのタイトル(21バイト、大文字のAscii、余った部分はスペースでパディングされます)
  FFC0h  タイトルの最初のバイト(頭文字、海賊版のXin1カートリッジでは 5Chのfarjumpオペコードが格納されます)
  FFD4h  タイトルの最後のバイト (00h にすると初期型拡張ヘッダがあることを指示します)
  FFD5h  Rom Makeup / ROM Speed and Map Mode (後述)
  FFD6h  チップセット (カートリッジの基盤構成、後述)
  FFD7h  ROM size (1 SHL n) Kbytes (usually 8=256KByte .. 0Ch=4MByte)
          Values are rounded-up for carts with 10,12,20,24 Mbits
  FFD8h  RAM size (1 SHL n) Kbytes (usually 1=2Kbyte .. 5=32Kbyte) (0=None)
  FFD9h  製造地域コード (これによってPAL/NTSCも決まります) (後述)
  FFDAh  製造元ID  (00h=不明/自作, 01h=Nintendo, etc.) (33h=New)
  FFDBh  バージョン (0スタート、つまり v1.0 = 00h)
  FFDCh  Checksum complement (same as below, XORed with FFFFh)
  FFDEh  チェックサム (all bytes in ROM added together; assume [FFDC-F]=FF,FF,0,0)
```

## 拡張ヘッダ (Area FFB0h..FFBFh) (newer carts only)

### 初期型拡張ヘッダ (1993) (when [FFD4h]=00h; Last byte of Title=00h):

```
  FFB0h  予約領域 (15バイト, 全部0)
  FFBFh  チップセットのサブタイプ (基本的に0、[FFD6h]=Fxhの場合のみ参照)
```

### 後期型拡張ヘッダ (1994) (when [FFDAh]=33h; Old Maker Code=33h):

```
  FFB0h  製造元ID (ASCII2文字, 例: "01"=Nintendo)
  FFB2h  ゲームコード  (ASCII4文字) (古いタイプはASCII2文字で20h,20hでパディングされています)
  FFB6h  予約領域   (6バイト、全部0)
  FFBCh  Expansion FLASH Size (1 SHL n) Kbytes (used in JRA PAT)
  FFBDh  Expansion RAM Size (1 SHL n) Kbytes (in GSUn games) (without battery?)
  FFBEh  Special Version      (usually zero) (eg. promotional version)
  FFBFh  チップセットのサブタイプ (基本的に0、[FFD6h]=Fxhの場合のみ参照)
```

注意: 初期型拡張ヘッダは ST010/11 ゲーム で飲み使われていました。

> ST0xxは セタ製のチップで、主にAIを強化する目的で使われていました。

4文字のゲームコードの最初の文字が "Z"の場合、そのカートリッジにはサテラビューのようなデータパック/フラッシュカートリッジスロットがあります。これは、"Zxxx"の4文字コードにのみ適用され、スペース埋めされた古い2文字コード(`Zx  `)には適用されません。

## Cartridge Header Variants

The BS-X Satellaview FLASH Card Files, and Sufami Turbo Mini-Cartridges are using similar looking (but not fully identical) headers (and usually the same .SMC file extension) than normal ROM cartridges. Detecting the content of .SMC files can be done by examining ID Strings (Sufami Turbo), or differently calculated checksum values (Satellaview). For details, see:

[SNES Cart Satellaview (satellite receiver & mini flashcard)](https://problemkaputt.github.io/fullsnes.htm#snescartsatellaviewsatellitereceiverminiflashcard)
[SNES Cart Sufami Turbo (Mini Cartridge Adaptor)](https://problemkaputt.github.io/fullsnes.htm#snescartsufamiturbominicartridgeadaptor)

Homebrew games (and copiers & cheat devices) are usually having several errors in the cartridge header (usually no checksum, zero-padded title, etc), they should (hopefully) contain valid entryoints in range 8000h..FFFEh. Many Copiers are using 8Kbyte ROM bank(s) - in that special case the exception vectors are located at offset 1Fxxh within the ROM-image.

## 例外ベクタ (Area FFE0h..FFFFh)

```
  FFE0h  Zerofilled (or ID "XBOO" for WRAM-Boot compatible files)
  FFE4h  COP vector     (65C816 mode) (COP opcode)
  FFE6h  BRK vector     (65C816 mode) (BRK opcode)
  FFE8h  ABORT vector   (65C816 mode) (not used in SNES)
  FFEAh  NMI vector     (65C816 mode) (SNES V-Blank Interrupt)
  FFECh  ...
  FFEEh  IRQ vector     (65C816 mode) (SNES H/V-Timer or External Interrupt)
  FFF0h  ...
  FFF4h  COP vector     (6502 mode)
  FFF6h  ...
  FFF8h  ABORT vector   (6502 mode) (not used in SNES)
  FFFAh  NMI vector     (6502 mode)
  FFFCh  RESET vector   (6502 mode) (CPU is always in 6502 mode on RESET)
  FFFEh  IRQ/BRK vector (6502 mode)
```

Note: Exception Vectors are variable in SA-1 and CX4, and fixed in GSU.

## Text Fields

The ASCII fields can use chr(20h..7Eh), actually they are JIS (with Yen instead backslash).

## ROM Size / Checksum Notes

The ROM size is specified as "(1 SHL n) Kbytes", however, some cartridges contain "odd" sizes:

- Game uses 2-3 ROM chips (eg. one 8MBit plus one 2MBit chip)
- Game originally designed for 2 ROMs, but later manufactured as 1 ROM (?)
- Game uses a single 24MBit chip (23C2401)

In all three cases the ROM Size entry in \[FFD7h\] is rounded-up. In memory, the "bigger half" is mapped to address 0, followed by the "smaller half", then followed by mirror(s) of the smaller half. Eg. a 10MBit game would be rounded to 16MBit, and mapped (and checksummed) as "8Mbit + 4x2Mbit". In practice:

タイトル | ハードウェア | サイズ | チェックサム
-- | -- | -- | --
  Dai Kaiju Monogatari 2 (J)  | ExHiROM+S-RTC | 5MB | 4MB + 4 x Last 1MB
  Tales of Phantasia (J) | ExHiROM | 6MB | \<???\>
  Star Ocean (J) | LoROM+S-DD1 | 6MB | 4MB + 2 x Last 2MB
  Far East of Eden Zero (J) | HiROM+SPC7110+RTC | 5MB | 5MB
  Momotaro Dentetsu Happy (J) | HiROM+SPC7110 | 3MB | 2 x 3MB
  Sufami Turbo BIOS | LoROM in Minicart | xx | without checksum
  Sufami Turbo Games | LoROM in Minicart | xx | without checksum
  Dragon Ball Z - Hyper Dimension | LoROM+SA-1 | 3MB | Overdump 4MB
  SD Gundam GNext (J) | LoROM+SA-1 | 1.5MB | Overdump 2MB
  Megaman X2 | LoROM+CX4 | 1.5MB | Overdump 2MB
  BS Super Mahjong Taikai (J) | BS | Overdump/Mirr+Empty | ?

SPC7110タイトル | ROMサイズ(ヘッダ値) | チェックサム
-- | -- | --
Super Power League 4        | 2MB (rounded to 2MB) | 1x(All 2MB)
Momotaro Dentetsu Happy (J) | 3MB (rounded to 4MB) | 2x(All 3MB)
Far East of Eden Zero (J)   | 5MB (rounded to 8MB) | 1x(All 5MB)

On-chip ROM contained in external CPUs (DSPn,ST01n,CX4) is NOT counted in the ROM size entry, and not included in the checksum.

Homebrew files often contain 0000h,0000h or FFFFh,0000h as checksum value.

## ROM Speed and Map Mode (FFD5h)

```
  Bit7-6 常に0
  Bit5   常に1 (maybe meant to be MSB of bit4, for "2" and "3" MHz)
  Bit4   Speed (0=Slow, 1=Fast)              (Slow 200ns, Fast 120ns)
  Bit3-0 Map Mode
```

Map Mode can be:

```
  0=LoROM/32K Banks             Mode 20 (LoROM)
  1=HiROM/64K Banks             Mode 21 (HiROM)
  2=LoROM/32K Banks + S-DD1     Mode 22 (mappable) "Super MMC"
  3=LoROM/32K Banks + SA-1      Mode 23 (mappable) "Emulates Super MMC"
  5=HiROM/64K Banks             Mode 25 (ExHiROM)
  A=HiROM/64K Banks + SPC7110   Mode 25? (mappable)
```

Note: ExHiROM is used only by "Dai Kaiju Monogatari 2 (JP)" and "Tales of Phantasia (JP)".

## チップセット (ROM/RAM information on cart) (FFD6h) (and some subclassed via FFBFh)

```
  00h     ROM
  01h     ROM+RAM
  02h     ROM+RAM+Battery
  x3h     ROM+Co-processor
  x4h     ROM+Co-processor+RAM
  x5h     ROM+Co-processor+RAM+Battery
  x6h     ROM+Co-processor+Battery
  x9h     ROM+Co-processor+RAM+Battery+RTC-4513
  xAh     ROM+Co-processor+RAM+Battery+overclocked GSU1 ? (Stunt Race)
  x2h     Same as x5h, used in "F1 Grand Prix Sample (J)" (?)
  0xh     Co-processor is DSP    (DSP1,DSP1A,DSP1B,DSP2,DSP3,DSP4)
  1xh     Co-processor is GSU    (MarioChip1,GSU1,GSU2,GSU2-SP1)
  2xh     Co-processor is OBC1
  3xh     Co-processor is SA-1
  4xh     Co-processor is S-DD1
  5xh     Co-processor is S-RTC
  Exh     Co-processor is Other  (Super Gameboy/Satellaview)
  Fxh.xxh Co-processor is Custom (subclassed via [FFBFh]=xxh)
  Fxh.00h Co-processor is Custom (SPC7110)
  Fxh.01h Co-processor is Custom (ST010/ST011)
  Fxh.02h Co-processor is Custom (ST018)
  Fxh.10h Co-processor is Custom (CX4)
```

実際には以下の値が使用されます。

```
  00h     ROM             ;if gamecode="042J" --> ROM+SGB2
  01h     ROM+RAM (if any such produced?)
  02h     ROM+RAM+Battery ;if gamecode="XBND" --> ROM+RAM+Batt+XBandModem
                          ;if gamecode="MENU" --> ROM+RAM+Batt+Nintendo Power
  03h     ROM+DSP
  04h     ROM+DSP+RAM (no such produced)
  05h     ROM+DSP+RAM+Battery
  13h     ROM+MarioChip1/ExpansionRAM (and "hacked version of OBC1")
  14h     ROM+GSU+RAM                    ;\ROM size up to 1MByte -> GSU1
  15h     ROM+GSU+RAM+Battery            ;/ROM size above 1MByte -> GSU2
  1Ah     ROM+GSU1+RAM+Battery+Fast Mode? (Stunt Race)
  25h     ROM+OBC1+RAM+Battery
  32h     ROM+SA1+RAM+Battery (?) "F1 Grand Prix Sample (J)"
  34h     ROM+SA1+RAM (?) "Dragon Ball Z - Hyper Dimension"
  35h     ROM+SA1+RAM+Battery
  43h     ROM+S-DD1
  45h     ROM+S-DD1+RAM+Battery
  55h     ROM+S-RTC+RAM+Battery
  E3h     ROM+Super Gameboy      (SGB)
  E5h     ROM+Satellaview BIOS   (BS-X)
  F5h.00h ROM+Custom+RAM+Battery     (SPC7110)
  F9h.00h ROM+Custom+RAM+Battery+RTC (SPC7110+RTC)
  F6h.01h ROM+Custom+Battery         (ST010/ST011)
  F5h.02h ROM+Custom+RAM+Battery     (ST018)
  F3h.10h ROM+Custom                 (CX4)
```

## 製造地域コード (FFD9h)

製造国が決まるとPAL/NTSCのどちらであるかも決まります。

値 | FFD9h | 地域 | PAL/NTSC
-- | ---- | ---- | -- 
00h | - | 全世界向け(SGBなど) | any
00h | J | 日本 | NTSC
01h | E | アメリカ、カナダ | NTSC
02h | P | ヨーロッパ、オセアニア、アジア | PAL
03h | W | スウェーデン、スカンジナビア | PAL
04h | - | フィンランド | PAL
05h | - | デンマーク | PAL
06h | F | フランス | SECAM, PAL-like 50Hz
07h | H | オランダ | PAL
08h | S | スペイン | PAL
09h | D | ドイツ、オーストラリア、スイス | PAL
0Ah | I | イタリア | PAL
0Bh | C | 中国、香港 | PAL
0Ch | - | インドネシア | PAL
0Dh | K | 韓国 | NTSC
0Eh | A | Common (?) | ?
0Fh | N | カナダ | NTSC
10h | B | ブラジル | PAL-M, NTSC-like 60Hz
11h | U | オーストラリア | PAL
12h | X | その他 | ?
13h | Y | その他 | ?
14h | Z | その他 | ?


FFD9hの項目は、`[FFD9h]`の値とゲームコードの4文字目を表しています。(製造地域コードが決まると`[FFD9h]`とゲームコードの4文字目が一意に定まるものがある)

## ゲームコード (FFB2h)

この領域は`[FFDAh]=33h`のときに利用されます。

```
  "xxxx"  Normal 4-letter code (usually "Axxx") (or "Bxxx" for newer codes)
  "xx  "  Old 2-letter code (space padded)
  "042J"  Super Gameboy 2
  "MENU"  Nintendo Power FLASH Cartridge Menu
  "Txxx"  NTT JRA-PAT and SPAT4 (SFC Modem BIOSes)
  "XBND"  X-Band Modem BIOS
  "Zxxx"  Special Cartridge with satellaview-like Data Pack Slot
```

ゲームコードが2文字のみのもの、"MENU"/"XBND"以外の、ゲームコードは、4文字目が製造地域を表しています。
