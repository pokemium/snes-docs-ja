# 回転・シフト命令

## 算術・論理左シフト

オペコード | フラグ | サイクル | Native | Nocash | 内容
-- | -- | -- | -- | -- | -- 
0A | nzc--- | 2 | ASL A | SHL A | SHL A
06 nn | nzc--- | 5 | ASL nn | SHL \[nn\] | SHL \[D+nn\]
16 nn | nzc--- | 6 | ASL nn,X | SHL \[nn+X\] | SHL \[D+nn+X\]
0E nn nn | nzc--- | 6 | ASL nnnn | SHL \[nnnn\] | SHL \[DB:nnnn\]
1E nn nn | nzc--- | 7 | ASL nnnn,X | SHL \[nnnn+X\] | SHL \[DB:nnnn+X\]

## 論理右シフト

オペコード | フラグ | サイクル | Native | Nocash | 内容
-- | -- | -- | -- | -- | -- 
4A       | 0zc--- | 2 | LSR A      | SHR A        | SHR A
46 nn    | 0zc--- | 5 | LSR nn     | SHR \[nn\]     | SHR \[D+nn\]
56 nn    | 0zc--- | 6 | LSR nn,X   | SHR \[nn+X\]   | SHR \[D+nn+X\]
4E nn nn | 0zc--- | 6 | LSR nnnn   | SHR \[nnnn\]   | SHR \[DB:nnnn\]
5E nn nn | 0zc--- | 7 | LSR nnnn,X | SHR \[nnnn+X\] | SHR \[DB:nnnn+X\]


## CY付き左ローテート

オペコード | フラグ | サイクル | Native | Nocash | 内容
-- | -- | -- | -- | -- | -- 
2A       | nzc--- | 2 | ROL A      | RCL A        | RCL A
26 nn    | nzc--- | 5 | ROL nn     | RCL \[nn\]     | RCL \[D+nn\]
36 nn    | nzc--- | 6 | ROL nn,X   | RCL \[nn+X\]   | RCL \[D+nn+X\]
2E nn nn | nzc--- | 6 | ROL nnnn   | RCL \[nnnn\]   | RCL \[DB:nnnn\]
3E nn nn | nzc--- | 7 | ROL nnnn,X | RCL \[nnnn+X\] | RCL \[DB:nnnn+X\]


## CY付き右ローテート

オペコード | フラグ | サイクル | Native | Nocash | 内容
-- | -- | -- | -- | -- | -- 
6A       | nzc--- | 2 | ROR A      | RCR A        | RCR A
66 nn    | nzc--- | 5 | ROR nn     | RCR \[nn\]     | RCR \[D+nn\]
76 nn    | nzc--- | 6 | ROR nn,X   | RCR \[nn+X\]   | RCR \[D+nn+X\]
6E nn nn | nzc--- | 6 | ROR nnnn   | RCR \[nnnn\]   | RCR \[DB:nnnn\]
7E nn nn | nzc--- | 7 | ROR nnnn,X | RCR \[nnnn+X\] | RCR \[DB:nnnn+X\]

Notes:

ROR命令は、1976年6月以降のMCS650Xマイクロプロセッサで利用可能です。

ROLとRORは8ビットの値をキャリー付きで回転させます（つまり合計9ビットを回転させます）。

