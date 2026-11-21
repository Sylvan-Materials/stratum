

Loading model... |-\|/-\|/-\|/-\|/-\|/-\|/-\|/ 

▄▄ ▄▄
██ ██
██ ██  ▀▀█▄ ███▄███▄  ▀▀█▄    ▄████ ████▄ ████▄
██ ██ ▄█▀██ ██ ██ ██ ▄█▀██    ██    ██ ██ ██ ██
██ ██ ▀█▄██ ██ ██ ██ ▀█▄██ ██ ▀████ ████▀ ████▀
                                    ██    ██
                                    ▀▀    ▀▀

build      : b1-307e3f1
model      : .cache/models/qwen2.5-coder-7b-instruct-q4_k_m.gguf
ftype      : Q4_K - Medium
modalities : text

available commands:
  /exit or Ctrl+C     stop or exit
  /regen              regenerate the last response
  /clear              clear the chat history
  /read <file>        add a text file
  /glob <pattern>     add text files using globbing pattern



> # Stratum Pipeline Stage: ISO C++ Specification Distiller

You are the Stratum Language Specification Distiller. Your sole mandate is to compress raw ISO C++ Working Draft LaTeX text into a concise, high-signal Markdown constraint sheet for downstream C++26/29 code-synthesis agents.

## 1. Strict Grounding & Anti-Hallucination Rules (Mandatory)
- **Zero External Invention:** Extract grammar rules, standard library names, headers, and `__cpp_*` feature-test macros **exclusively** from the provide ... (truncated)
### Core Grammar & Syntax Rules

- **Operators:**
  - `[]` (subscript operator)
  - `()` (function call operator)

- **Expressions:**
  - `vertex_value(g, uid)` (returns a value associated with a vertex)
  - `out_edges(g, *find_vertex(g, uid))` (returns incident edges of a vertex)
  - `edge_value(g, uv)` (returns a value associated with an edge)
  - `target_id(g, uv)` (returns the target id of an edge)

- **Declarations:**
  - `compressed_graph<VId, EIndex, EV, VV, GV, EProj = identity, VProj = identity>` (template class for compressed graph)
  - `operator[](vertex_id_t<G>)` (not provided; use `vertex_value(g, uid)` and `out_edges(g, *find_vertex(g, uid))`)

### Compiler Traps & Ill-Formed Constructs

- **No external invention:** Extract grammar rules, standard library names, headers, and `__cpp_*` feature-test macros exclusively from the provided text.
- **Exact grammar & keyword order:** Preserve exact keyword ordering and punctuation (e.g., `template`, `consteval`, or `constexpr`).
- **No parroting:** Do not repeat instructions from this prompt in your output.

### Standard-Year Purity Rules

- Avoid legacy C++ template metaprogramming patterns superseded by the facilities introduced in this chapter, such as custom type traits for graph properties.

### Feature-Test Macros & Header Requirements

- `<compressed_graph.hpp>`
- `__cpp_concepts`
- `__cpp_constexpr`

[ Prompt: 19.9 t/s | Generation: 5.4 t/s ]


Exiting...
