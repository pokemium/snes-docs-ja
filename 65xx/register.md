# [レジスタ](https://problemkaputt.de/fullsnes.htm#cpuregistersandflags)

65XXのCPUは3本以下の汎用レジスタ(A, X, Y)を備えています。

しかし、レジスタの数が限られているというデメリットも、快適なメモリ操作でカバーできる部分もあります。

特にメモリのページ0（アドレス`0000h..00FFh`）は比較的高速で複雑な操作が可能であるため、実質的に、CPUはメモリ内に約256個の8bitレジスタ（または128個の16bitレジスタ）を持っていると言えるでしょう。

詳細を知りたい場合は、[アドレッシング](addressing.md)を参照してください。

## 👾 レジスタ

 bit幅 | 名前 | 内容
---- | ---- | ---- 
8/16  | A   | アキュムレータ
8/16  | X   | インデックスレジスタX
8/16  | Y   | インデックスレジスタY
16    | PC  | プログラムカウンタ
8/16  | S   | スタックポインタ
8     | P   | ステータスレジスタ
16    | D   | ゼロページオフセット(8bit`[nn]`を16bit`[00:nn+D]`に拡張する)
8     | DB  | データバンク(16bit`[nnnn]`を24bit`[DB:nnnn]`に拡張する)
8     | PB  | プログラムカウンタバンク(16bit`PC`を24bit`PB:PC`に拡張する)

## 📚 スタックポインタ

スタックポインタは、メモリのページ1の256バイトをアドレスとしています。つまり、`00h..FFh`の値は、`0100h..01FFh`のメモリをアドレスとしています。

他の多くのCPUと同様に、データを格納する際のスタックポインタはデクリメントされます。

しかし、65XXの世界では、スタック上の最初のFREEバイトを指しているので、スタックをトップに初期化する際には、`S=(2)00h`ではなく、`S=(1)FFh`を設定します。

## 🚩 ステータスレジスタ(PSR)

PSR = Processor Status Register

 bit | 名前 | 内容
---- | ---- | ---- 
  0  | C   | Carry         (0=No Carry, 1=Carry)
  1  | Z   | Zero          (0=Nonzero, 1=Zero)
  2  | I   | IRQ Disable   (0=IRQ Enable, 1=IRQ Disable)
  3  | D   | Decimal Mode  (0=Normal, 1=BCD Mode for ADC/SBC opcodes)
  4  | X/B | Break Flag    (0=IRQ/NMI, 1=BRK/PHP opcode)  (0=16bit, 1=8bit)
  5  | M/U | Unused        (Always 1)                     (0=16bit, 1=8bit)
  6  | V   | Overflow      (0=No Overflow, 1=Overflow)
  7  | N   | Negative/Sign (0=Positive, 1=Negative)
  --  | E   | 後述(0=16bit, 1=8bit)

### Emulation Flag (E)

6502エミュレーションモードを起動します。このフラグはXCE命令でのみアクセスできます。

このモードでは、A/X/Yは（X/Mを設定した場合と同じで）8bitとなります。Sも同様に8bit(`0100h..01FFh`)になります。

XフラグはBフラグ(Break, 後述)に、MフラグはUフラグ(不使用、常に1)に変更されます。

例外ベクタは6502アドレスに変更され、例外は（24ビットではなく）16ビットのリターンアドレスのみをプッシュします。条件付き分岐は、ページをまたぐときに1サイクル余分にかかります。

新しい65C02と65C816のオペコードを使い続けることができます。また、オリジナルの6502の未定義オペコードはサポートされていません。

### Accumulator/Memory Flag (M)

Aを8bitモードに切り替えます。Aの上位8bitにはXBAでアクセス可能です。

さらに、A,X,Yのオペランドを使用しないすべてのオペコード（STZ, inc/dec、Rotate/Shiftなどの read-modify-write オペコード）を8bitモードに切り替えます。

### Index Register Flag (X)

X,Yを8bitモードに切り替えます。上位8bitは強制的に00hになります

### Carry Flag (C)

減算（SBC、CMP）に使用される場合、キャリーフラグは0以上の場合にセットされます。これは通常の80x86やZ80のCPUのキャリーフラグとは逆の意味を持っていることに注意してください。

他のすべての命令（ADC、ASL、LSR、ROL、ROR）では通常通り動作しますが、ROL/RORはキャリーを介して回転します。これは80x86ではROL/RORよりも、RCL/RCRに近い挙動です。

### Zero Flag (Z), Negative/Sign Flag (N), Overflow Flag (V)

どこでも同じように動作しますが、Zは結果がゼロのときにセットされ、Nは符号付き(bit7が1)のときにセットされます。Vは加算減算が符号付き数値の最大範囲(`-128..+127`)を超えた場合に設定されます。

### IRQ Disable Flag (I)

Iフラグをセットすると、IRQが無効化されます。NMIとBRK命令を無効にすることはできません。

### Decimal Mode Flag (D)

Packed BCD mode (range 00h..99h) for ADC and SBC opcodes.

### Break Flag (B)

The Break flag is intended to separate between IRQ and BRK which are both using the same vector, \[FFFEh\]. The flag cannot be accessed directly, but there are 4 situations which are writing the P register to stack, which are then allowing the examine the B-bit in the pushed value: The BRK and PHP opcodes always write "1" into the bit, IRQ/NMI execution always write "0".

## 👽 レジスタ名のエイリアス一覧

```
  DPR   D  (used in homebrew specs)
  K     PB (used by PHK)
  PBR   PB (used in specs)
  DBR   DB (used in specs)
  B     DB (used by PLB,PHB)
  B     Upper 8bit of A (used by XBA)
  C     Full 16bit of A (used by TDC,TCD,TSC,TCS)
```

