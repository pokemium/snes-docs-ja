# 制御命令

## 制御命令

オペコード | フラグ | サイクル | Native | Nocash | 内容
-- | -- | -- | -- | -- | -- 
18    | --0--- | 2 | CLC     | CLC      | C=0    ;Clear carry flag
58    | ---0-- | 2 | CLI     | EI       | I=0    ;Clear interrupt disable bit
D8    | ----0- | 2 | CLD     | CLD      | D=0    ;Clear decimal mode
B8    | -----0 | 2 | CLV     | CL?      | V=0    ;Clear overflow flag
38    | --1--- | 2 | SEC     | STC      | C=1    ;Set carry flag
78    | ---1-- | 2 | SEI     | DI       | I=1    ;Set interrupt disable bit
F8    | ----1- | 2 | SED     | STD      | D=1    ;Set decimal mode
C2 nn | xxxxxx | 3 | REP #nn | CLR P,nn | P=P AND NOT nn
E2 nn | xxxxxx | 3 | SEP #nn | SET P,nn | P=P OR nn
FB    | --c--- | 2 | XCE     | XCE      | C=E, E=C

## 特殊命令

オペコード | フラグ | サイクル | Native | Nocash | 内容
-- | -- | -- | -- | -- | --
DB    | ------ | -  | STP     | KILL   | STOP/KILL
EB    | nz---- | 3  | XBA     | SWAP A | `A=B, B=A, NZ=LSB`
CB    | ------ | 3x | WAI     | HALT   | HALT命令
42 nn |        | 2  | WDM #nn | NUL nn | 何もしない
EA    | ------ | 2  | NOP     | NOP    | 何もしない

WAI/HALT stops the CPU until an exception (usually an IRQ or NMI) request occurs; in case of IRQs this works even if IRQs are disabled (via I=1).

## Conditional Branch Page Crossing

The branch opcode with parameter takes up two bytes, causing the PC to get incremented twice (PC=PC+2), without any extra boundary cycle. The signed parameter is then added to the PC (PC+disp), the extra clock cycle occurs if the addition crosses a page boundary (next or previous 100h-page).
