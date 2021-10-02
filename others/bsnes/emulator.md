# エミュレータのインターフェイス

[ソースコードについて](source.md)で、`/bsnes/emulator`にはエミュレータのコアのインターフェイスが書いてあると説明しました。

ここでは具体的にどのようなインターフェイスになっているかみていきます。

```c++
struct Interface {
  struct Information {
    string manufacturer;
    string name;
    string extension;
    bool resettable = false;
  };

  struct Display {
    struct Type { enum : uint {
      CRT,
      LCD,
    };};
    uint id = 0;
    string name;
    uint type = 0;
    uint colors = 0;
    uint width = 0;
    uint height = 0;
    uint internalWidth = 0;
    uint internalHeight = 0;
    double aspectCorrection = 0;
  };

  struct Port {
    uint id;
    string name;
  };

  struct Device {
    uint id;
    string name;
  };

  struct Input {
    struct Type { enum : uint {
      Hat,
      Button,
      Trigger,
      Control,
      Axis,
      Rumble,
    };};

    uint type;
    string name;
  };

  // general-purpose interface
  virtual auto information() -> Information { return {}; }
  virtual auto display() -> Display { return {}; }
  virtual auto color(uint32 color) -> uint64 { return 0; }

  // game(cartridge) interface
  virtual auto loaded() -> bool { return false; }
  virtual auto hashes() -> vector<string> { return {}; }
  virtual auto manifests() -> vector<string> { return {}; }
  virtual auto titles() -> vector<string> { return {}; }
  virtual auto title() -> string { return {}; }
  virtual auto load() -> bool { return false; }
  virtual auto save() -> void {}
  virtual auto unload() -> void {}

  // system interface
  virtual auto ports() -> vector<Port> { return {}; }
  virtual auto devices(uint port) -> vector<Device> { return {}; }
  virtual auto inputs(uint device) -> vector<Input> { return {}; }
  virtual auto connected(uint port) -> uint { return 0; }
  virtual auto connect(uint port, uint device) -> void {}
  virtual auto power() -> void {}
  virtual auto reset() -> void {}
  virtual auto run() -> void {}

  // time functions
  virtual auto rtc() -> bool { return false; }
  virtual auto synchronize(uint64 timestamp = 0) -> void {}

  // state functions
  virtual auto serialize(bool synchronize = true) -> serializer { return {}; }
  virtual auto unserialize(serializer&) -> bool { return false; }

  // cheat functions
  virtual auto read(uint24 address) -> uint8 { return 0; }
  virtual auto cheats(const vector<string>& = {}) -> void {}

  // configuration
  virtual auto configuration() -> string { return {}; }
  virtual auto configuration(string name) -> string { return {}; }
  virtual auto configure(string configuration = "") -> bool { return false; }
  virtual auto configure(string name, string value) -> bool { return false; }

  // settings
  virtual auto cap(const string& name) -> bool { return false; }
  virtual auto get(const string& name) -> any { return {}; }
  virtual auto set(const string& name, const any& value) -> bool { return false; }
  
  // misc
  virtual auto frameSkip() -> uint { return 0; }
  virtual auto setFrameSkip(uint frameSkip) -> void {}
  virtual auto runAhead() -> bool { return false; }
  virtual auto setRunAhead(bool runAhead) -> void {}
};

```

## 汎用インターフェイスについて

汎用的な機能のインターフェイスです。

```c++
  // general-purpose interface

  // コアがエミュレーションしているハードウェアの情報を返す
  virtual auto information() -> Information { return {}; }
  // コアがエミュレーションしているハードウェアのディスプレイの情報を返す
  virtual auto display() -> Display { return {}; }
  // 不明
  virtual auto color(uint32 color) -> uint64 { return 0; }
```

汎用インターフェイスの実装は`/bsnes/sfc/interface/interface.cpp`で行われています。

```c++
auto Interface::information() -> Information {
  Information information;
  information.manufacturer = "Nintendo";
  information.name         = "Super Famicom";
  information.extension    = "sfc";
  information.resettable   = true;
  return information;
}

auto Interface::display() -> Display {
  Display display;
  display.type   = Display::Type::CRT;
  display.colors = 1 << 19;
  display.width  = 256;
  display.height = 240;
  display.internalWidth  = 512;
  display.internalHeight = 480;
  display.aspectCorrection = 8.0 / 7.0;
  return display;
}

auto Interface::color(uint32 color) -> uint64 {
  uint r = color >>  0 & 31;
  uint g = color >>  5 & 31;
  uint b = color >> 10 & 31;
  uint l = color >> 15 & 15;

  // luma=0 is not 100% black; but it's much darker than normal linear scaling
  // exact effect seems to be analog; requires > 24-bit color depth to represent accurately
  double L = (1.0 + l) / 16.0 * (l ? 1.0 : 0.25);
  uint64 R = L * image::normalize(r, 5, 16);
  uint64 G = L * image::normalize(g, 5, 16);
  uint64 B = L * image::normalize(b, 5, 16);

  if(SuperFamicom::configuration.video.colorEmulation) {
    static const uint8 gammaRamp[32] = {
      0x00, 0x01, 0x03, 0x06, 0x0a, 0x0f, 0x15, 0x1c,
      0x24, 0x2d, 0x37, 0x42, 0x4e, 0x5b, 0x69, 0x78,
      0x88, 0x90, 0x98, 0xa0, 0xa8, 0xb0, 0xb8, 0xc0,
      0xc8, 0xd0, 0xd8, 0xe0, 0xe8, 0xf0, 0xf8, 0xff,
    };
    R = L * gammaRamp[r] * 0x0101;
    G = L * gammaRamp[g] * 0x0101;
    B = L * gammaRamp[b] * 0x0101;
  }

  return R << 32 | G << 16 | B << 0;
}
```

## ゲーム(カートリッジ)のインターフェイスについて

```c++
  // game(cartridge) interface
  virtual auto loaded() -> bool { return false; }
  virtual auto hashes() -> vector<string> { return {}; }
  virtual auto manifests() -> vector<string> { return {}; }
  virtual auto titles() -> vector<string> { return {}; }
  virtual auto title() -> string { return {}; }
  virtual auto load() -> bool { return false; }
  virtual auto save() -> void {}
  virtual auto unload() -> void {}
```

実装は`/bsnes/sfc/cartridge/cartridge.cpp`で行われています。

```c++
auto Cartridge::hashes() const -> vector<string> {
  vector<string> hashes;
  hashes.append(game.sha256);
  if(slotGameBoy.sha256) hashes.append(slotGameBoy.sha256);
  if(slotBSMemory.sha256) hashes.append(slotBSMemory.sha256);
  if(slotSufamiTurboA.sha256) hashes.append(slotSufamiTurboA.sha256);
  if(slotSufamiTurboB.sha256) hashes.append(slotSufamiTurboB.sha256);
  return hashes;
}

auto Cartridge::manifests() const -> vector<string> {
  vector<string> manifests;
  manifests.append(string{BML::serialize(game.document), "\n", BML::serialize(board)});
  if(slotGameBoy.document) manifests.append(BML::serialize(slotGameBoy.document));
  if(slotBSMemory.document) manifests.append(BML::serialize(slotBSMemory.document));
  if(slotSufamiTurboA.document) manifests.append(BML::serialize(slotSufamiTurboA.document));
  if(slotSufamiTurboB.document) manifests.append(BML::serialize(slotSufamiTurboB.document));
  return manifests;
}

auto Cartridge::titles() const -> vector<string> {
  vector<string> titles;
  titles.append(game.label);
  if(slotGameBoy.label) titles.append(slotGameBoy.label);
  if(slotBSMemory.label) titles.append(slotBSMemory.label);
  if(slotSufamiTurboA.label) titles.append(slotSufamiTurboA.label);
  if(slotSufamiTurboB.label) titles.append(slotSufamiTurboB.label);
  return titles;
}

auto Cartridge::title() const -> string {
  if(slotGameBoy.label) return slotGameBoy.label;
  if(has.MCC && slotBSMemory.label) return slotBSMemory.label;
  if(slotBSMemory.label) return {game.label, " + ", slotBSMemory.label};
  if(slotSufamiTurboA.label && slotSufamiTurboB.label) return {slotSufamiTurboA.label, " + ", slotSufamiTurboB.label};
  if(slotSufamiTurboA.label) return slotSufamiTurboA.label;
  if(slotSufamiTurboB.label) return slotSufamiTurboB.label;
  if(has.Cx4 || has.DSP1 || has.DSP2 || has.DSP4 || has.ST0010) return {"[HLE] ", game.label};
  return game.label;
}

auto Cartridge::load() -> bool {
  // ...

  return true;
}

auto Cartridge::save() -> void {
  saveCartridge(game.document);
  if(has.GameBoySlot) {
    icd.save();
  }
  if(has.BSMemorySlot) {
    saveCartridgeBSMemory(slotBSMemory.document);
  }
  if(has.SufamiTurboSlotA) {
    saveCartridgeSufamiTurboA(slotSufamiTurboA.document);
  }
  if(has.SufamiTurboSlotB) {
    saveCartridgeSufamiTurboB(slotSufamiTurboB.document);
  }
}

auto Cartridge::unload() -> void {
  rom.reset();
  ram.reset();
}
```

## システムインターフェイスについて

```c++
  // system interface

  // 物理的なポートの種類を返す
  virtual auto ports() -> vector<Port> { return {}; }
  // 引数で渡されたポートに対して使えるデバイスの種類を返す
  virtual auto devices(uint port) -> vector<Device> { return {}; }
  // 引数で渡されたデバイスから受け取れる入力の種類を返す
  virtual auto inputs(uint device) -> vector<Input> { return {}; }
  // 引数で渡されたポートに対して何かしらのデバイスが接続されているかどうか
  virtual auto connected(uint port) -> uint { return 0; }
  // 引数で渡されたポートにデバイスを接続する
  virtual auto connect(uint port, uint device) -> void {}
  // 不明
  virtual auto power() -> void {}
  // 不明
  virtual auto reset() -> void {}
  // 不明
  virtual auto run() -> void {}
```

実装は`/bsnes/sfc/interface/interface.cpp`で行われています。

```c++
auto Interface::ports() -> vector<Port> { return {
  {ID::Port::Controller1, "Controller Port 1"},
  {ID::Port::Controller2, "Controller Port 2"},
  {ID::Port::Expansion,   "Expansion Port"   }};
}

auto Interface::devices(uint port) -> vector<Device> {
  if(port == ID::Port::Controller1) return {
    {ID::Device::None,    "None"   },
    {ID::Device::Gamepad, "Gamepad"},
    {ID::Device::Mouse,   "Mouse"  }
  };

  if(port == ID::Port::Controller2) return {
    {ID::Device::None,          "None"          },
    {ID::Device::Gamepad,       "Gamepad"       },
    {ID::Device::Mouse,         "Mouse"         },
    {ID::Device::SuperMultitap, "Super Multitap"},
    {ID::Device::SuperScope,    "Super Scope"   },
    {ID::Device::Justifier,     "Justifier"     },
    {ID::Device::Justifiers,    "Justifiers"    }
  };

  if(port == ID::Port::Expansion) return {
    {ID::Device::None,        "None"       },
    {ID::Device::Satellaview, "Satellaview"},
    {ID::Device::S21FX,       "21fx"       }
  };

  return {};
}

auto Interface::inputs(uint device) -> vector<Input> {
  using Type = Input::Type;

  if(device == ID::Device::None) return {
  };

  if(device == ID::Device::Gamepad) return {
    {Type::Hat,     "Up"    },
    {Type::Hat,     "Down"  },
    {Type::Hat,     "Left"  },
    {Type::Hat,     "Right" },
    {Type::Button,  "B"     },
    {Type::Button,  "A"     },
    {Type::Button,  "Y"     },
    {Type::Button,  "X"     },
    {Type::Trigger, "L"     },
    {Type::Trigger, "R"     },
    {Type::Control, "Select"},
    {Type::Control, "Start" }
  };

  if(device == ID::Device::Mouse) return {
    {Type::Axis,   "X-axis"},
    {Type::Axis,   "Y-axis"},
    {Type::Button, "Left"  },
    {Type::Button, "Right" }
  };

  // ...

  return {};
}

auto Interface::connected(uint port) -> uint {
  if(port == ID::Port::Controller1) return settings.controllerPort1;
  if(port == ID::Port::Controller2) return settings.controllerPort2;
  if(port == ID::Port::Expansion) return settings.expansionPort;
  return 0;
}

auto Interface::connect(uint port, uint device) -> void {
  if(port == ID::Port::Controller1) controllerPort1.connect(settings.controllerPort1 = device);
  if(port == ID::Port::Controller2) controllerPort2.connect(settings.controllerPort2 = device);
  if(port == ID::Port::Expansion) expansionPort.connect(settings.expansionPort = device);
}

auto Interface::power() -> void {
  system.power(/* reset = */ false);
}

auto Interface::reset() -> void {
  system.power(/* reset = */ true);
}

auto Interface::run() -> void {
  system.run();
}
```

## タイマーインターフェイスについて

```c++
  // time functions

  // rtcを持っているか
  virtual auto rtc() -> bool { return false; }
  // 現実の現在時刻とエミュレータのRTCを同期する
  virtual auto synchronize(uint64 timestamp = 0) -> void {}
```

実装は`/bsnes/sfc/interface/interface.cpp`で行われています。

```c++
auto Interface::rtc() -> bool {
  if(cartridge.has.EpsonRTC) return true;
  if(cartridge.has.SharpRTC) return true;
  return false;
}

auto Interface::synchronize(uint64 timestamp) -> void {
  if(!timestamp) timestamp = chrono::timestamp();
  if(cartridge.has.EpsonRTC) epsonrtc.synchronize(timestamp);
  if(cartridge.has.SharpRTC) sharprtc.synchronize(timestamp);
}
```

## チートインタフェイスについて

```c++
  // cheat functions
  virtual auto read(uint24 address) -> uint8 { return 0; }
  virtual auto cheats(const vector<string>& = {}) -> void {}
```

実装は`/bsnes/sfc/interface/interface.cpp`で行われています。

```c++
auto Interface::read(uint24 address) -> uint8 {
  return cpu.readDisassembler(address);
}

auto Interface::cheats(const vector<string>& list) -> void {
  if(cartridge.has.ICD) {
    icd.cheats.assign(list);
    return;
  }

  //make all ROM data writable temporarily
  Memory::GlobalWriteEnable = true;

  Cheat oldCheat = cheat;
  Cheat newCheat;
  newCheat.assign(list);

  //determine all old codes to remove
  for(auto& oldCode : oldCheat.codes) {
    bool found = false;
    for(auto& newCode : newCheat.codes) {
      if(oldCode == newCode) {
        found = true;
        break;
      }
    }
    if(!found) {
      //remove old cheat
      if(oldCode.enable) {
        bus.write(oldCode.address, oldCode.restore);
      }
    }
  }

  //determine all new codes to create
  for(auto& newCode : newCheat.codes) {
    bool found = false;
    for(auto& oldCode : oldCheat.codes) {
      if(newCode == oldCode) {
        found = true;
        break;
      }
    }
    if(!found) {
      //create new cheat
      newCode.restore = bus.read(newCode.address);
      if(!newCode.compare || newCode.compare() == newCode.restore) {
        newCode.enable = true;
        bus.write(newCode.address, newCode.data);
      } else {
        newCode.enable = false;
      }
    }
  }

  cheat = newCheat;

  //restore ROM write protection
  Memory::GlobalWriteEnable = false;
}
```