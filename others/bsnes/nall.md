# nallライブラリについて

`nall/nall.hpp`を見ると`nall`は次のような説明をされています。

> nallは、基本的なクラスと便利なクラスの両方を提供するヘッダライブラリで、移植性、一貫性、ミニマリズム、再利用性を目標としています。

つまりエミュ開発に特化したライブラリのようです。

```c++
#include <nall/platform.hpp>

#include <nall/algorithm.hpp>
#include <nall/any.hpp>
//#include <nall/arguments.hpp>
#include <nall/arithmetic.hpp>
#include <nall/array.hpp>
#include <nall/array-span.hpp>
#include <nall/array-view.hpp>
#include <nall/atoi.hpp>
#include <nall/bit.hpp>
#include <nall/chrono.hpp>
#include <nall/directory.hpp>
#include <nall/dl.hpp>
#include <nall/endian.hpp>
// ...
#include <nall/hash/crc16.hpp>
#include <nall/hash/crc32.hpp>
#include <nall/hash/crc64.hpp>
#include <nall/hash/sha256.hpp>

#if defined(PLATFORM_WINDOWS)
  #include <nall/windows/registry.hpp>
  #include <nall/windows/utf8.hpp>
#endif

#if defined(API_POSIX)
  #include <nall/serial.hpp>
#endif
```

