# VRAMへのアクセス

このページでは、VRAMにアクセスするためのレジスタについて解説します。

VRAM自体の内容については[こちら](../video/vram.md)を参照。

## 2115h - VMAIN - VRAM Address Increment Mode (W)

```
  7     Increment VRAM Address after accessing High/Low byte (0=Low, 1=High)
  6-4   不使用
  3-2   Address Translation    (0..3 = 0bit/None, 8bit, 9bit, 10bit)
  1-0   Address Increment Step (0..3 = Increment Word-Address by 1,32,128,128)
```

The address translation is intended for bitmap graphics (where one would have filled the BG Map by increasing Tile numbers), technically it does thrice left-rotate the lower 8, 9, or 10 bits of the Word-address:

```
  Translation  Bitmap Type              Port [2116h/17h]    VRAM Word-Address
  8bit rotate  4-color; 1 word/plane    aaaaaaaaYYYxxxxx --> aaaaaaaaxxxxxYYY
  9bit rotate  16-color; 2 words/plane  aaaaaaaYYYxxxxxP --> aaaaaaaxxxxxPYYY
  10bit rotate 256-color; 4 words/plane aaaaaaYYYxxxxxPP --> aaaaaaxxxxxPPYYY
```

Where "aaaaa" would be the normal address MSBs, "YYY" is the Y-index (within a 8x8 tile), "xxxxx" selects one of the 32 tiles per line, "PP" is the bit-plane index (for BGs with more than one Word per plane). For the intended result (writing rows of 256 pixels) the Translation should be combined with Increment Step=1.

For Mode 7 bitmaps one could eventually combine step 32/128 with 8bit/10bit rotate:

```
  8bit-rotate/step32   aaaaaaaaXXXxxYYY --> aaaaaaaaxxYYYXXX
  10bit-rotate/step128 aaaaaaXXXxxxxYYY --> aaaaaaxxxxYYYXXX
```

Though the SNES can't access enought VRAM for fullscreen Mode 7 bitmaps.

Step 32 (without translation) is useful for updating BG Map columns (eg. after horizontal scrolling).

## 2116h - VMADDL - VRAM Address (lower 8bit) (W)
## 2117h - VMADDH - VRAM Address (upper 8bit) (W)

VRAM Address for reading/writing. This is a WORD address (2-byte steps), the PPU could theoretically address up to 64K-words (128K-bytes), in practice, only 32K-words (64K-bytes) are installed in SNES consoles (VRAM address bit15 is not connected, so addresses 8000h-FFFFh are mirrors of 0-7FFFh).

After reading/writing VRAM Data, the Word-address can be automatically incremented by 1,32,128 (depending on the Increment Mode in Port 2115h) (Note: the Address Translation feature is applied only "temporarily" upon memory accesses, it doesn't affect the value in Port 2116h-17h).

Writing to 2116h/2117h does prefetch 16bit data from the new address (for later reading).

## 2118h - VMDATAL - VRAM Data Write (lower 8bit) (W)
## 2119h - VMDATAH - VRAM Data Write (upper 8bit) (W)

Writing to 2118h or 2119h does simply modify the LSB or MSB of the currently addressed VRAM word (with optional Address Translation applied). Depending on the Increment Mode the address does (or doesn't) get automatically incremented after the write.

## 2139h - RDVRAML - VRAM Data Read (lower 8bit) (R)
## 213Ah - RDVRAMH - VRAM Data Read (upper 8bit) (R)

Reading from these registers returns the LSB or MSB of an internal 16bit prefetch register. Depending on the Increment Mode the address does (or doesn't) get automatically incremented after the read.
The prefetch register is filled with data from the currently addressed VRAM word (with optional Address Translation applied) upon two situations:

```
  Prefetch occurs AFTER changing the VRAM address (by writing 2116h/17h).
  Prefetch occurs BEFORE incrementing the VRAM address (by reading 2139h/3Ah).
```

The "Prefetch BEFORE Increment" effect is some kind of a hardware glitch (Prefetch AFTER Increment would be more useful). Increment/Prefetch in detail:

```
  1st  Send a byte from OLD prefetch value to the CPU        ;-this always
  2nd  Load NEW value from OLD address into prefetch register;\these only if
  3rd  Increment address so it becomes the NEW address       ;/increment occurs
```

Increments caused by writes to 2118h/19h don't do any prefetching (the prefetch register is left totally unchanged by writes).

In practice: After changing the VRAM address (via 2116h/17h), the first byte/word will be received twice, further values are received from properly increasing addresses (as a workaround: issue a dummy-read that ignores the 1st or 2nd value).