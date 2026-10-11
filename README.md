# gm14

**gm14 is a standalone C++17 runtime that plays compiled GameMaker: Studio 1.4 games (`data.win`) on modern hardware.** The same core also powers the Nintendo 3DS front-end in [`gm14-3ds`](https://github.com/Dxrmy/gm14-3ds) and the desktop runner in [`gm14-x86_64`](https://github.com/Dxrmy/gm14-x86_64).

A compiled GameMaker game is a single IFF container holding every asset plus all logic lowered to GameMaker bytecode. gm14 parses that container, decodes the bytecode, and executes it in a small virtual machine — no recompilation, no GameMaker required.

This repository is the **platform-agnostic core**: standard C++17 with clean platform abstractions.

---

## Features

- **`data.win` Parser** — reads every IFF chunk, including assets, rooms, paths, audio, and code across bytecode versions 15, 16, and 17.
- **Bytecode VM** — complete stack machine with variables, arrays, reference chains, user script shadowing, instance/room lifecycle, alarms, persistence, surfaces, and extended builtins.
- **Configurable Engine Architecture (`gm14.ini`)** — bounded LRU texture caching, bounded PCM audio budget, watchdog instruction budget, tile culling margin, blend modes, and target framerate.
- **Path Traversal Engine** — full GameMaker path system (`path_start`, `path_end`, `path_position`, `path_speed`, etc.) with two-pass entrance marker placement.
- **Sprite Transformation & Blend Modes** — origin-pivoted rotation for `draw_sprite_ext`, `draw_self`, and default instance drawing (`obj_winkstar`), CCW rotation matrix, and additive blend mode support (`bm_add`).
- **Audio & Music Engine** — native mixer with gain, pitch shift, looping, embedded Ogg Vorbis/WAV decoding (`stb_vorbis`), GameMaker `audio_*` builtins, and Undertale `caster_*` support.
- **100% Passing Test Suite** — **181 passing unit tests** covering all runtime subsystems.
- **Portable** — builds on Linux, macOS, and Windows with only a standard C++17 compiler.

Verified against **Undertale 1.0.0.1539** (bytecode v16):

```
code entries        : 6,272
instructions        : 891,526
variables/functions : 10,899 / 427
sprites/objects/rooms: 2,583 / 1,709 / 336
unit tests          : 181 passed, 0 failed
```

---

## Contents

| File | Purpose |
|---|---|
| `dw.cpp` / `dw.hpp` | `data.win` parser, chunk unpacker, asset decode, room renderer |
| `vm.cpp` / `vm.hpp` | Bytecode VM, opcode executor, instance pool, builtins |
| `audio.cpp` / `audio.hpp` | Audio engine, multi-voice mixer, PCM sound cache |
| `bunzip.cpp` / `bunzip.hpp` | Lightweight BZip2 decompression for compressed chunks |
| `qoi.hpp` | QOI texture decoding header |
| `gm14.ini.example` | Template engine configuration file |
| `vm_tests.cpp` | Comprehensive 181-test unit test suite |
| `main_runner.cpp` | Interactive desktop runner |
| `main_host.cpp` | PC driver: parse a `data.win`, print stats, render rooms to PNG |
| `vm_host.cpp` | Headless VM driver: run a room for N frames, dump a frame |
| `stb_image.h` / `stb_image_write.h` | Single-header image I/O |
| `stb_vorbis.c` / `stb_vorbis_impl.c` | Ogg Vorbis audio decoding |
| `Makefile` | MinGW / Linux build configuration |

---

## Configuration (`gm14.ini`)

Runtime behavior is configured via `gm14.ini` (see [`gm14.ini.example`](gm14.ini.example)):

```ini
[engine]
; Texture cache limit in sheets (Old 3DS = 2, low-end PC/Pi = 6, 0 = unlimited)
max_texture_pages = 6
; Audio PCM cache budget in MB (Old 3DS = 8, low-end PC/Pi = 16)
pcm_cache_budget_mb = 16
; Bytecode loop execution watchdog budget
instruction_budget = 2000000
; Safety margin in pixels for tile frustum culling
culling_margin = 32
; Enable additive blend modes (bm_add)
enable_blend_modes = 1
; Target FPS (0 = default room_speed)
fps = 30
; Save directory
save_directory = ./saves
; Diagnostic sink logging
verbose_logging = 0
```

---

## Build & Run

```bash
# Build test suite and runner
mingw32-make -j24

# Run the 181 unit tests
./vm_tests path/to/data.win

# Run the game
./gm14_runner.exe --data path/to/data.win
```

---

## Related Projects

- **[gm14-x86_64](https://github.com/Dxrmy/gm14-x86_64)** — Dedicated PC (Windows / Linux) runner and host harness.
- **[gm14-3ds](https://github.com/Dxrmy/gm14-3ds)** — Nintendo 3DS hardware port.

---

## License

MIT — see [`LICENSE`](LICENSE).
