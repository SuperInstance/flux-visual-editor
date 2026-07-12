# FLUX Visual Editor

> A browser-based node editor for composing FLUX programs visually. Drag nodes, wire them together, compile to bytecode. No build tools required.

[![License](https://img.shields.io/github/license/SuperInstance/flux-visual-editor)](README.md)

The visual programming layer for FLUX. Instead of writing assembly by hand, you drag nodes onto a canvas, wire them together, and hit **Compile**. The editor generates valid FLUX assembly, assembles it to bytecode, and can execute it on the embedded JS FLUX VM — all in a single HTML file, no npm, no build step, no dependencies. Just a browser.

This is the 4th layer from [NEXT_HORIZONS.md](https://github.com/SuperInstance/flux/blob/main/NEXT_HORIZONS.md) — lowering the barrier from "write assembly" to "compose behavior." FLUX assembly is powerful but forbidding. The visual editor makes policy composition accessible to operators who think in flows, not opcodes.

## What It Does

The editor presents a drag-and-drop canvas where you compose FLUX programs from typed nodes. Each node represents a FLUX instruction category: constants (`MOVI`), arithmetic (`IADD`/`ISUB`/`IMUL`), comparison (`CMP`), branching (`JE`/`JNE`/`JMP`), memory (`STORE`/`LOAD`), stack operations (`PUSH`/`POP`/`DUP`), and control flow (`HALT`). You drag nodes from the left palette, click output ports then input ports to create wires, and the editor maintains the connection graph.

When you hit **Compile**, the editor performs topological sort on the graph and generates FLUX assembly text — the same `.flx` source format used throughout the SuperInstance ecosystem. A second pass through the embedded assembler converts this to FLUX bytecode hex, which you can execute directly in the browser on the integrated FLUX VM. The output panel shows register states, memory contents, and console output in real time.

The editor ships with four example programs that double as tutorials: **Hello World** (constant → output → halt), **Counter Loop** (PUSH/POP loop pattern), **Deadband** (if/else branches with CMP + JNE — the thermostat controller), and **Factorial** (multiply loop with JNZ). Each example loads with its nodes pre-wired so you can immediately compile and run it.

## Quick Start

```bash
# Clone and open — that's it
git clone https://github.com/SuperInstance/flux-visual-editor.git
cd flux-visual-editor
open index.html   # macOS
# Or: xdg-open index.html   # Linux
# Or: just double-click index.html in your file browser
```

No npm. No build step. No dependencies. Just a browser.

### Using the Editor

1. **Drag nodes** from the left palette onto the canvas
2. **Wire connections** — click an output port, then click an input port
3. **Configure values** — click a node to edit its constants/parameters
4. **Compile** — generates FLUX assembly + bytecode hex
5. **Run** — executes bytecode on the embedded VM, shows register state and output
6. **Export** — copy the assembly text or bytecode hex for use elsewhere

## Node Types

| Node | Visual | Compiles To | Description |
|------|--------|-------------|-------------|
| **Constant** | 📦 | `MOVI` | Load an immediate value into a register |
| **Arithmetic** | 🔧 | `IADD`/`ISUB`/`IMUL`/`IDIV`/`IMOD` | Binary math operations |
| **Compare** | 🔍 | `CMP` | Compare two registers, sets flags |
| **Branch** | 🔀 | `JE`/`JNE`/`JZ`/`JNZ`/`JMP` | Conditional/unconditional jumps |
| **Memory** | 💾 | `STORE`/`LOAD` | Write/read to memory addresses |
| **Stack** | 📚 | `PUSH`/`POP`/`DUP` | Stack manipulation |
| **Output** | 📤 | Register read | Display a value in the output panel |
| **Halt** | ⏹ | `HALT` | Stop execution |

## Architecture

```
index.html (single file, no dependencies)
│
├── Embedded FLUX VM (trimmed from flux-js)
│   ├── FluxVM class — bytecode interpreter
│   │   ├── 16 general-purpose registers (R0–R15)
│   │   ├── Condition flags (zero, sign)
│   │   ├── Stack operations
│   │   ├── Memory (256-cell address space)
│   │   └── Full opcode set
│   └── assemble() — text-to-bytecode assembler
│       └── Two-pass with label resolution
│
├── Visual Editor
│   ├── Node palette (left panel — drag source)
│   ├── SVG canvas (workspace + wire rendering)
│   ├── Node definitions (compile rules per type)
│   ├── Wire manager (port connections, cycle detection)
│   └── Topological sort → assembly generation
│
└── Output Panels (right side)
    ├── Assembly text (human-readable FLUX source)
    ├── Bytecode hex (FLX0 binary, base64)
    ├── Register state (R0–R15, flags)
    └── Console (output + execution trace)
```

Single file. No frameworks. No dependencies. Vanilla JS + SVG.

## Example Programs

### Hello World
The simplest program: a constant loaded into a register, displayed as output, then halt. Teaches the basic node → wire → compile → run flow.

### Counter Loop
Demonstrates stack-based iteration using `PUSH`/`POP`. A counter increments in a loop until it reaches a target value, then falls through to halt.

### Deadband (Thermostat)
The canonical FLUX example: a thermostat controller with hysteresis. Uses `CMP` to compare temperature against thresholds, `JNE` to branch between heating, cooling, and idle states. This is the same policy published in the [flux-registry](https://github.com/SuperInstance/flux-registry) as `deadband-controller`.

### Factorial
Computes N! using a multiply loop with `JNZ`. Demonstrates conditional backward jumps for iteration, register management for accumulator and loop counter.

## Testing

The visual editor is a single static HTML file with no test framework. Testing is manual:

1. **Open `index.html`** in a browser
2. **Load each example** from the Examples dropdown
3. **Compile** — verify assembly output matches expected
4. **Run** — verify register states and console output match expected
5. **Modify** — add/remove nodes, re-wire, recompile to test edge cases

For automated testing of the FLUX VM itself (which is embedded from flux-js), use the [flux-policy-tester](https://github.com/SuperInstance/flux-policy-tester) framework.

## Philosophy

Assembly language is powerful but inaccessible. Most operators who need conservation policies — the people who need to say "block any output that repeats more than 30%" — are not assembly programmers. The visual editor bridges that gap: compose behavior as a flowchart, compile to bytecode, deploy to production.

This is the accessibility layer of the FLUX ecosystem. The [conservation-enforcer](https://github.com/SuperInstance/conservation-enforcer) enforces policies. The [flux-registry](https://github.com/SuperInstance/flux-registry) distributes them. The [flux-policy-tester](https://github.com/SuperInstance/flux-policy-tester) verifies them. The visual editor *creates* them — and in doing so, makes conservation enforcement accessible to anyone who can draw a flowchart.

For the bigger picture, see [AI-Writings](https://github.com/SuperInstance/AI-Writings) and [NEXT_HORIZONS](https://github.com/SuperInstance/flux/blob/main/NEXT_HORIZONS.md).

## Ecosystem

### FLUX Runtime
- [flux-js](https://github.com/SuperInstance/flux-js) — JavaScript VM + assembler (the VM embedded in this editor) — `npm install flux-js`
- [flux-vm](https://github.com/SuperInstance/flux-vm) — Python VM — `pip install flux-vm`
- [flux-core](https://github.com/SuperInstance/flux-core) — Rust VM — `cargo add fluxvm`

### Policies
- [flux-registry](https://github.com/SuperInstance/flux-registry) — Pre-compiled policy registry — `pip install flux-registry`
- [conservation-enforcer](https://github.com/SuperInstance/conservation-enforcer) — Conservation-law enforcement for LLM outputs
- [flux-policy-tester](https://github.com/SuperInstance/flux-policy-tester) — Testing framework for FLUX policies

### Philosophy
- [AI-Writings](https://github.com/SuperInstance/AI-Writings) — Essays, fiction, poetry
- [NEXT_HORIZONS](https://github.com/SuperInstance/SuperInstance/blob/main/NEXT_HORIZONS.md) — Strategy

## License

MIT — see [LICENSE](LICENSE).
