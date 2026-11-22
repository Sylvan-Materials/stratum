# Stratum Module Synthesis Contract: Core Graph CPOs, Concepts & Descriptors

You are the Stratum Automation Code Generator. Translate this specification into a single, production-grade, header-only C++26 component (`graph/graph.hpp`) adhering strictly to Stratum's design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- **Standard Headers:** Include `<concepts>`, `<cstddef>`, `<cstdint>`, `<functional>`, `<iterator>`, `<ranges>`, `<tuple>`, `<type_traits>`, `<utility>`, and `<vector>` explicitly at the top of the file.
- **Explicit Types & Trailing Returns:** Declare concrete variable types explicitly inside function bodies. Reserve `auto` strictly for trailing return placeholders (`-> decltype(auto)` or `-> void`) and concept-constrained return types (`-> std::integral auto`). Never write `const auto` before a function name, and never place `auto` inside template argument lists or `requires(...)` parameter lists.
- **Separate Non-Template Niebloid Structs:** Declare every CPO functor (`vertices_fn`, `edges_fn`, `vertex_id_fn`, `target_id_fn`, `source_id_fn`, `vertex_value_fn`, `edge_value_fn`, `degree_fn`, `num_vertices_fn`, `num_edges_fn`, `find_vertex_fn`) as a plain non-template `struct`, followed immediately by its `inline constexpr` Niebloid instance variable.
- **Constrained Overload Sets (Exact C++26 Form):**
  Every CPO overload inside a functor struct must follow this exact syntax (never omit `auto operator()(...) const -> decltype(auto)` or the second `requires` keyword):
  ```cpp
  template <typename G>
  auto operator()(G&& g) const -> decltype(auto)
      requires requires(G&& g) { tag_invoke(std::declval<vertices_fn>(), std::forward<G>(g)); }
  {
      return tag_invoke(*this, std::forward<G>(g));
  }
  ```
  - Inside each CPO functor struct, implement each customization tier as a separate templated `operator()` overload carrying a parameterized `requires requires(...)` clause on its signature.
  - Inside signature `requires` clauses, use `std::declval<FunctorType>()` as the tag argument to `tag_invoke` (never `*this` or `FunctorType{}` inside the incomplete struct body). Inside the evaluated function body, invoke `tag_invoke(*this, ...)` or the member access/method directly (never call `std::declval` inside an evaluated body, and never recursively call the outer Niebloid instance).
  - Negate higher-priority `tag_invoke` constraints on fallback overloads to prevent overload ambiguity.
- **Concept Syntax:** Every `concept` declaration must begin with an explicit `template <...>` parameter header. Inside `requires { ... }` blocks, use `{ expr } -> ConceptName;` for value expressions and `requires ConceptName<Type>;` for type constraints.
- **Consistent Parameter Names (g, u, e):** Name graph parameters g, vertex parameters u, and edge parameters e consistently in both function signatures (G&& g, const U& u) and requires(...) parameter lists, and use those exact parameter names (std::forward<G>(g), u, e) inside the function body. Never invent dummy names like gx, ux, or ex.
- **Strictly namespace graph (Never namespace std::graph):** Despite P1709 using std::graph, emit all core graph CPOs, concepts, traits, and descriptors inside top-level namespace graph (never namespace std::graph).

## 2. Target Component Archetype (`namespace gNraph` & `namespace amet`)
- **Purpose:** Foundational graph abstraction layer supporting all seven Stratum container archetypes (Standard Adjacency Lists, Static CSR, Zero-Storage Procedural Graphs, PCSR, \(k^2\)-Trees, HBA, and Flat Edge Lists) while providing exact API compatibility for `amet`.
- **Root Namespaces:** Define all core graph CPOs, concepts, type aliases, and copyable ingestion descriptors directly inside `namespace graph`.

## 3. Exact Top-to-Bottom Declaration Sequence (Mandatory)

### Part A: Inside `namespace graph`
1. **ADL Poison-Pill:** Immediately after opening `namespace graph`, declare a zero-parameter `void tag_invoke();` function.
2. **Primary Graph CPOs (Functors + `inline constexpr` Instances):**
   - **`vertices` (`vertices_fn`):** Takes `(G&& g)`. Overload A checks `requires(G&& g) { tag_invoke(std::declval<vertices_fn>(), std::forward<G>(g)); }` and returns `tag_invoke(*this, std::forward<G>(g))`; Overload B checks `requires(G&& g) { std::forward<G>(g).vertices(); }` and returns `std::forward<G>(g).vertices()`; Overload C checks `std::ranges::forward_range<G>` directly (so minimal structural ranges like `MockGraph` with `begin()`/`end()` work out of the box) and returns `std::forward<G>(g)`.
   - **`edges` (`edges_fn`):** Takes (G&& g, const U& u). Emit ONLY two overloads: Overload A (tag_invoke(*this, std::forward<G>(g), u)) and Overload B (std::forward<G>(g).edges(u)). Do not emit a third overload.
   - **`target_id` (`target_id_fn`):**
     - Binary Overload A taking `(G&& g, const E& e)`: checks `tag_invoke(std::declval<target_id_fn>(), std::forward<G>(g), e)` and returns `tag_invoke(*this, std::forward<G>(g), e)`.
     - Binary Overload B taking `(G&& g, const E& e)`: checks `std::forward<G>(g).target_id(e)` (when Overload A does not match) and returns `std::forward<G>(g).target_id(e)`.
     - Binary Overload C taking `(G&& g, const E& e)`: when neither `tag_invoke` nor `std::forward<G>(g).target_id(e)` matches, delegates directly to `(*this)(e)` so view descriptors like `neighbor_info` and `edge_info` work seamlessly.
     - Unary overloads taking `(const E& e)`: checks `tag_invoke(std::declval<target_id_fn>(), e)`, `e.target_id()`, or `e.target_id`.
   - **`find_vertex` (`find_vertex_fn`):** Takes `(G&& g, const VId& uid)`. Overload A checks `tag_invoke(std::declval<find_vertex_fn>(), std::forward<G>(g), uid)`; Overload B checks `std::forward<G>(g).find_vertex(uid)`; Overload C checks `requires(G&& g) { graph::vertices(std::forward<G>(g)); }` and returns `std::ranges::next(std::ranges::begin(graph::vertices(std::forward<G>(g))), static_cast<std::ptrdiff_t>(uid), std::ranges::end(graph::vertices(std::forward<G>(g))))`.

   - **`target_id` (`target_id_fn`):**
     - Binary Overload A taking `(G&& g, const E& e)`: checks `tag_invoke(std::declval<target_id_fn>(), std::forward<G>(g), e)` and returns `tag_invoke(*this, std::forward<G>(g), e)`.
     - Binary Overload B taking `(G&& g, const E& e)`: checks `std::forward<G>(g).target_id(e)` (when Overload A does not match) and returns `std::forward<G>(g).target_id(e)`.
     - Binary Overload C taking `(G&& g, const E& e)`: when neither `tag_invoke` nor `std::forward<G>(g).target_id(e)` matches, delegates directly to `(*this)(e)` so view descriptors like `neighbor_info` and `edge_info` work seamlessly.
     - Unary overloads taking `(const E& e)`: checks `tag_invoke(std::declval<target_id_fn>(), e)`, `e.target_id()`, or `e.target_id`.   - **`source_id` (`source_id_fn`):**
     - Binary overload taking `(G&& g, const E& e)`: checks `tag_invoke(std::declval<source_id_fn>(), std::forward<G>(g), e)` or `std::forward<G>(g).source_id(e)`.
     - Unary overload taking `(const E& e)`: checks `tag_invoke(std::declval<source_id_fn>(), e)`, `e.source_id()`, or `e.source_id`.
   - **`vertex_value` (`vertex_value_fn`):** Takes `(G&& g, const U& u)`. Checks `tag_invoke(std::declval<vertex_value_fn>(), std::forward<G>(g), u)` or `std::forward<G>(g).vertex_value(u)`.
   - **`edge_value` (`edge_value_fn`):** Takes `(G&& g, const E& e)`. Checks `tag_invoke(std::declval<edge_value_fn>(), std::forward<G>(g), e)` or `std::forward<G>(g).edge_value(e)`.
   - **1degree (degree_fn):** Takes (G&& g, const U& u). Overload A checks tag_invoke(std::declval<degree_fn>(), std::forward<G>(g), u); Overload B checks std::forward<G>(g).degree(u); Overload C checks requires(G&& g, const U& u) { graph::edges(std::forward<G>(g), u); } and returns static_cast<std::size_t>(std::ranges::distance(graph::edges(std::forward<G>(g), u))).
   - **num_vertices (num_vertices_fn):** Takes (G&& g). Overload A checks tag_invoke(std::declval<num_vertices_fn>(), std::forward<G>(g)); Overload B checks std::forward<G>(g).num_vertices(); Overload C checks requires(G&& g) { graph::vertices(std::forward<G>(g)); } and returns static_cast<std::size_t>(std::ranges::distance(graph::vertices(std::forward<G>(g)))).
   - **`num_edges` (`num_edges_fn`):** Takes `(G&& g)`. Overload A checks `tag_invoke(std::declval<num_edges_fn>(), std::forward<G>(g))`; Overload B checks `std::forward<G>(g).num_edges()`; Overload C checks `requires(G&& g) { graph::vertices(std::forward<G>(g)); }` and sums edges using a plain range-based `for` loop (`std::size_t total = 0; for (const auto& u : graph::vertices(g)) { total += static_cast<std::size_t>(std::ranges::distance(graph::edges(g, u))); } return total;` — never call `std::accumulate`).
   - **No Forward References in CPO Constraints:** Inside Item 2 (vertices_fn through num_edges_fn), never reference vertex_t, edge_t, vertex_range_t, or edge_range_t (which are declared later in Item 3), and never emit a third fallback overload on edges_fn that calls graph::edges recursively. Only emit the exact overloads listed below (tag_invoke and member call .edges(u) / .vertex_id(u) / .target_id(e) / .source_id(e)).

3. **Graph Associated Type Traits:**
   - `template <typename G> using vertex_range_t = decltype(graph::vertices(std::declval<G&>()));`
   - `template <typename G> using vertex_iterator_t = std::ranges::iterator_t<vertex_range_t<G>>;`
   - `template <typename G> using vertex_t = std::ranges::range_value_t<vertex_range_t<G>>;`
   - `template <typename G> using vertex_id_t = decltype(graph::vertex_id(std::declval<G&>(), std::declval<vertex_t<G>>()));` (falling back to unary `graph::vertex_id(std::declval<vertex_t<G>>())` if binary is not defined—implement via a helper variable/type trait or constrained overload on `vertex_id_fn`).
   - `template <typename G> using edge_range_t = decltype(graph::edges(std::declval<G&>(), std::declval<vertex_t<G>>()));`
   - `template <typename G> using edge_t = std::ranges::range_value_t<edge_range_t<G>>;`
4. **Core Graph Concepts:**
   - **`vertex_list_graph`:** `template <typename G> concept vertex_list_graph = requires(G& g) { { graph::vertices(g) } -> std::ranges::forward_range; };`
   - **`incidence_graph`:** `template <typename G> concept incidence_graph = vertex_list_graph<G> && requires(G& g, vertex_t<G> u) { { graph::edges(g, u) } -> std::ranges::forward_range; };`
   - **adjacency_list:** template <typename G> concept adjacency_list = incidence_graph<G>;
   - **`viewable_incidence_graph`:** `template <typename G> concept viewable_incidence_graph = vertex_list_graph<G>;` (allowing both full incidence graphs and minimal structural vertex-range fixtures like `MockGraph` in `amet` to satisfy viewability cleanly).
5. **Copyable Ingestion Descriptors (Top-Level Struct Templates inside `namespace graph`, Never Nested):**
```cpp
template <typename VId = std::size_t, typename VV = void>
struct copyable_vertex_t {
    VId id{};
    VV value{};
};
  1.Omit value completely in the <VId, void> partial specializations (never write void value{};).

template <typename VId>
struct copyable_vertex_t<VId, void> {
    VId id{};
};

template <typename VId = std::size_t, typename EV = void>
struct copyable_edge_t {
    VId source_id{};
    VId target_id{};
    EV value{};
};

template <typename VId>
struct copyable_edge_t<VId, void> {
    VId source_id{};
    VId target_id{};
};
```
   - **`copyable_edge_t`:** `template <typename VId = std::size_t, typename EV = void> struct copyable_edge_t` with members `VId source_id{}; VId target_id{}; EV value{};` (and a partial specialization `template <typename VId> struct copyable_edge_t<VId, void>` with only `VId source_id{}; VId target_id{};`).
   - **Sub-namespace namespace adj_list (Inside namespace graph):** ```cpp
   namespace adj_list {
using graph::adjacency_list;
using graph::incidence_graph;
using graph::vertex_list_graph;
using graph::vertex_t;
using graph::edge_t;
using graph::vertex_id_t;
using graph::vertices;
using graph::edges;
using graph::vertex_id;
using graph::target_id;
using graph::source_id;
using graph::degree;
using graph::num_vertices;
using graph::num_edges;
using graph::find_vertex;
} // namespace adj_list
```

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the complete, self-contained C++26 header enclosed within header guard defines (`#ifndef STRATUM_GRAPH_GRAPH_HPP` / `#define STRATUM_GRAPH_GRAPH_HPP` / `#endif`). Do not include conversational remarks before or after the block.
