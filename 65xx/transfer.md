# データ転送命令

## レジスタからレジスタへのデータ転送

オペコード | フラグ | サイクル | Native | Nocash | Bits | Effect
 -- | -- | -- | -- | --  | --  | --
 A8 | nz---- | 2 | TAY | MOV Y,A  | x  | Y=A
 AA | nz---- | 2 | TAX | MOV X,A  | x  | X=A
 BA | nz---- | 2 | TSX | MOV X,S  | x  | X=S
 98 | nz---- | 2 | TYA | MOV A,Y  | m  | A=Y
 8A | nz---- | 2 | TXA | MOV A,X  | m  | A=X
 9A | ------ | 2 | TXS | MOV S,X  | e  | S=X
 9B | nz---- | 2 | TXY | MOV Y,X  | x  | Y=X
 BB | nz---- | 2 | TYX | MOV X,Y  | x  | X=Y
 7B | nz---- | 2 | TDC | MOV A,D  | 16 | A=D
 5B | nz---- | 2 | TCD | MOV D,A  | 16 | D=A
 3B | nz---- | 2 | TSC | MOV A,SP | 16 | A=SP
 1B | ------ | 2 | TCS | MOV SP,A | e? | SP=A

## メモリからレジスタへの読み込み

オペコード | フラグ | サイクル | Native | Nocash | Bits | Effect
 -- | -- | -- | -- | --  | --  | --
  A9 nn       | nz---- | 2  | LDA #nn      | MOV A,nn          | -- | A=nn
  A5 nn       | nz---- | 3  | LDA nn       | MOV A,\[nn\]        | -- | A=\[D+nn\]
  B5 nn       | nz---- | 4  | LDA nn,X     | MOV A,\[nn+X\]      | -- | A=\[D+nn+X\]
  A3 nn       | nz---- |    | LDA nn,S     | MOV A,\[nn+S\]      | -- | A=\[nn+S\]
  AD nn nn    | nz---- | 4  | LDA nnnn     | MOV A,\[nnnn\]      | -- | A=\[DB:nnnn\]
  BD nn nn    | nz---- | 4* | LDA nnnn,X   | MOV A,\[nnnn+X\]    | -- | A=\[DB:nnnn+X\]
  B9 nn nn    | nz---- | 4* | LDA nnnn,Y   | MOV A,\[nnnn+Y\]    | -- | A=\[DB:nnnn+Y\]
  AF nn nn nn | nz---- |    | LDA nnnnnn   | MOV A,\[nnnnnn\]    | -- | A=\[nnnnnn\]
  BF nn nn nn | nz---- |    | LDA nnnnnn,X | MOV A,\[nnnnnn+X\]  | -- | A=\[nnnnnn+X\]
  B2 nn       | nz---- |    | LDA (nn)     | MOV A,\[\[nn\]\]      | -- | A=\[WORD\[D+nn\]\]
  A1 nn       | nz---- | 6  | LDA (nn,X)   | MOV A,\[\[nn+X\]\]    | -- | A=\[WORD\[D+nn+X\]\]
  B1 nn       | nz---- | 5* | LDA (nn),Y   | MOV A,\[\[nn\]+Y\]    | -- | A=\[WORD\[D+nn\]+Y\]
  B3 nn       | nz---- |    | LDA (nn,S),Y | MOV A,\[\[nn+S\]+Y\]  | -- | A=\[WORD\[nn+S\]+Y\]
  A7 nn       | nz---- |    | LDA \[nn\]     | MOV A,\[FAR\[nn\]\]   | -- | A=\[FAR\[D+nn\]\]
  B7 nn       | nz---- |    | LDA \[nn\],y   | MOV A,\[FAR\[nn\]+Y\] | -- | A=\[FAR\[D+nn\]+Y\]
  A2 nn       | nz---- | 2  | LDX #nn      | MOV X,nn          | -- | X=nn
  A6 nn       | nz---- | 3  | LDX nn       | MOV X,\[nn\]        | -- | X=\[D+nn\]
  B6 nn       | nz---- | 4  | LDX nn,Y     | MOV X,\[nn+Y\]      | -- | X=\[D+nn+Y\]
  AE nn nn    | nz---- | 4  | LDX nnnn     | MOV X,\[nnnn\]      | -- | X=\[DB:nnnn\]
  BE nn nn    | nz---- | 4* | LDX nnnn,Y   | MOV X,\[nnnn+Y\]    | -- | X=\[DB:nnnn+Y\]
  A0 nn       | nz---- | 2  | LDY #nn      | MOV Y,nn          | -- | Y=nn
  A4 nn       | nz---- | 3  | LDY nn       | MOV Y,\[nn\]        | -- | Y=\[D+nn\]
  B4 nn       | nz---- | 4  | LDY nn,X     | MOV Y,\[nn+X\]      | -- | Y=\[D+nn+X\]
  AC nn nn    | nz---- | 4  | LDY nnnn     | MOV Y,\[nnnn\]      | -- | Y=\[DB:nnnn\]
  BC nn nn    | nz---- | 4* | LDY nnnn,X   | MOV Y,\[nnnn+X\]    | -- | Y=\[DB:nnnn+X\]

> \* Add one cycle if indexing crosses a page boundary.

## レジスタからメモリへの書き込み

オペコード | フラグ | サイクル | Native | Nocash | Bits | Effect
 -- | -- | -- | -- | --  | --  | --
  64 nn       | ------ | 3 | STZ nn       | MOV \[nn\],0        | m | \[D+nn\]=0
  74 nn       | ------ | 4 | STZ nn_x     | MOV \[nn+X\],0      | m | \[D+nn+X\]=0
  9C nn nn    | ------ | 4 | STZ nnnn     | MOV \[nnnn\],0      | m | \[DB:nnnn\]=0
  9E nn nn    | ------ | 5 | STZ nnnn_x   | MOV \[nnnn+X\],0    | m | \[DB:nnnn+X\]=0
  85 nn       | ------ | 3 | STA nn       | MOV \[nn\],A        | m | \[D+nn\]=A
  95 nn       | ------ | 4 | STA nn,X     | MOV \[nn+X\],A      | m | \[D+nn+X\]=A
  83 nn       | ------ |   | STA nn,S     | MOV \[nn+S\],A      | m | \[nn+S\]=A
  8D nn nn    | ------ | 4 | STA nnnn     | MOV \[nnnn\],A      | m | \[DB:nnnn\]=A
  9D nn nn    | ------ | 5 | STA nnnn,X   | MOV \[nnnn+X\],A    | m | \[DB:nnnn+X\]=A
  99 nn nn    | ------ | 5 | STA nnnn,Y   | MOV \[nnnn+Y\],A    | m | \[DB:nnnn+Y\]=A
  8F nn nn nn | ------ |   | STA nnnnnn   | MOV \[nnnnnn\],A    | m | \[nnnnnn\]=A
  9F nn nn nn | ------ |   | STA nnnnnn,X | MOV \[nnnnnn+X\],A  | m | \[nnnnnn+X\]=A
  81 nn       | ------ | 6 | STA (nn,X)   | MOV \[\[nn+X\]\],A    | m | \[WORD\[D+nn+X\]\]=A
  91 nn       | ------ | 6 | STA (nn),Y   | MOV \[\[nn\]+Y\],A    | m | \[WORD\[D+nn\]+Y\]=A
  92 nn       | ------ |   | STA (nn)     | MOV \[\[nn\]\],A      | m | \[WORD\[D+nn\]\]=A
  93 nn       | ------ |   | STA (nn,S),Y | MOV \[\[nn+S\]+Y\],A  | m | \[WORD\[nn+S\]+Y\]=A
  87 nn       | ------ |   | STA \[nn\]     | MOV \[FAR\[nn\]\],A   | m | \[FAR\[D+nn\]\]=A
  97 nn       | ------ |   | STA \[nn\],y   | MOV \[FAR\[nn\]+Y\],A | m | \[FAR\[D+nn\]+Y\]=A
  86 nn       | ------ | 3 | STX nn       | MOV \[nn\],X        | x | \[D+nn\]=X
  96 nn       | ------ | 4 | STX nn,Y     | MOV \[nn+Y\],X      | x | \[D+nn+Y\]=X
  8E nn nn    | ------ | 4 | STX nnnn     | MOV \[nnnn\],X      | x | \[DB:nnnn\]=X
  84 nn       | ------ | 3 | STY nn       | MOV \[nn\],Y        | x | \[D+nn\]=Y
  94 nn       | ------ | 4 | STY nn,X     | MOV \[nn+X\],Y      | x | \[D+nn+X\]=Y
  8C nn nn    | ------ | 4 | STY nnnn     | MOV \[nnnn\],Y      | x | \[DB:nnnn\]=Y

## スタックのPush,Pull

オペコード | フラグ | サイクル | Native | Nocash | Bits | Effect
 -- | -- | -- | -- | --  | --  | --
  48       | ------ | 3 | PHA       | PUSH A        |  m | \[S\]=A
  DA       | ------ | 3 | PHX       | PUSH X        |  x | \[S\]=X
  5A       | ------ | 3 | PHY       | PUSH Y        |  x | \[S\]=Y
  08       | ------ | 3 | PHP       | PUSH P        |  8 | \[S\]=P
  8B       | ------ | 3 | PHB       | PUSH DB       |  8 | \[S\]=DB
  4B       | ------ | 3 | PHK       | PUSH PB       |  8 | \[S\]=PB
  0B       | ------ | 4 | PHD       | PUSH D        | 16 | \[S\]=D
  D4 nn    | ------ | 6 | PEI nn    | PUSH WORD\[nn\] | 16 | \[S\]=WORD\[D+nn\]
  F4 nn nn | ------ | 5 | PEA nnnn  | PUSH nnnn     | 16 | \[S\]=NNNN
  62 nn nn | ------ | 6 | PER rel16 | PUSH disp16   | 16 | \[S\]=$+3+disp
  68       | nz---- | 4 | PLA       | POP  A        |  m | A=\[S\]
  FA       | nz---- | 4 | PLX       | POP  X        |  x | X=\[S\]
  7A       | nz---- | 4 | PLY       | POP  Y        |  x | Y=\[S\]
  2B       | nz---- | 5 | PLD       | POP  D        | 16 | D=\[S\]
  AB       | nz---- | 4 | PLB       | POP  DB       |  8 | DB=\[S\]
  28       | nzcidv | 4 | PLP       | POP  P        |  8 | P=\[S\]

## ブロック単位のメモリ転送

オペコード | フラグ | サイクル | Native | Nocash | 備考
 -- | -- | -- | -- | --  | -- 
 44 dd ss | ------ | 7x | MVP ss,dd | LDDR \[dd:Y\],\[ss:X\],A+1 | DEC X/Y
 54 dd ss | ------ | 7x | MVN ss,dd | LDIR \[dd:Y\],\[ss:X\],A+1 | INC X/Y
