# FLUX Visual Editor

A browser-based node editor that lets you compose FLUX programs visually and compile them to FLUX assembly and bytecode.

**🔗 [SuperInstance/flux-visual-editor](https://github.com/SuperInstance/flux-visual-editor)**

## What It Is

The visual programming layer for FLUX. Instead of writing assembly by hand, you drag nodes onto a canvas, wire them together, and hit **Compile**. The editor generates valid FLUX assembly, assembles it to bytecode, and can execute it on the embedded JS FLUX VM — all in a single HTML file, no build tools required.

This is the 4th layer from [NEXT_HORIZONS.md](https://github.com/SuperInstance/flux/blob/main/NEXT_HORIZONS.md) — lowering the barrier from "write assembly" to "compose behavior."

## Quick Start

```bash
# Clone and open
git clone https://github.com/SuperInstance/flux-visual-editor.git
cd flux-visual-editor
open index.html   # or just double-click index.html
```

That's it. No npm, no build step, no dependencies. Just a browser.

## Node Types

| Node | Visual | Compiles To |
|------|--------|-------------|
| **Constant** | 📦 Value node | `MOVI` |
| **Arithmetic** | 🔧 ADD/SUB/MUL/DIV/MOD | `IADD`/`ISUB`/`IMUL`/`IDIV`/`IMOD` |
| **Compare** | 🔍 A vs B | `CMP` |
| **Branch** | 🔀 if/else | `JE`/`JNE`/`JZ`/`JNZ`/`JMP` |
| **Memory** | 💾 Store/Load | `STORE`/`LOAD` |
| **Stack** | 📚 Push/Pop/Dup | `PUSH`/`POP`/`DUP` |
| **Output** | 📤 Display value | Register read (shown in output panel) |
| **Halt** | ⏹ Stop | `HALT` |

## Features

- **Drag-and-drop canvas** — drag node types from the left palette onto the workspace
- **Wire connections** — click an output port, then click an input port to connect nodes
- **Compile** — generates FLUX assembly text and bytecode hex
- **Run** — executes bytecode on the embedded FLUX VM, shows register state and output
- **Example programs**:
  - **Hello World** — constant → output → halt
  - **Counter Loop** — PUSH/POP loop pattern
  - **Deadband** — if/else branches with CMP + JNE
  - **Factorial** — multiply loop with JNZ

## Architecture

```
index.html
├── Embedded FLUX VM (trimmed from flux.js)
│   ├── FluxVM class — bytecode interpreter
│   └── assemble() — text-to-bytecode assembler
├── Visual Editor
│   ├── Node palette (drag source)
│   ├── SVG canvas (workspace + wire rendering)
│   ├── Node definitions (compile rules per type)
│   └── Topological sort → assembly generation
└── Output panels (assembly, hex, registers, console)
```

Single file. No frameworks. No dependencies. Vanilla JS + SVG.

## Uses the FLUX VM

The VM is embedded directly from [flux-js](https://github.com/SuperInstance/flux-js), supporting:
- 16 general-purpose registers (R0–R15)
- Condition flags (zero, sign) for branches
- Stack operations (PUSH/POP/DUP)
- Memory (LOAD/STORE, 256-cell address space)
- Full opcode set (arithmetic, logic, jumps, halt)

## Related

- [FLUX](https://github.com/SuperInstance/flux) — main repo, spec, multi-language VMs
- [flux-js](https://github.com/SuperInstance/flux-js) — JavaScript VM + assembler + vocabulary interpreter
- [NEXT_HORIZONS.md](https://github.com/SuperInstance/flux/blob/main/NEXT_HORIZONS.md) — roadmap including this editor

## License

MIT
