# FLUX Visual Editor

> A browser-based node editor for composing FLUX programs visually. Drag nodes, wire them together, compile to bytecode. No build tools required.

[![CI](https://github.com/SuperInstance/flux-visual-editor/actions/workflows/ci.yml/badge.svg)](https://github.com/SuperInstance/flux-visual-editor/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/SuperInstance/flux-visual-editor)](LICENSE)

The visual programming layer for FLUX conservation policies. Instead of writing assembly by hand, drag nodes onto a canvas, wire them together, and hit **Compile**. The editor generates valid FLUX assembly, assembles it to bytecode, and lets you execute it on the embedded FLUX VM — all in a single HTML file.

## What's New in v3

This is a major upgrade focused on **SuperInstance conservation policy composition**:

- **New node types**: Conservation Budget, Action Gate, Decision Point, LLM Call, Room Protocol, I/O Port
- **Conservation Ledger**: Live preview of all budgets enforced by your graph
- **Download .bin**: Export compiled programs as FLX0 binary files
- **3 new examples**: Token Budget Enforcer, Rate Limiter, Room Entry Protocol
- **Redesigned UI**: GitHub-inspired dark theme with categorized palette
- **Colored wires**: Connection colors match source node type
- **16 tests** in CI covering VM operations and node compilation

## Quick Start

```bash
git clone https://github.com/SuperInstance/flux-visual-editor.git
cd flux-visual-editor
open index.html       # macOS
# xdg-open index.html # Linux
# Or just double-click index.html
```

No npm. No build step. No dependencies. Just a browser.

## Using the Editor

1. **Drag nodes** from the left palette onto the canvas (organized by category)
2. **Wire connections** — click an output port (●), then click an input port (●)
3. **Configure values** — click node fields to edit constants/parameters
4. **Compile** — generates FLUX assembly + bytecode hex in the right panel
5. **Run** — executes bytecode on the embedded VM, shows register state
6. **Download .bin** — saves the FLX0 binary for deployment
7. **Export/Import/Share** — save programs as JSON or share via URL

## Node Types

### Conservation Nodes

| Node | Icon | Purpose | Compiles To |
|------|------|---------|-------------|
| **Conservation Budget** | 🛡 | Defines a bounded quantity (tokens, actions, time, calls) | `MOVI` + `STORE` to memory address |
| **Action Gate** | 🚦 | Checks budget, permits or blocks an action | `LOAD` + `CMP` + conditional `HALT` or `ISUB` + `STORE` |
| **Decision Point** | 🔀 | If/else branch on a comparison | `CMP` + `JE`/`JNE` |

### Agent & Room Nodes

| Node | Icon | Purpose | Compiles To |
|------|------|---------|-------------|
| **LLM Call** | 🤖 | Invokes a model with bounded parameters | `MOVI` (token budget) + `PUSH` + `CALL` stub |
| **Room Protocol** | 💬 | Send/receive PLATO room messages | `MOVI` (action code) + `PUSH` |

### Flow Nodes

| Node | Icon | Purpose | Compiles To |
|------|------|---------|-------------|
| **I/O Port** | 埠 | Entry and exit points for the program | `MOVI` (input/output markers) |
| **Constant** | 📦 | Static value in a register | `MOVI` |
| **Comparator** | 🔍 | Compare two registers (==, !=, <, >, <=, >=) | `CMP` |

### Low-Level Nodes

| Node | Icon | Purpose | Compiles To |
|------|------|---------|-------------|
| **Arithmetic** | 🔧 | Binary math (+, −, ×, ÷, mod) | `IADD`/`ISUB`/`IMUL`/`IDIV`/`IMOD` |
| **Branch** | ↪ | Jump instructions | `JE`/`JNE`/`JZ`/`JNZ`/`JMP` |
| **Memory** | 💾 | Read/write memory | `STORE`/`LOAD` |
| **Stack** | 📚 | Stack operations | `PUSH`/`POP`/`DUP` |
| **Output** | 📊 | Display a register value | Register read (comment) |
| **Halt** | ⏹ | Stop execution | `HALT` |

## Conservation Ledger

The ledger panel (right side) shows a live summary of all budgets defined in your graph:

- **Name** — The budget identifier
- **Type** — tokens, actions, time, or calls
- **Limit** — The maximum value

Updates automatically as you add, remove, or edit Budget nodes.

## Example Programs

### Token Budget Enforcer
A simple conservation gate: a 1000-token budget with 100 tokens consumed per LLM call. After 10 calls, the gate blocks further actions. Demonstrates Budget → Gate → LLM flow.

### Rate Limiter
Time-windowed rate limiting: max 5 actions per time window. Uses a comparator to check if the action count exceeds the time budget. Demonstrates Budget × 2 + Comparator + Decision flow.

### Room Entry Protocol
Full PLATO room interaction: check budget → enter room → invoke LLM → send result → exit room. Demonstrates Budget → Gate → Room(enter) → LLM → Room(send) → Room(exit) flow.

### Classic Examples (from v2)
- **Hello World** — Constant → Output → Halt
- **Counter Loop** — Stack-based iteration
- **Deadband Controller** — If/else branching with CMP
- **Factorial** — Multiply loop with JNZ

## Architecture

```
index.html (single file, no dependencies)
│
├── Embedded FLUX VM (from flux-js, trimmed)
│   ├── FluxVM class — bytecode interpreter
│   │   ├── 16 general-purpose registers (R0–R15)
│   │   ├── Condition flags (zero, sign)
│   │   ├── Stack operations
│   │   ├── Memory (256-cell address space)
│   │   └── Full opcode set (35+ instructions)
│   └── assemble() — text-to-bytecode assembler
│       └── Two-pass with label resolution
│
├── Visual Editor
│   ├── Node palette (left — categorized: Conservation, Agents, Flow, Low-Level)
│   ├── SVG canvas (workspace + colored wire rendering)
│   ├── Node definitions (compile rules per type)
│   ├── Wire manager (port connections, cycle detection)
│   ├── Topological sort → assembly generation
│   └── Conservation ledger (live budget preview)
│
├── Binary Export
│   └── FLX0 format: magic(4) + version(2) + length(4) + bytecode + checksum(2)
│
└── Output Panels (right side)
    ├── Assembly text (human-readable FLUX source)
    ├── Bytecode hex (addressed dump)
    ├── Conservation Ledger (budget summary)
    ├── Execution state (registers, flags, stack)
    └── Console (log + execution trace)
```

## FLX0 Binary Format

Downloaded `.bin` files use the FLX0 container:

```
Offset  Size  Field
0       4     Magic: "FLX0" (0x46 0x4C 0x58 0x30)
4       2     Version (little-endian uint16)
6       4     Bytecode length (little-endian uint32)
10      N     Bytecode
10+N    2     Checksum (sum of all bytecode bytes mod 0xFFFF)
```

## Testing

CI runs on every push and pull request via GitHub Actions:

```bash
# Tests cover:
# - VM arithmetic (ADD, SUB, MUL)
# - Memory operations (STORE, LOAD)
# - Stack operations (PUSH, POP)
# - Branching (CMP, JE, JNE)
# - INC/DEC
# - Factorial computation (integration test)
# - Node definition existence (all 8+ types)
# - Node compilation output validation
```

## Ecosystem

### FLUX Runtime
- [flux-js](https://github.com/SuperInstance/flux-js) — JavaScript VM + assembler (the VM embedded in this editor)
- [flux-vm](https://github.com/SuperInstance/flux-vm) — Python VM
- [flux-core](https://github.com/SuperInstance/flux-core) — Rust VM

### Policies
- [flux-registry](https://github.com/SuperInstance/flux-registry) — Pre-compiled policy registry
- [conservation-enforcer](https://github.com/SuperInstance/conservation-enforcer) — Conservation-law enforcement for LLM outputs
- [flux-policy-tester](https://github.com/SuperInstance/flux-policy-tester) — Testing framework

### Philosophy
- [AI-Writings](https://github.com/SuperInstance/AI-Writings) — Essays, fiction, poetry
- [NEXT_HORIZONS](https://github.com/SuperInstance/SuperInstance/blob/main/NEXT_HORIZONS.md) — Strategy

## Philosophy

Assembly language is powerful but inaccessible. Most operators who need conservation policies — the people who need to say "block any output that repeats more than 30%" — are not assembly programmers. The visual editor bridges that gap: compose behavior as a flowchart, compile to bytecode, deploy to production.

This is the accessibility layer of the FLUX ecosystem. The conservation-enforcer enforces policies. The flux-registry distributes them. The flux-policy-tester verifies them. The visual editor *creates* them.

## License

MIT — see [LICENSE](LICENSE).
