# Stratum Module Synthesis Contract: Vector-of-Vectors Traits & Dynamic Adjacency Graph

You are the Stratum Automation Code Generator. Translate this specification into a single, production-grade, header-only C++26 component (`graph/container/dynamic_adjacency_graph.hpp`) adhering strictly to Stratum's design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- **Standard Headers & Core Include:** Include `"graph/graph.hpp"`, `<algorithm>`, `<concepts>`, `<cstddef>`, `<cstdint>`, `<functional>`, `<iterator>`, `<ranges>`, `<stdexcept>`, `<tuple>`, `<type_traits>`, `<utility>`, and `<vector>` explicitly at the top of the file.
- **Explicit Types & Trailing Returns:** Declare concrete variable types explicitly inside function bodies. Reserve `auto` strictly for trailing return placeholders (`-> decltype(auto)` or `-> void`) and concept-constrained return types. Never write `const auto` before a function name, and never place `auto` inside template argument lists or `requires(...)` parameter lists.
- **Class Template Constraint Placement:** When constraining a class template, place the concept directly inside `template <...>` brackets or in a `requires` clause *before* the `class` keyword—never between the class name and its opening brace `{`. Terminate every `struct` and `class` definition with `;`.
- **Safe Reference Fallbacks:** Never return `{}` from a function returning a `const` reference; use a local `static const` empty container instance or bounds-checked storage.
- **Bare auto for Trailing Returns:** Always write bare, unqualified auto on the left side of any function with a -> trailing return type (never auto& or const auto).

## 2. Target Component Archetype (`namespace graph::container`)
- **Archetype:** Standard Adjacency List container (`dynamic_adjacency_graph`) parameterized by `vov_graph_traits`, matching `stdgraph/P1709` and `amet`'s exact container interface.
- **Directed & Undirected Modes:** Must support both strictly directed graphs (`Sourced == false` / `Undirected == false`) and undirected / bidirectional graphs (`Undirected == true`), as well as `void` and non-`void` vertex/edge/graph value types.

## 3. Exact Top-to-Bottom Declaration Sequence inside `namespace graph::container` (Mandatory)

### 1. `vov_graph_traits` Struct Template
Define:
```cpp
template <
    typename EV = void,
    typename VV = void,
    typename GV = void,
    std::integral VId = std::size_t,
    bool Undirected = false
>
struct vov_graph_traits;
```
Inside `vov_graph_traits`, expose these exact public type aliases:
- `using edge_value_type = EV;`
- `using vertex_value_type = VV;`
- `using graph_value_type = GV;`
- `using vertex_id_type = VId;`
- `static constexpr bool is_undirected = Undirected;`

### 2. Internal Edge & Vertex Descriptor Types
Define these two struct templates at namespace scope immediately after `vov_graph_traits` (above `dynamic_adjacency_graph`). Always suffix data member names with a trailing underscore (`_`) so they never collide with accessor member function names:
- **`template <typename VId, typename EV = void> struct dynamic_edge`:**
  - Primary template stores `VId source_id_{}; VId target_id_{}; EV value_{};` and provides member accessors `source_id()`, `target_id()`, and mutable/`const` `value()`.
  - Partial specialization `template <typename VId> struct dynamic_edge<VId, void>` stores only `VId source_id_{}; VId target_id_{};` and provides `source_id()` and `target_id()`.
- **`template <typename VId, typename VV = void, typename EV = void> struct dynamic_vertex`:**
  - Primary template stores `VId id_{}; VV value_{}; std::vector<dynamic_edge<VId, EV>> edges_{};` and provides member accessors `id()`, mutable/`const` `value()`, and mutable/`const` `edges()`.
  - Partial specialization `template <typename VId, typename EV> struct dynamic_vertex<VId, void, EV>` stores only `VId id_{}; std::vector<dynamic_edge<VId, EV>> edges_{};` and provides `id()` and mutable/`const` `edges()`.

### 3. `dynamic_adjacency_graph` Class Template
Define:
```cpp
template <typename Traits = vov_graph_traits<>>
class dynamic_adjacency_graph;
```
Inside dynamic_adjacency_graph, always refer to the vertex ID type via the class's public alias vertex_id_type (never VId).
Inside `dynamic_adjacency_graph`, implement these exact public types, storage, and member functions:
- **Public Type Aliases:**
  - `using traits_type = Traits;`
  - `using vertex_id_type = typename Traits::vertex_id_type;`
  - `using vertex_value_type = typename Traits::vertex_value_type;`
  - `using edge_value_type = typename Traits::edge_value_type;`
  - `using graph_value_type = typename Traits::graph_value_type;`
  - `using vertex_type = dynamic_vertex<vertex_id_type, vertex_value_type, edge_value_type>;`
  - `using edge_type = dynamic_edge<vertex_id_type, edge_value_type>;`
- **Primary Storage:**
  - `std::vector<vertex_type> vertices_{};` indexed directly by `vertex_id_type` (`0` through `N - 1`), automatically resizing if an inserted vertex ID or edge endpoint exceeds `vertices_.size()`.
- **Capacity & Bulk Ingestion Methods (Matching `amet/test/src/test.cpp`):**
  1. `auto reserve_vertices(std::size_t count) -> void` — reserves capacity on `vertices_`.
  2. auto reserve_edges([[maybe_unused]] std::size_t count) -> void — no-op (use [[maybe_unused]] to satisfy -Wall -Wextra). When indexing vertices_ by a std::size_t vertex ID (std::size_t idx = static_cast<std::size_t>(id);), assign vertices_[idx].id_ = id; (always store the index in an explicit std::size_t idx variable before indexing vertices_[idx] so bracket expressions never nest).
  3. **`load_vertices(VRng&& vrng, VProj vproj = {}) -> void`:**
     - Iterates over every element `elem` in `vrng` and invokes `auto projected = std::invoke(vproj, elem);`.
     - Must accept **both** `graph::copyable_vertex_t<VId, VV>` (which has `.id` and `.value` members) **and** tuple-like / pair-like projected results (such as `std::make_pair(v.id, v.value)` or `std::tuple<VId, VV>` via `std::tuple_size` / `std::get<0>` and `std::get<1>`), as well as bare `VId` values when `VV` is `void`.
     - Resizes `vertices_` as needed so `vertices_[id]` exists and assigns `vertices_[id].id = id` and `vertices_[id].value = value` (when `!std::is_void_v<VV>`).
  4. **`load_edges(ERng&& erng, EProj eproj = {}) -> void`:** 
     - Iterates over every element `elem` in `erng` and invokes `auto projected = std::invoke(eproj, elem);`.
     - Must accept **both** `graph::copyable_edge_t<VId, EV>` **and** any structural edge record possessing `.source_id`, `.target_id`, and optional `.value` members (such as `amet`'s `abstract_edge_record`), as well as 2-element or 3-element tuple-like records (`std::get<0>`, `std::get<1>`, `std::get<2>`).
     - Ensures `vertices_` is sized to hold `std::max(src, dst) + 1`, populates `vertices_[src].id = src` and `vertices_[dst].id = dst`, and pushes the edge onto `vertices_[src].edges`.
     - **Important Note on Directedness:** In `amet`, callers already push both `(u, v)` and `(v, u)` explicitly when building undirected graphs even when `Undirected == true`; therefore, `load_edges` inserts the projected edge strictly as given (`src -> dst`) without duplicating edges a second time.
  5. **When indexing vertices_** by a std::size_t vertex ID (std::size_t idx = static_cast<std::size_t>(id);), 
     assign vertices_[idx].id_ = id; (always store the index in an explicit std::size_t idx variable before indexing vertices_[idx] so bracket expressions never nest)
- **CPO Customization Hooks (Member Functions Inside `dynamic_adjacency_graph`):**
  - `auto vertices() -> std::vector<vertex_type>&` and `auto vertices() const -> const std::vector<vertex_type>&`
  - `auto edges(const vertex_type& u) -> std::vector<edge_type>&` and `auto edges(const vertex_type& u) const -> const std::vector<edge_type>&`
  - `auto edges(vertex_id_type uid) -> std::vector<edge_type>&` and `auto edges(vertex_id_type uid) const -> const std::vector<edge_type>&`
  - `auto vertex_id(const vertex_type& u) const noexcept -> vertex_id_type` (returns `u.id`)
  - `auto target_id(const edge_type& e) const noexcept -> vertex_id_type` (returns `e.target_id`)
  - `auto source_id(const edge_type& e) const noexcept -> vertex_id_type` (returns `e.source_id`)
  - `auto vertex_value(vertex_type& u) -> vertex_value_type&` and `const` overload (enabled via `requires (!std::is_void_v<vertex_value_type>)`)
  - `auto edge_value(edge_type& e) -> edge_value_type&` and `const` overload (enabled via `requires (!std::is_void_v<edge_value_type>)`)
  - `auto size() const noexcept -> std::size_t` (returns `vertices_.size()`)
  - `auto operator[](vertex_id_type uid) -> vertex_type&` and `const` overload.
  6. **Detect projected record shapes using structural if constexpr (requires { ... }) checks rather than std::is_same_v<..., graph::copyable_edge_t<...>>:**
	 - In load_vertices: check if constexpr (requires { projected.id; }) first, then else if constexpr (requires { std::get<0>(projected); }), then else (for bare vertex_id_type).
	 - In load_edges: check if constexpr (requires { projected.source_id; projected.target_id; }) first (checking requires { projected.value; } inside for non-void edge values), then else if constexpr (requires { std::get<0>(projected); std::get<1>(projected); }).

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the complete, self-contained C++26 header enclosed within header guard defines (`#ifndef STRATUM_GRAPH_DYNAMIC_ADJACENCY_GRAPH_HPP` / `#define STRATUM_GRAPH_DYNAMIC_ADJACENCY_GRAPH_HPP` / `#endif`). Do not include conversational remarks before or after the block.
