# WRAM

SNESは128KBバイトのWork RAM(WRAM)を搭載しており、いくつかの方法でアクセスすることができます。

```
  128KBのWRAMはアドレス空間 7E0000h-7FFFFFh に配置されています。
  先頭8KBは xx0000h-xx1FFFh (xx=00h..3Fh, 80h..BFh) にもミラーされています。
  加えて、218xhレジスタを通じてWRAMにはアクセス可能です。(主にDMA関連で使用します。)
```

## 2180h - WMDATA - WRAM Data Read/Write (R/W)

```
  7-0   Work RAM Data
```

このレジスタでは、単純に\[2181h-2183h\]の示すアドレス上のバイトを読み書きし、アドレスを1つインクリメントします。

このレジスタを通したWRAMの読み込み処理は`7E0000h-7FFFFFh`経由のアクセスより高速であるにもかかわらず、プリフェッチの有無とは無関係です。また、2180hを読むと、2180hや`7E0000h-7FFFFFFh`への書き込みと混在していても、常に現在アドレスされているバイトが返されます。

## 2181h - WMADDL - WRAM Address (0-7bit) (W)
## 2182h - WMADDM - WRAM Address (7-15bit) (W)
## 2183h - WMADDH - WRAM Address (16bit) (W)

このレジスタでは、WMDATA(`2180h`)を通して128KBのWRAMにアクセスするための17bitアドレスを保持します。

```c
addr = ([2183h] << 16) | ([2182h] << 8) | [2181h]
```

## DMAについて

WRAM間のDMAはできません。A-BusからB-Bus方向にも、その逆にもできません。外部的には別々のアドレス線が存在しますが、WRAMのチップは両方を一度に処理することができないからです。

## アクセス速度

WRAMは2.6MHzでアクセスされます。つまり、WRAM上のすべての変数、スタック、プログラムコードへのアクセスは低速です。

なお、スーファミには高速アクセス可能なRAMは搭載されていません。しかし、次の工夫をすることで、3.5MHzでWRAMにアクセスすることができます。

- \[2180h\]を通じたWRAMへのシーケンシャルな読み込みは3.5MHzでアクセスされ、読み出しアドレスも自動でインクリメントされます。
- DMA registers at 43x0h-43xBh provide 8x12 bytes of read/write-able "memory".
- External RAM could be mapped to 5000h-5FFFh (but usually it's at slow 6000h).
- External RAM could be mapped to C00000h-FFFFFFh (probably rarely done too).

## Other Notes

The B-Bus feature with auto-increment is making it fairly easy to boot the SNES without any ROM/EPROM by simply writing program bytes to WRAM (and mirroring it to the Program and Reset vector to ROM area):

[SNES Xboo Upload (WRAM Boot)](SNES Xboo Upload (WRAM Boot))

Interestingly, the WRAM-to-ROM Area mirroring seems to be stable even when ROM Area is set to 3.5MHz Access Time - so it's unclear why Nintendo has restricted normal WRAM Access to 2.6MHz - maybe some WRAM chips are slower than others, or maybe they become unstable at certain room temperatures.
