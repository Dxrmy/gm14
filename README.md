# gm14

**gm14 is a standalone C++17 runtime that plays compiled GameMaker: Studio 1.4
games (`data.win`) on modern hardware.** The same portable core also powers the
Nintendo 3DS front-end in [`gm14-3ds`](https://github.com/Dxrmy/gm14-3ds).

GameMaker 1.4 ships games as a single IFF container (`FORM ... data.win`)
holding every asset and all logic already lowered to GameMaker bytecode. gm14
parses that container, decodes the bytecode, and executes it in a small virtual
machine — no recompilation, no GameMaker required.

This repository is the **platform-agnostic core**: standard C++17 with **zero
console or OS SDK headers**. The Nintendo 3DS back-end lives in `gm14-3ds`, and
is wired in there as a submodule at `tools/gm14/core`.

## Features

- **Full `data.win` parser** — every IFF chunk: `GEN8`, `STRG`, `TXTR`, `TPAG`,
  `SPRT`, `BGND`, `OBJT`, `ROOM`, `CODE`, `VARI`, `FUNC`, and the rest.
- **Bytecode VM** — stack machine for bytecode v15/16 with variables, arrays,
  scripts, instance/room lifecycle, alarms, persistence and ~130 builtins.
- **Reference-chain resolution** — decodes the `VARI`/`FUNC` pointer chains and
  resolves string literals to `STRG` indices.
- **Static room renderer** — composites backgrounds, tiles and sprites to an RGBA
  framebuffer (PNG output for the host driver).
- **Portable by construction** — builds on Linux/macOS/Windows with only a C++17
  compiler; no console SDK, no engine dependency.

Verified against **Undertale 1.0.0.1539** (bytecode v16):

```
code entries      : 6272
instructions      : 891526
variables/functions: 10899 / 427
resolved var/fn refs: 206598 / 78929
sprites/objects/rooms: 2583 / 1709 / 336
```

## Contents

| File | Purpose |
|---|---|
| `dw.cpp` / `dw.hpp` | `data.win` IFF chunk parser, asset decode, room compositor |
| `vm.cpp` / `vm.hpp` | Bytecode VM, opcode executor, instance pool |
| `main_host.cpp` | PC driver: parse a `data.win`, print stats, render rooms to PNG |
| `vm_host.cpp` | Headless VM driver: run a room for N frames, dump a frame |
| `stb_image.h` / `stb_image_write.h` | Single-header image I/O |
| `Makefile.host` | g++/clang++ build for Linux/macOS |

## Build & run

```bash
make -f Makefile.host
./gm14_host path/to/data.win            # print parser stats
./gm14_host path/to/data.win 6 4        # render rooms #6 and #4 -> out_room_*.png
./vm_host   path/to/data.win 6 120      # run room #6 for 120 frames
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

## Related

- **[gm14-3ds](https://github.com/Dxrmy/gm14-3ds)** — Nintendo 3DS front-end
  (citro3d/citro2d, hid input, ndsp audio) that consumes this core.

## License

MIT — see `LICENSE`.
