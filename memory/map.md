# 🗺 メモリマップ

SNESは24bit幅(`000000h-FFFFFFh`)のアドレスバスを持っています。

この24bitのアドレスは、8bitのバンク番号(`00h-FFh`)と16bitのオフセット(`0000h-FFFFh`)によく分けられます。

これらのバンクの中には、`0000h..7FFFh=システムエリア`、`8000h..FFFFh=カートリッジROM`と、32KBずつ2つに分割されているものもあります。さらに、メモリはWS1とWS2のエリアに分かれており、これらのエリアでは異なるウェイトステートを設定することができます。

## メモリマップ全体

バンク | オフセット | 内容 | 動作速度
-- | -- | -- | -- 
00h-3Fh | 0000h-7FFFh | システム領域 (8K WRAM, I/O Ports, Expansion)   | 後述
00h-3Fh | 8000h-FFFFh  | WS1 LoROM (最大 2048 KB) (64x32K)         | 2.68MHz
00h     | FFE0h-FFFFh  | 例外ベクタ (Reset,Irq,Nmi,etc.)   | 2.68MHz
40h-7Dh | 0000h-FFFFh  | WS1 HiROM (最大 3968 KB) (62x64K)         | 2.68MHz
7Eh-7Fh | 0000h-FFFFh  | WRAM (Work RAM, 128 KB) (2x64K)          | 2.68MHz
80h-BFh | 0000h-7FFFh  | システム領域 (8K WRAM, I/O Ports, Expansion)  | 後述
80h-BFh | 8000h-FFFFh  | WS2 LoROM (最大 2048 KB) (64x32K)         | max 3.58MHz
C0h-FFh | 0000h-FFFFh  | WS2 HiROM (最大 4096 KB) (64x64K)         | max 3.58MHz

内部メモリ領域は、WRAMとメモリマップドI/Oポートです。

外部メモリ領域には、LoROM、HiROM、Expansionの各領域があります。

CPUのアドレス空間にマッピングされていない、I/Oを通してアクセス可能な、追加メモリ領域は以下の通りです。

```
  OAM          (512+32 B) (256+16 words)
  VRAM         (64 KB)    (32 Kwords)
  Palette      (512 B)    (256 words)
  Sound RAM    (64 KB)
  Sound ROM    (64 B BIOS Boot ROM)
```

## システム領域(バンク: 00-3Fh, 80-BFh)

`0x00xxxx..3Fxxxx`, `0x80xxxx-BFxxxx`部分です。(`xxxx=0000..7FFFh`)

オフセット | 内容 | 動作速度
-- | -- | --
0000h-1FFFh | 7E0000h-7E1FFFh(WRAMの先頭8KB)のミラー | 2.68MHz
2000h-20FFh | 不使用                                | 3.58MHz
2100h-21FFh | I/Oポート (B-Bus)                     | 3.58MHz
2200h-3FFFh | 不使用                                | 3.58MHz
4000h-41FFh | I/Oポート (manual joypad access)      | 1.78MHz
4200h-5FFFh | I/Oポート                             | 3.58MHz
6000h-7FFFh | Expansion                            | 2.68MHz

## カートリッジ容量

24bitのアドレスバスで16MBのアドレスが可能ですが、かなりの部分がWRAMやI/Oのミラー領域で占められており、WS1/WS2のLoROM/HiROM領域のカートリッジROMは約11.9MBしか残っていません。（Expansion領域やI/O領域の隙間も利用すれば、もうちょっと利用可能です）

ほとんどのカートリッジでは、WS1とWS2は鏡のようになっていて、ほとんどのゲームはLoROMのみ、またはHiROMのみの領域を使用するため、以下のような容量になります。

```
  LoROMゲーム --> 最大 2MB ROM (banks 00h-3Fh, with mirror at 80h-BFh)
  HiROMゲーム --> 最大 4MB ROM (banks 40h-7Dh, with mirror at C0h-FFh)
```

カートリッジ容量の限界を克服する方法はいくつかあります。

LoROMゲームの中には、追加の "LoROM"バンクをHiROMエリア（BigLoROM）やWS2エリア（SpecialLoROM）にマッピングするものがあります。

HiROMゲームの中には、HiROMの追加バンクをWS2領域にマッピングするものがあります（ExHiROM）。また、SA-1、S-DD1、SPC7110、X-in-1などのマルチカートでは、バンク切り替えを採用しているカートリッジがあります。

## 32KB LoROM (システム領域を同一バンクに持つ32KBのROMバンク)

ROMは非連続の32KBブロックに分割されています。

CPUのDB、PBレジスタの設定を変更することなく、ROMやシステム領域（I/OポートやWRAM）にアクセスできるというメリットがあります。

## HiROM (64KBのROMバンク)

HiROMマッピングは、連続したROMアドレスを提供しますが、ROMバンク内のI/OやWRAM領域は含まれ**ません**。

HiROMバンク(64KB)の上半分は、通常、対応するLoROMバンク(32KB)とミラーリングされています。これは、`40FFE0h-40FFFFh`から`00FFE0h-00FFFFh`への割り込みベクタおよびリセットベクタのマッピングに重要です。

## バッテリ式のSRAM

バッテリ式のSRAMは、多くのゲームでセーブデータの保存に使用されています。SRAMのサイズは通常2KB、8KB、32KBです。一部のゲームでは32KB以上あります。

SRAMのアドレスマッピングには、LoROMゲーム用とHiROMゲーム用の2つの基本方式があります。

```
  HiROM ---> SRAM at 30h-3Fh,B0h-BFh:6000h-7FFFh    ; small 8K SRAM bank(s)
  LoROM ---> SRAM at 70h-7Dh,F0h-FFh:0000h-7FFFh    ; big 32K SRAM bank(s)
```

SRAMは通常、WS1とWS2の両エリアにもミラーリングされます。

バンク`30h-3Fh`のSRAMは、しばしば`20h-2Fh`（場合によっては`10h-1Fh`）にもミラーリングされます。バンク`70h-7Dh`のSRAMは、`70h-71h`または`70h-77h`にcrippleされたり、`60h-7Dh`に拡張されたり、また、オフセット`8000h-FFFFh`にミラーリングされることもあります。

## AバスとBバス

スーファミには、24bitのアドレスバス（Aバス）のほかに、特定のI/Oポートにアクセスするための8bitのアドレスバス（Bバス）が存在します。2つのアドレスバスは同じデータバスを共有していますが、それぞれのバスは独自のリード/ライト信号を持っています。

CPUはシステム領域内のオフセット`2100h..21FFh`でB-Busにアクセスできます。CPUのアクセスにとって、B-BusはA-Busのサブセットに過ぎません。

DMAコントローラは、BバスとAバスの両方に同時にアクセスすることができます。ソースアドレスとデスティネーションアドレスを同時に2つのバスに出力することができるので、これまでのように`Read-and-Write`ではなく、`Read-then-Write`を1つのステップで行うことができます。

## バンク切り替え

ほとんどのスーファミゲームは、24bitのアドレス空間で満足しています。バンク切り替えは、特殊なチップを搭載した一部のゲームでのみ使用されています。

```
  S-DD1, SA-1, and SPC7110 chips (with mappable 1MByte-banks)
  Satellaview FLASH carts (can enable/disable ROM, PSRAM, FLASH)
  Nintendo Power FLASH carts (can map FLASH and SRAM to desired address)
  Pirate X-in-1 multicart mappers (mappable offset in 256Kbyte units)
  Cheat devices (and X-Band modem) can map their BIOS and can patch ROM bytes
  Copiers can map internal BIOS/DRAM/SRAM and external Cartridge memory
  Hotel Boxes (eg. SFC-Box) can map multiple games/cartridges
```

また、APU側では、64BのブートROMを有効/無効にすることができます。
