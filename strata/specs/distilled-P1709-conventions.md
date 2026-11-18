### A. Core Grammar & Syntax Rules

- `code{G}`: Graph.
- `code{graph_reference_t<G>`: Graph reference.
- `code{GV}`: Graph Value, value or reference.
- `code{EL}`: Edge list.
- `code{V}`: Vertex descriptor; `u` is the source vertex, `v` is the target vertex.
- `code{VId}`: Vertex id; `uid` is the source vertex id, `vid` is the target vertex id.
- `code{VV}`: Vertex Value, value or reference. Can be user-defined or returned by a function object.
- `code{VR}`: Vertex Range.
- `code{VI}`: Vertex Iterator; `ui` and `vi` are source/target iterators.
- `code{VVF}`: Vertex Value Function.
- `code{VProj}`: Vertex data projection function.
- `code{partition_id_t<G>`: Partition id.
- `code{P}`: Number of partitions.
- `code{PVR}`: Partition vertex range.
- `code{E}`: Edge descriptor; `uv` is an edge from vertices `u` to `v`, `vw` is an edge from vertices `v` to `w`.
- `code{EV}`: Edge Value, value or reference. Can be user-defined or returned by a function object.
- `code{ER}`: Edge Range for edges of a vertex.
- `code{EI}`: Edge Iterator; `uvi` and `vwi` are iterators for an edge from vertices `u` to `v`, `v` to `w`.
- `code{EVF}`: Edge Value Function.
- `code{EProj}`: Edge data projection function.

### B. Compiler Traps & Ill-Formed Constructs (Hard Prohibitions)

- No legacy C++ template metaprogramming or type-trait patterns are explicitly mentioned as being superseded by the facilities in this chapter.

### C. Standard-Year Purity Rules (Anti-Drift)

- Avoid using legacy C++ template metaprogramming or type-trait patterns that are superseded by the facilities introduced in this chapter.

### D. Feature-Test Macros & Header Requirements

- None specified in excerpt

