# [アドレッシング](https://problemkaputt.de/fullsnes.htm#cpumemoryaddressing)

## アドレッシングモード

名前 | Native | Nocash
-- | -- | --
Implied      | -      | A,X,Y,S,P
Immediate    | #nn    | nn
Zero Page    | nn     | \[nn\]
Zero Page,X  | nn,X   | \[nn+X\]
Zero Page,Y  | nn,Y   | \[nn+Y\]
Absolute     | nnnn   | \[nnnn\]
Absolute,X   | nnnn,X | \[nnnn+X\]
Absolute,Y   | nnnn,Y | \[nnnn+Y\]
(Indirect,X) | (nn,X) | \[\[nn+X\]\]
(Indirect),Y | (nn),Y | \[\[nn\]+Y\]

### ゼロページ

`[nn]`,`[nn+X]`,`[nn+Y]`

Uses an 8bit parameter (one byte) to address the first 256 of memory at `0000h..00FFh`. This limited range is used even for "nn+X" and "nn+Y", ie. "C0h+60h" will access 0020h (not 0120h).

### 絶対アドレス

`[nnnn]`,`[nnnn+X]`,`[nnnn+Y]`

Uses a 16bit parameter (two bytes) to address the whole 64K of memory at `0000h..FFFFh`. Because of the additional parameter bytes, this is a bit slower than Zero Page accesses.

### Indirect

`[[nn+X]]`,`[[nn+Y]]`

Uses an 8bit parameter that points to a 16bit parameter in page zero.
Even though the CPU doesn't support 16bit registers (except for the program counter), this (double-)indirect addressing mode allows to use variable 16bit pointers.

### On-Chip Bi-directional I/O port

Addresses (00)00h and (00)01h are occupied by an I/O port which is built-in into 6510, 8500, 7501, 8501 CPUs (eg. used in C64 and C16), be sure not to use the addresses as normal memory. For description read chapter about I/O ports.

### Caution

Because of the identical format, assemblers will be more or less unable to separate between `[XXh+r]` and `[00XXh+r]`, the assembler will most likely produce `[XXh+r]` when address is already known to be located in page 0, and `[00XXh+r]` in case of forward references.
Beside for different opcode size/time, `[XXh+r]` will always access page 0 memory (even when XXh+r>FFh), while `[00XXh+r]` may direct to memory in page 0 or 1, to avoid unpredictable results be sure not to use (00)XXh+r>FFh if possible.
