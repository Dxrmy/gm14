# gm14

Platform-agnostic C++17 runtime for compiled **GameMaker: Studio 1.4** games
(`data.win`, cookie `FORM`, bytecode v15/16). It parses every IFF chunk, decodes
the `CODE` bytecode, resolves `VARI`/`FUNC` reference chains, and executes a
bytecode VM over object events (`Create`/`Step`/`Draw`/`Alarm`/`Other`).

This repository contains **only the portable core** — no libctru / Citro3D /
console SDK headers. The Nintendo 3DS front-end lives in `gm14-3ds`.

## Contents

| File | Purpose |
|---|---|
| `dw.cpp` / `dw.hpp` | `data.win` IFF chunk parser, asset decode, room compositor |
| `vm.cpp` / `vm.hpp` | Bytecode VM, opcode executor, instance pool |
| `main_host.cpp` | PC host driver: parse a `data.win`, print stats, render rooms to PNG |
| `vm_host.cpp` | Headless VM driver (run a room for N frames, save frames) |
| `stb_image.h` / `stb_image_write.h` | Single-header image I/O |
| `Makefile.host` | g++/clang++ build for Linux/macOS |

## Build & run

```bash
make -f Makefile.host
./gm14_host path/to/data.win            # print stats
./gm14_host path/to/data.win 6 4        # render room #6 and #4 -> out_room_*.png
./vm_host  path/to/data.win 6 120       # run room #6 for 120 frames
```

## Format notes

* Instruction header: `[Kind:8][Type2:4][Type1:4][CmpKind:8][low16:16]`.
* Sizes: push Int32/Variable/String = 2 words, Double/Int64 = 3, Int16 = 1;
  pop.variable = 2; call = 2; break.i = 2; everything else = 1.
* Branch offsets are instruction-word offsets relative to the branch.
* Variable/function refs form a **chain**: operand low 27 bits point at the next
  occurrence; the final one holds the STRG id. `VARI`/`FUNC` store the first
  occurrence + count.
* String literals in code are **STRG indices**.
* Array access pushes `value`, `instance_type`, `index` before `pop array`
  (and `instance_type`, `index` before `push array`).
* `TXTR` blobs are 0x80-aligned raw PNGs; size = next blob start − this start.

## License

MIT — see `LICENSE`.
