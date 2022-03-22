# ジャンプ命令

## 無条件ジャンプ

オペコード | フラグ | サイクル | Native | Nocash | 内容
-- | -- | -- | -- | -- | -- 
80 dd       | ------ | 3xx | BRA disp8    | JMP disp      | PC=PC+/-disp8
82 dd dd    | ------ | 4   | BRL disp16   | JMP disp      | PC=PC+/-disp16
4C nn nn    | ------ | 3   | JMP nnnn     | JMP nnnn      | PC=nnnn
5C nn nn nn | ------ | 4   | JMP nnnnnn   | JMP nnnnnn    | PB:PC=nnnnnn
6C nn nn    | ------ | 5   | JMP (nnnn)   | JMP \[nnnn\]    | PC=WORD\[00:nnnn\]
7C nn nn    | ------ | 6   | JMP (nnnn,X) | JMP \[nnnn+X\]  | PC=WORD\[PB:nnnn+X\]
DC nn nn    | ------ | 6   | JML ...      | JMP FAR\[nnnn\] | PB:PC=\[00:nnnn\]
20 nn nn    | ------ | 6   | JSR nnnn     | CALL nnnn     | \[S\]=PC+2,PC=nnnn
22 nn nn nn | ------ | 4   | JSL nnnnnn   | CALL nnnnnn   | PB:PC=nnnnnn \[S\]=PB:PC+3
FC nn nn    | ------ | 6   | JSR (nnnn,X) | CALL \[nnnn+X\] | PC=WORD\[PB:nnnn+X\] \[S\]=PC
40          | nzcidv | 6   | RTI          | RETI          | P=\[S+1\],PB:PC=\[S+2\],S=S+4
6B          | ------ | ?   | RTL          | RETF          | PB:PC=\[S+1\]+1, S=S+3
60          | ------ | 6   | RTS          | RET           | PC=\[S+1\]+1, S=S+2

<pre>
Note: RTIは、Bフラグおよび不使用フラグを変更することはできません。
</pre>

Glitch: For `JMP [nnnn]` the operand word cannot cross page boundaries, ie. `JMP [03FFh]` would fetch the MSB from `[0300h]` instead of `[0400h]`. Very simple workaround would be to place a ALIGN 2 before the data word.

## 条件付きジャンプ

オペコード | フラグ | サイクル | Native | Nocash | 条件
-- | -- | -- | -- | -- | -- 
10 dd | ------ | 2<sup>[1](#cycle)</sup> | BPL     | JNS     disp  | `N=0` (plus/positive)
30 dd | ------ | 2<sup>[1](#cycle)</sup> | BMI     | JS      disp  | `N=1` (minus/negative/signed)
50 dd | ------ | 2<sup>[1](#cycle)</sup> | BVC     | JNO     disp  | `V=0` (no overflow)
70 dd | ------ | 2<sup>[1](#cycle)</sup> | BVS     | JO      disp  | `V=1` (overflow)
90 dd | ------ | 2<sup>[1](#cycle)</sup> | BCC/BLT | JNC/JB  disp  | `C=0` (less/below/no carry)
B0 dd | ------ | 2<sup>[1](#cycle)</sup> | BCS/BGE | JC/JAE  disp  | `C=1` (above/greater/equal/carry)
D0 dd | ------ | 2<sup>[1](#cycle)</sup> | BNE/BZC | JNZ/JNE disp  | `Z=0` (not zero/not equal)
F0 dd | ------ | 2<sup>[1](#cycle)</sup> | BEQ/BZS | JZ/JE   disp  | `Z=1` (zero/equal)

<sup id="cycle">1: 実行時間は、条件が偽の場合（分岐が実行されない）、2サイクルです。それ以外の場合は、ジャンプ先が同じメモリページ内にある場合は3サイクル、ページ境界を越える場合は4サイクルです。</sup>

<pre>
Note: x86やZ80のCPUとは異なり、減算（SBCやCMP）は以下(`A-B`として、`A >= B`)のときに`carry=set`となります。
</pre>

## 割り込み、例外、ブレーク

```
  Opcode                                                      6502     65C816
  00   BRK    Break      B=1 [S]=$+2,[S]=P,D=0 I=1, PB=00, PC=[00FFFE] [00FFE6]
  02   COP    ;65C816    B=1 [S]=$+2,[S]=P,D=0 I=1, PB=00, PC=[00FFF4] [00FFE4]
  --   /ABORT ;65C816                               PB=00, PC=[00FFF8] [00FFE8]
  --   /IRQ   Interrupt  B=0 [S]=PC, [S]=P,D=0 I=1, PB=00, PC=[00FFFE] [00FFEE]
  --   /NMI   NMI        B=0 [S]=PC, [S]=P,D=0 I=1, PB=00, PC=[00FFFA] [00FFEA]
  --   /RESET Reset      D=0 E=1 I=1 D=0000, DB=00  PB=00, PC=[00FFFC] N/A
```

IRQはIフラグで無効にできますが、`BRK`，`/NMI`，`/RESET`信号はIフラグで無効にすることはできません。

例外(Exception)時は、まずBフラグが変更され（6502モードのとき）、次に`P`をスタックに書き込んだあと、Iフラグをセットし、Dフラグがクリアされます（オリジナルの6502とは異なります）。

6502モードでは、BRKとIRQは同じベクタを共有しており、ソフトウェアはプッシュされたBフラグのみを調べることで、BRKとIRQを見分けることができます。

The RTI opcode can be used to return from BRK/IRQ/NMI, note that using the return address from BRK skips one dummy/parameter byte following after the BRK opcode.

ソフトウェアまたはハードウェアは、`/IRQ`または`/NMI`信号を処理した後、アクノリッジまたはリセットを行うよう注意する必要があります。

```
IRQs are executed whenever "/IRQ=LOW AND I=0".
NMIs are executed whenever "/NMI changes from HIGH to LOW".
```

`/IRQ`をLOWにした場合、`I=0`にするとすぐに同じ（古い）割り込みが再度実行されます。 `/NMI`をLOWにした場合、それ以上NMIを実行することはできません。
