# 処理の流れ

## `nall::main()`

ルートの関数は`/bsnes/target-bsnes/bsnes.cpp`の`nall::main`関数です。

```c++
auto nall::main(Arguments arguments) -> void {
    // ...

    program.create();

    Application::run();

    // ...
}
```

## `Program::create()`

`state().onMain`に`Program::main`をセットします。

```c++
// /bsnes/target-bsnes/program/program.cpp
auto Program::create() -> void {
  Emulator::platform = this;

  // ...
  if(gameQueue) load();
  if(startFullScreen && emulator->loaded()) {
    toggleVideoFullScreen();
  }
  Application::onMain({&Program::main, this});
}

// /hiro/core/application.cpp
auto Application::onMain(const function<void ()>& callback) -> void {
  state().onMain = callback;
}
```

## `Application::run()`

`Program::create`で`state().onMain`にセットされた`Program::main`を呼び出してエミュレータのメインの処理に入ります。

`Program::main`はエミュレータのメイン処理を逐次実行する関数で、ループ自体は`pApplication::run`のwhile文で行っています。

```c++
// /hiro/core/application.cpp
auto Application::run() -> void {
  return pApplication::run();
}

// /hiro/qt/application.cpp
auto pApplication::run() -> void {
  if(Application::state().onMain) {
    while(!Application::state().quit) {
      Application::doMain();
      processEvents();
    }
  } else {
    QApplication::exec();
  }
}

// /hiro/core/application.cpp
auto Application::doMain() -> void {
  if(state().onMain) return state().onMain();
}
```

## `Program::main()`

```c++
// /bsnes/target-bsnes/program/program.cpp
auto Program::main() -> void {
  updateStatus();
  video.poll();

  if(Application::modal()) {
    audio.clear();
    return;
  }

  inputManager.poll();
  inputManager.pollHotkeys();

  static bool previouslyInactive = true;
  bool currentlyInactive = inactive();

  // check if emulator has transitioned from active to inactive or vice versa
  if(previouslyInactive != currentlyInactive) {
    previouslyInactive = currentlyInactive;
    Application::setScreenSaver(currentlyInactive || settings.general.screenSaver);
  }

  if(currentlyInactive) {
    audio.clear();
    usleep(20 * 1000);
    if(settings.emulator.runAhead.frames == 0) viewportRefresh();
    return;
  }

  rewindRun();

  if(!settings.emulator.runAhead.frames || fastForwarding || rewinding) {
    emulator->run();
  } else {
    emulator->setRunAhead(true);
    emulator->run();
    auto state = emulator->serialize(0);
    if(settings.emulator.runAhead.frames >= 2) emulator->run();
    if(settings.emulator.runAhead.frames >= 3) emulator->run();
    if(settings.emulator.runAhead.frames >= 4) emulator->run();
    emulator->setRunAhead(false);
    emulator->run();
    state.setMode(serializer::Mode::Load);
    emulator->unserialize(state);
  }

  if(emulatorSettings.autoSaveMemory.checked()) {
    auto currentTime = chrono::timestamp();
    if(currentTime - autoSaveTime >= settings.emulator.autoSaveMemory.interval) {
      autoSaveTime = currentTime;
      emulator->save();
    }
  }
}

// /bsnes/sfc/interface/interface.cpp
auto Interface::run() -> void {
  system.run();
}

// /bsnes/sfc/system/system.cpp
auto System::run() -> void {
  scheduler.mode = Scheduler::Mode::Run;
  scheduler.enter();
  if(scheduler.event == Scheduler::Event::Frame) frameEvent();
}
```

## `Scheduler::enter()`

```c++
struct Scheduler {
    // ...

    cothread_t active = nullptr;

    // ...

    auto enter() -> void {
      host = co_active();
      co_switch(active);
    }
}
```

`scheduler.active`には`System::power`で値がセットされています。

```c++
// bsnes/sfc/system/system.cpp
auto System::power(bool reset) -> void {
    // ...

    scheduler.active = cpu.thread;

    // ...
}
```

さらに`cpu.thread`には`Thread::create`で値がセットされています。

```c++
// bsnes/sfc/sfc.hpp
namespace SuperFamicom {
    struct Thread {
        auto create(auto (*entrypoint)() -> void, uint frequency_) -> void {
            if(!thread) {
                thread = co_create(Thread::Size, entrypoint);
            } else {
                thread = co_derive(thread, Thread::Size, entrypoint);
            }
            frequency = frequency_;
            clock = 0;
        }
    }
}
```

この`Thread::create`は`CPU::power`で呼び出されています。

```c++
// /bsnes/sfc/cpu/cpu.cpp
auto CPU::power(bool reset) -> void {
  WDC65816::power();
  Thread::create(Enter, system.cpuFrequency());

  // ...
}
```

よって、実質的なメイン処理は`CPU::Enter`ということになります。

## `CPU::Enter()`

```c++
// /bsnes/sfc/cpu/cpu.cpp
auto CPU::Enter() -> void {
  while(true) {
    scheduler.synchronize();
    cpu.main();
  }
}

// /bsnes/sfc/cpu/cpu.cpp
auto CPU::main() -> void {
  if(r.wai) return instructionWait();
  if(r.stp) return instructionStop();
  if(!status.interruptPending) return instruction();

  if(status.nmiPending) {
    status.nmiPending = 0;
    r.vector = r.e ? 0xfffa : 0xffea;
    return interrupt();
  }

  if(status.irqPending) {
    status.irqPending = 0;
    r.vector = r.e ? 0xfffe : 0xffee;
    return interrupt();
  }

  if(status.resetPending) {
    status.resetPending = 0;
    for(uint repeat : range(22)) step<6,0>();  //step(132);
    r.vector = 0xfffc;
    return interrupt();
  }

  status.interruptPending = 0;
}
```

`CPU::main`の`instruction()`で命令を1ステップ実行しています。

ようやく命令を実行しているところにたどり着けました。

