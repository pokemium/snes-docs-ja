# VRAMへのアクセス

このページでは、VRAMにアクセスするためのレジスタについて解説します。

VRAM自体の内容については[こちら](../video/vram.md)を参照。

## 2115h - VMAIN - VRAM Address Increment Mode (W)

```
  7     上位/下位バイトのどちらにアクセスした後、VRAMアドレスをインクリメントするか (0=下位, 1=上位)
  6-4   不使用
  3-2   アドレス変換 (後述, 0..3 = 0bit/None, 8bit, 9bit, 10bit)
  1-0   VMADDのインクリメント量 (0..3 = Increment Word-Address by 1,32,128,128)
```

アドレス変換はビットマップ式を対象としており、技術的にはワードアドレスの下位8,9,10bitを3bitだけ左回転させます。

変換方法 | Bitmap Type | `Port [2116h/17h] --> VRAM Word-Address`
-- | -- | -- 
8bit rotate  | 4-color; 1 word/plane    | `aaaaaaaaYYYxxxxx --> aaaaaaaaxxxxxYYY`
9bit rotate  | 16-color; 2 words/plane  | `aaaaaaaYYYxxxxxP --> aaaaaaaxxxxxPYYY`
10bit rotate | 256-color; 4 words/plane | `aaaaaaYYYxxxxxPP --> aaaaaaxxxxxPPYYY`

`aaaaa`は通常のアドレスのMSB、`YYY`は8x8タイル内のYインデックス、`xxxxx`は1ラインあたり32タイルのうちの1つを選択、`PP`はビットプレーンのインデックス（1プレーンに複数のWordがあるBGの場合）です。

256ピクセルの行を書きたいなら、アドレス変換する際には`Increment Step=1`と組み合わせる必要があります。

Mode7のビットマップの場合、最終的には32/128ステップ(bit1-0)と8bit/10bit回転(bit3-2)の組み合わせになります。

```
  8bit-rotate/step32   aaaaaaaaXXXxxYYY --> aaaaaaaaxxYYYXXX
  10bit-rotate/step128 aaaaaaXXXxxxxYYY --> aaaaaaxxxxYYYXXX
```

しかし、SNESはMode7ビットマップで画面全体を描画するのに十分大きなVRAMを持っていません。

アドレス変換なしの32ステップはBGマップの列を更新するのに便利です。(例: X方向のスクロール後)

## 2116h - VMADDL - VRAMアドレスレジスタ (下位8bit) (W)
## 2117h - VMADDH - VRAMアドレスレジスタ (上位8bit) (W)

VRAMの読み出し/書き込み用アドレスを指定するレジスタです。これはワード単位（2バイト単位）で指定します。つまり、実際のアドレスは`2 * VMADD`です。

理論上最大128KB(64Kワード)までアドレス指定できますが、実際にはスーファミ本体には64KB(32Kワード)しか搭載されていません（VMADD.15は不使用で0なので、`8000h..FFFFh`は`0..7FFFh`のミラーリングとなります）。

VRAMデータの読み出し/書き込み後、`VMAIN(2115h)`のインクリメントモードにより、ワードアドレス(`VMADD`)が 1,32,128ずつ自動的にインクリメントされることがあります。

注: アドレス変換機能は、メモリアクセス時に「一時的に」適用されるだけで、ポート`2116h..2117h`の値には影響を与えません。

2116h/2117hへの書き込みは、後で読み出すために新しいアドレスから16bitデータをプリフェッチします。

## 2118h - VMDATAL - VRAMデータ書き込みレジスタ (下位8bit) (W)
## 2119h - VMDATAH - VRAMデータ書き込みレジスタ (上位8bit) (W)

`2118h`または`2119h`への書き込みは、現在(アドレス変換を適用した)アドレスで指定されているVRAMのLSBまたはMSBを変更するだけです。VMAINのインクリメントモードにより、書き込み後、アドレスは自動的にインクリメントされます。

## 2139h - RDVRAML - VRAMデータ読み込みレジスタ (下位8bit) (R)
## 213Ah - RDVRAMH - VRAMデータ読み込みレジスタ (上位8bit) (R)

これらのレジスタから読み取りを行うと、内部の16bitプリフェッチレジスタのLSBまたはMSBが返されます。VMAINのインクリメントモードによって、読み出し後にアドレスが自動的にインクリメントされます。

プリフェッチレジスタは、次の2つの場合に、VMADDL/VMADDHでアドレス指定されているVRAMワード（オプションのアドレス変換が適用されている）からデータで満たされます。

```
プリフェッチが行われる2つのパターン:
  2116h/17hへの書き込みによってVRAMアドレスが変化したとき
  2139h/3Ahからの読み取りでVRAMアドレスがインクリメントされたとき
```

The "Prefetch BEFORE Increment" effect is some kind of a hardware glitch (Prefetch AFTER Increment would be more useful). Increment/Prefetch in detail:

```
  1st  Send a byte from OLD prefetch value to the CPU        ;-this always
  2nd  Load NEW value from OLD address into prefetch register;\these only if
  3rd  Increment address so it becomes the NEW address       ;/increment occurs
```

Increments caused by writes to 2118h/19h don't do any prefetching (the prefetch register is left totally unchanged by writes).

In practice: After changing the VRAM address (via 2116h/17h), the first byte/word will be received twice, further values are received from properly increasing addresses (as a workaround: issue a dummy-read that ignores the 1st or 2nd value).