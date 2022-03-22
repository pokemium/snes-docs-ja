# snes-docs-ja

<img src="images/Nintendo-Super-Famicom-Set-FL.png" height="280" />

SNES, スーパーファミコン(SFC)について、技術的な詳細を日本語でまとめたものです。自分のメモの側面もあります。

突然消えたり非公開にする可能性もあるので心配な方はクローンしておくことをお勧めします。

## コンテンツ一覧

### SNES

- [仕様](spec.md)
- [米国版スーファミとの違い](sfc_snes.md)
- [メモリマップ](memory/map.md)

### CPU

- [レジスタ](65xx/register.md)
- [アドレッシング](65xx/addressing.md)
- [データ転送命令](65xx/transfer.md)
- [ALU](65xx/alu.md)
- [回転・シフト命令](65xx/rotate_shift.md)
- [ジャンプ命令](65xx/jump.md)
- [制御命令](65xx/control.md)
- [65xxファミリ](65xx/family.md)

### メモリ

- [メモリマップ](memory/map.md)
- [制御レジスタ](memory/control.md)
- [WRAMへのレジスタ経由のアクセス](memory/wram.md)

### グラフィック

- [概要](video/README.md)

### カートリッジ

- [カートリッジヘッダ](cartridge/header.md)

### その他

#### コードリーディング

- [bsnes-emu/bsnes](others/bsnes/README.md)
- [snes9xgit/snes9x](others/snes9x/README.md)

## 関連するレポジトリ

- [gb-docs-ja](https://github.com/pokemium/gb-docs-ja): GameBoyについて
- [gba-docs-ja](https://github.com/pokemium/gba-docs-ja): GameBoy Advanceについて
- [nds-docs-ja](https://github.com/pokemium/nds-docs-ja): Nintendo DSについて

## 参考記事

- [SNES Development Wiki](https://wiki.superfamicom.org/)
- [Fullsnes - nocash SNES hardware specifications](https://problemkaputt.de/fullsnes.htm)
- [SNES研究室](http://hp.vector.co.jp/authors/VA042397/snes/index.html)
- [Super Nintendo Architecture](https://www.copetti.org/writings/consoles/super-nintendo/)
- [Super Nintendo Graphics Guide – Mega Cat Studios](https://megacatstudios.com/blogs/retro-development/super-nintendo-graphic-guide)
- [Wikipedia - スーパーファミコン](https://ja.wikipedia.org/wiki/%E3%82%B9%E3%83%BC%E3%83%91%E3%83%BC%E3%83%95%E3%82%A1%E3%83%9F%E3%82%B3%E3%83%B3)

