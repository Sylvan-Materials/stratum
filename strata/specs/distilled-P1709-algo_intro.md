

Loading model... |-\|/-\|/-\|/-\|/-\|/-\|/-\|/-\|/-\|/-\|/ 

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

- `consteval` function declarations.
- `constexpr` variable declarations.
- `noexcept` specifier in function signatures.

### Compiler Traps & Ill-Formed Constructs (Hard Prohibitions)

- Algorithms that require a `stack` or `queue`.
- Algorithms that may throw exceptions, such as `bad_alloc`.

### Standard-Year Purity Rules (Anti-Drift)

- Avoid using legacy C++ template metaprogramming patterns for algorithms.

### Feature-Test Macros & Header Requirements

- `<cstddef>`: For `size_t`.
- `<stdexcept>`: For `std::bad_alloc`.

[ Prompt: 14.2 t/s | Generation: 5.3 t/s ]


Exiting...
