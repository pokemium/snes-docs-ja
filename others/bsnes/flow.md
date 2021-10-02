# 処理の流れ

ルートの関数は`/bsnes/target-bsnes/bsnes.cpp`の`nall::main`関数です。

```c++
auto nall::main(Arguments arguments) -> void {
    // ...

    program.create();

    Application::run();

    // ...
}

// hiro/core/
auto Application::run() -> void {
  return pApplication::run();
}
```

