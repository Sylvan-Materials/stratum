### A. Core Grammar & Syntax Rules

- `namespace std::graph`
  - `template <typename G> auto vertices(G&& g);` (Returns a range of vertex identifiers)
  - `template <typename G> auto vertex_id(G&& g, vertex_iterator_t<G>);` (Returns the identifier for a given vertex iterator)
  - `template <typename G> auto vertex_value(G&& g, vertex_t<G>);` (Returns the value associated with a given vertex)

### B. Compiler Traps & Ill-Formed Constructs

- The function templates must be used within their respective namespaces (`std::graph`).
- The return types of these functions are constrained to specific ranges and values.
- `vertex_id` requires an iterator type that is compatible with the graph's vertex range.

### C. Standard-Year Purity Rules (Anti-Drift)

- Avoid using legacy template metaprogramming or type-trait patterns for defining concepts, traits, and operations related to graphs.

### D. Feature-Test Macros & Header Requirements

- `<graph>`
- `__cpp_concepts`
- `__cpp_ranges`

