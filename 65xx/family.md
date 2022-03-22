# 65xxファミリ

6502が属する65xxファミリのCPUについてです。

これらのプロセッサは、プログラマの観点から見た場合はすべて同じです。

```
 6501   6502のプロトタイプ
 6502   CBMフロッピーや一部の8bitコンピュータで使用された
 6507   Used in Atari 2600, 28pins (only 13 address lines, no /IRQ, no /NMI).
 6510   C64で使われていた6bitのI/Oポート付きプロセッサ
 7501   Used in C16,C116,Plus/4, with built-in 7bit I/O Port, without /NMI pin.
 8500   Used in C64-II, with different pin-outs.
 8501   7501と同じ
 8502   Used in C128s.
```

100%の互換性がないプロセッサもあります。

```
 65C02  6502を拡張したもの
 65SC02 65C02からオペコードを少し削ったもの
 65CE02 6502を拡張したものでC65で使用された
 65C816 Extended 65C02 with new opcodes and 16 bit operation modes.
 2A03   Nintendo NES/Famicom, modified 6502 with built-in sound controller.
```
