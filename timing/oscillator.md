# 周波数

## NTSC版

```
  NTSC crystal      21.4772700MHz (X1, type number D214K1)
  NTSC color clock  3.57954500MHz (21.47727MHz/6)  (generated by PPU2 chip)
  NTSC master clock 21.4772700MHz (21.47727MHz/1)  (without multiplier/divider)
  NTSC dot clock    5.36931750MHz (21.47727MHz/4)  (generated by PPU chip)
  NTSC cpu clock    3.57954500MHz (21.47727MHz/6)  (without waitstates)
  NTSC cpu clock    2.68465875MHz (21.47727MHz/8)  (short waitstates)
  NTSC cpu clock    1.78977250MHz (21.47727MHz/12) (joypad waitstates)
  NTSC frame rate   60.09880627Hz (21.477270MHz/(262*1364-4/2))
  NTSC interlace    30.xxxxxxxxHz (21.477270MHz/(525*1364))
```

## PAL版

```
  PAL crystal       17.7344750MHz (X1, type number D177F2)
  PAL color clock   4.43361875MHz (17.7344750MHz/4)   (generated by S-CLK chip)
  PAL master clock  21.2813700MHz (17.7344750MHz*6/5) (generated by S-CLK chip)
  PAL dot clock     5.32034250MHz (21.2813700MHz/4)   (generated by PPU chip)
  PAL cpu clock     3.54689500MHz (21.2813700MHz/6)   (without waitstates)
  PAL cpu clock     2.66017125MHz (21.2813700MHz/8)   (short waitstates)
  PAL cpu clock     1.77344750MHz (21.2813700MHz/12)  (joypad waitstates)
  PAL frame rate    50.00697891Hz (21.281370MHz/(312*1364))
  PAL interlace     25.xxxxxxxxHz (21.281370MHz/(625*1364+4/2))
```

## APU

```
  APU oscillator    24.576MHz (X2, type number 24.57MX)
  DSP sample rate   32000Hz   (24.576MHz/24/32)
  SPC700 cpu clock  1.024MHz  (24.576MHz/24)
  SPC700 timer 0+1  8000Hz    (24.576MHz/24/128)
  SPC700 timer 2    64000Hz   (24.576MHz/24/16)
  CIC clock         3.072MHz  (24.576MHz/8)
  Expansion Port    8.192MHz  (24.576MHz/3)
```

## CPUクロック

CPUのクロックは通常3.5MHzまたは2.6MHz（またはその混在）です。

```
  3.5MHz   Used for Fast ROM, most I/O ports, and internal CPU cycles
  2.6MHz   Used for Slow ROM, for WRAM, and for DMA/HDMA transfers
  1.7MHz   Used only for (some) Joypad I/O Ports
```

CPUはメモリリフレッシュのために（1364サイクルのスキャンラインあたり）40マスターサイクル一時停止し、事実上CPUは約3％遅くなります。また、DMA/HDMA転送を使用する場合にも、CPUは一時停止します。

Nintendo specifies the following ROM timings to be required:

```
  3.5MHz   use 120ns or faster ROM/EPROMs
  2.6MHz   use 200ns or faster ROM/EPROMs
```

## Dot Clock Notes

## カートリッジ内の外部デバイス
