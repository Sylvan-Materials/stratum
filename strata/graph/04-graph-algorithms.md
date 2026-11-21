# Stratum Module Synthesis Contract: Core Graph Algorithms (Dijkstra & DFS)

You are the Stratum Automation Code Generator. Translate this specification into a single, production-grade, header-only C++26 component (`graph/algorithm/graph_algorithms.hpp`) adhering strictly to Stratum's design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- **Standard Headers & Core Includes:** Include `"graph/graph.hpp"`, `<algorithm>`, `<concepts>`, `<cstddef>`, `<cstdint>`, `<functional>`, `<limits>`, `<queue>`, `<ranges>`, `<stack>`, `<type_traits>`, `<utility>`, and `<vector>` explicitly at the top of the file.
- **Explicit Types & Trailing Returns:** Declare concrete variable types explicitly inside function bodies. Reserve `auto` strictly for trailing return placeholders (`-> void` or `-> decltype(auto)`) and concept-constrained return types. Never write `const auto` before a function name, and never place `auto` inside template argument lists or `requires(...)` parameter lists.
- **CPO-Generic Operation:** Never hardcode `dynamic_adjacency_graph` inside algorithm bodies; access vertices, edges, and IDs strictly through `graph::vertices(g)`, `graph::edges(g, u)`, `graph::vertex_id(g, u)`, `graph::target_id(g, e)`, and `std::ranges::size(graph::vertices(g))` so algorithms work across all seven Stratum container archetypes.
- **Vertex Count Lookup:** Never call graph::size(g) or graph::num_vertices(g) (neither exists in graph/graph.hpp). Always compute the vertex count via const std::size_t num_verts = static_cast<std::size_t>(std::ranges::distance(graph::vertices(g)));.
- **Standard Range Iterators Only:** Never call graph::begin(...) or graph::end(...) (neither exists in namespace graph). Always use std::ranges::begin(...) and std::ranges::end(...).
- **Always Qualify std:: Types & Declare Local Variables:** Always prefix standard library containers and integers with std:: (std::vector, std::stack, std::priority_queue, std::uint8_t, std::pair, std::greater), and always include the full type when declaring local variables (const std::size_t num_verts = graph::num_vertices(g);, never bare num_verts = ...;).


## 2. Target Component Archetype (`namespace graph` & `namespace graph::algorithm`)
- **Purpose:** Foundational shortest-path and traversal algorithms matching `stdgraph/P1709` signatures required by `amet` (`Suurballe.h`) and downstream parser/thresher ecosystems (`cnpm`, `winnow`, `*-thresher`).

## 3. Exact Top-to-Bottom Declaration Sequence inside `namespace graph` (Mandatory)

### 0. `container_value_fn` Property-Map Adaptor
Define `container_value_fn` directly inside `namespace graph` above `dijkstra_shortest_paths`:
```cpp
template <typename Container>
struct container_value_fn {
    Container* container_{nullptr};
    explicit constexpr container_value_fn(Container& c) noexcept : container_(&c) {}
    template <typename G, typename VId>
    constexpr auto operator()(const G&, const VId& uid) const -> decltype(auto) {
        return (*container_)[static_cast<std::size_t>(uid)];
    }
};
```

### 1. `dijkstra_shortest_paths` (Required by `amet/Suurballe.h`)
Implement the `P1709`-compatible `dijkstra_shortest_paths` function templates inside `namespace graph` (and alias into `namespace graph::algorithm`):
- **Primary Overload (Multi-Source & Single-Source):**
  ```cpp
  template <
      typename G,
      typename Sources,
      typename Distances,
      typename Predecessors,
      typename WF = std::function<double(const G&, const graph::edge_t<G>&)>,
      typename Alloc = std::allocator<std::byte>
  >
  auto dijkstra_shortest_paths(
      G&& g,
      const Sources& sources,
      Distances& distances,
      Predecessors& predecessors,
      WF&& weight = {},
      [[maybe_unused]] const Alloc& alloc = {}
  ) -> void;
  ```
- **Single-Source Convenience Overload:**
  - Overload accepting a single `graph::vertex_id_t<G> source` instead of a `Sources` range, delegating to the primary overload via a single-element `std::array<graph::vertex_id_t<G>, 1>{source}`.
- **Default Weight Handling:**
  - If `WF` is omitted or default-constructed without a target, or if a caller passes a custom callable `weight(g, uv)` (such as `amet`'s `[](const auto& gr, const auto& uv) -> double { return 1.0; }`), invoke `std::invoke(weight, g, uv)` to compute edge weights. Also support unary weight callables `weight(uv)` via `if constexpr (std::invocable<WF, const G&, const graph::edge_t<G>&>)`.
- **Initialization & Relaxation Invariants:**
  - Use `using Weight = std::decay_t<decltype(std::invoke(weight, g, std::declval<graph::edge_t<G>>()))>;` (or `typename std::ranges::range_value_t<Distances>`).
  - Initialize `distances[v]` to `std::numeric_limits<Weight>::max()` (or `std::numeric_limits<Weight>::infinity()` when floating-point) for all `v`, and `predecessors[v] = v` for all `v`.
  - For every `s` in `sources`, set `distances[s] = Weight{0}` and push `{Weight{0}, s}` into a `std::priority_queue` min-heap (`std::greater<>`).
  - Relax outgoing edges `uv` from `u` to `v = graph::target_id(g, uv)` whenever `distances[u] + w < distances[v]`, updating `distances[v]` and `predecessors[v] = u`.
  - Declare the min-heap explicitly as:
  using Entry = std::pair<Weight, graph::vertex_id_t<G>>;
  std::priority_queue<Entry, std::vector<Entry>, std::greater<Entry>> pq;
  - Read and write distances and predecessors via helper lambdas dist_of(u) and pred_of(u) that check if constexpr (std::invocable<Distances&, const G&, graph::vertex_id_t<G>>) { return distances(g, u); } else { return distances[static_cast<std::size_t>(u)]; } (do not call .resize() on distances or predecessors when std::invocable is true).

### 2. Iterative Depth-First Search (`depth_first_search` — For `cnpm`, `winnow`, `*-thresher`)
Implement a zero-recursion, stack-allocated iterative DFS traversal inside `namespace graph` (and alias into `namespace graph::algorithm`):
- **Visitor Hooks (Optional via `if constexpr (requires { ... })`):**
  - Supports lightweight visitor objects with any subset of `discover_vertex(uid)`, `examine_edge(e)`, `tree_edge(e)`, `back_edge(e)`, and `finish_vertex(uid)`.
- **Signature:**
  - `template <typename G, typename Visitor> auto depth_first_search(G&& g, graph::vertex_id_t<G> start, Visitor&& vis) -> void` (plus an all-components overload `depth_first_search(G&& g, Visitor&& vis) -> void` that sweeps unvisited vertices `0 .. N-1`).
  - Uses explicit `std::vector<std::uint8_t> state` (`0 = unvisited`, `1 = visiting`, `2 = visited`) and an explicit `std::vector` stack to prevent stack overflow on deep AST/parse graphs in `antlr4-thresher` and `json-thresher`.
  - Declare std::vector<std::uint8_t> state(graph::num_vertices(g), 0); and std::stack<graph::vertex_id_t<G>> st; with explicit std:: prefixes.
- **Stop Immediately After depth_first_search:** Immediately after closing namespace graph::algorithm following depth_first_search, emit #endif and close the ``` code fence. Do not re-emit vertex_info, neighbor_info, edge_info, std::tuple_size, or any view definitions.

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the complete, self-contained C++26 header enclosed within header guard defines (`#ifndef STRATUM_GRAPH_ALGORITHMS_HPP` / `#define STRATUM_GRAPH_ALGORITHMS_HPP` / `#endif`). Do not include conversational remarks before or after the block.
