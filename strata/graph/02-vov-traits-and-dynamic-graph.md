# Stratum Module Synthesis Contract: Vector-of-Vectors Traits & Dynamic Adjacency Graph

You are the Stratum Automation Code Generator. Translate this specification into a single, production-grade, header-only C++26 component (`graph/container/dynamic_adjacency_graph.hpp`) adhering strictly to Stratum's design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- **Standard Headers & Core Include:** Include `"graph/graph.hpp"`, `<algorithm>`, `<concepts>`, `<cstddef>`, `<cstdint>`, `<functional>`, `<iterator>`, `<ranges>`, `<stdexcept>`, `<tuple>`, `<type_traits>`, `<utility>`, and `<vector>` explicitly at the top of the file.
- **No degree Member Function:** Do not declare any degree(...) member function on dynamic_adjacency_graph or dynamic_vertex (graph::degree in "graph/graph.hpp" already computes out-degree automatically via graph::edges(g, u)).
- **Explicit Types & Trailing Returns:** Declare concrete variable types explicitly inside function bodies. Reserve `auto` strictly for trailing return placeholders (`-> decltype(auto)` or `-> void`) and concept-constrained return types. Always write bare, unqualified `auto` on the left side of any function with a `->` trailing return type (never `auto&` or `const auto`). Never place `auto` inside template argument lists or `requires(...)` parameter lists.
- **Class Template Constraint Placement:** When constraining a class template, place the concept directly inside `template <...>` brackets or in a `requires` clause *before* the `class` keyword—never between the class name and its opening brace `{`. Terminate every `struct` and `class` definition with `;`.
- **Safe Reference Fallbacks:** Never return `{}` from a function returning a `const` reference; use a local `static const` empty container instance or bounds-checked storage.

## 2. Target Component Archetype (`namespace graph::container`)
- **Archetype:** Standard Adjacency List container (`dynamic_adjacency_graph`) parameterized by `vov_graph_traits`, matching `stdgraph/P1709` and `amet`'s exact container interface.
- **Directed & Undirected Modes:** Must support both strictly directed graphs (`Undirected == false`) and undirected / bidirectional graphs (`Undirected == true`), as well as `void` and non-`void` vertex/edge/graph value types.

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
Define these two struct templates at namespace scope immediately after `vov_graph_traits` (above `dynamic_adjacency_graph`). Suffix data member names with a trailing underscore (`_`) so they never collide with accessor member functions, and always use bare `auto` on the left of every `->` trailing return type (e.g., `Declare mutable and const accessors using plain auto without ref-qualifiers before noexcept: auto value() noexcept -> EV& and auto value() const noexcept -> const EV& (never auto&).`):
- **`template <typename VId, typename EV = void> struct dynamic_edge`:**
  - Primary template stores `VId source_id_{}; VId target_id_{}; EV value_{};` and provides member accessors `source_id()`, `target_id()`, and mutable/`const` `value()`.
  - Partial specialization `template <typename VId> struct dynamic_edge<VId, void>` stores only `VId source_id_{}; VId target_id_{};` and provides `source_id()` and `target_id()`.
- **`template <typename VId, typename VV = void, typename EV = void> struct dynamic_vertex`:**
  - Primary template stores `VId id_{}; VV value_{}; std::vector<dynamic_edge<VId, EV>> edges_{};` and provides member accessors `id()`, mutable/`const` `value()`, and mutable/`const` `edges()`.
  - Partial specialization `template <typename VId, typename EV> struct dynamic_vertex<VId, void, EV>` stores only `VId id_{}; std::vector<dynamic_edge<VId, EV>> edges_{};` and provides `id()` and mutable/`const` `edges()`.
  - Keep `dynamic_edge` and `dynamic_vertex` as plain C++26 aggregate structs (do **not** declare custom constructors or variadic `Args...` constructors; aggregate brace initialization `{...}` works automatically).
- **Default Constructible (Required by `std::vector::resize`):** Keep `dynamic_edge` and `dynamic_vertex` as plain C++26 aggregate structs with **zero user-declared constructors** (do **not** declare `dynamic_vertex(VId)` or any other constructor; if any constructor is emitted, you MUST also declare `dynamic_vertex() = default;`).

### 3. `dynamic_adjacency_graph` Class Template
Define:
```cpp
template <typename Traits = vov_graph_traits<>>
class dynamic_adjacency_graph;
```
Inside `dynamic_adjacency_graph`, refer to member types strictly via `vertex_id_type`, `vertex_value_type`, `edge_value_type`, and `graph_value_type` (never `VId`, `VV`, or `EV`). Implement these exact public types, storage, and member functions:
- **Public Type Aliases:**
  - `using traits_type = Traits;`
  - `using vertex_id_type = typename Traits::vertex_id_type;`
  - `using vertex_value_type = typename Traits::vertex_value_type;`
  - `using edge_value_type = typename Traits::edge_value_type;`
  - `using graph_value_type = typename Traits::graph_value_type;`
  - `using vertex_type = dynamic_vertex<vertex_id_type, vertex_value_type, edge_value_type>;`
  - `using edge_type = dynamic_edge<vertex_id_type, edge_value_type>;`
- **Primary Storage:**
  - `std::vector<vertex_type> vertices_{};` indexed directly by `std::size_t idx = static_cast<std::size_t>(id);`, resizing `vertices_` whenever `idx >= vertices_.size()`.
  - **Safe Container Storage & Initialization (ASan-Clean):**
    - Inside `dynamic_vertex` and `dynamic_edge`, always value-initialize payload members (`VV value{};` and `EV value{};` — never leave `VV value;` uninitialized).
    - Inside `load_vertices` and `load_edges`, manage `vertices_` exclusively via standard `std::vector` operations (`vertices_.resize(new_size);` followed by member assignment `vertices_[id].id = id; vertices_[id].value = ...;`) — never use raw `operator new`, `std::malloc`, or unconstructed `std::allocator::allocate`.
- **ADL Hidden Friends (Strictly Inside `public:` Body of `class dynamic_adjacency_graph` Before `};`):**
  - Declare both hidden `friend` functions inside the `public:` section of `class dynamic_adjacency_graph` **before** its closing `};` (never after `};`):
    - `friend constexpr auto num_vertices(const dynamic_adjacency_graph& g) noexcept -> std::size_t { return g.vertices_.size(); }`
    - `friend constexpr auto num_edges(const dynamic_adjacency_graph& g) noexcept -> std::size_t` (summing `v.edges_.size()` across `g.vertices_`).
- **Capacity & Bulk Ingestion Methods (Matching `amet/test/src/test.cpp`):**
  1. `auto reserve_vertices(std::size_t count) -> void` — calls `vertices_.reserve(count);`.
  2. `auto reserve_edges([[maybe_unused]] std::size_t count) -> void` — no-op using `[[maybe_unused]]`.
  3. **`load_vertices(VRng&& vrng, VProj vproj = {}) -> void`:**
    - Iterates over every element `elem` in `vrng`, invokes `auto projected = std::invoke(vproj, elem);`, and declares `vertex_id_type id{};` before the `if constexpr` chain so it remains in scope after the branches close.
    - **Step 1 (Extract `id` ONLY — do NOT index `vertices_[id]` inside this `if constexpr` chain):**
      - `if constexpr (requires { projected.id; })` — sets `id = static_cast<vertex_id_type>(projected.id);`.
      - `else if constexpr (requires { std::get<0>(projected); })` — sets `id = static_cast<vertex_id_type>(std::get<0>(projected));`.
      - `else` — sets `id = static_cast<vertex_id_type>(projected);`.
    - **Step 2 (Resize `vertices_` FIRST):**
      - Compute `std::size_t idx = static_cast<std::size_t>(id);` and execute `if (idx >= vertices_.size()) { vertices_.resize(idx + 1); }` (comparing against `vertices_.size()`, never `vertices_.capacity()`) **before** accessing `vertices_[idx]`.
    - **Step 3 (Populate `vertices_[idx]` AFTER resize):**
      - Assign `vertices_[idx].id_ = id;`.
      - `if constexpr (!std::is_void_v<vertex_value_type>)` — if `requires { projected.value; }`, assign `vertices_[idx].value_ = std::move(projected.value);`; else if `requires { std::get<1>(projected); }`, assign `vertices_[idx].value_ = std::move(std::get<1>(projected));`.
- **Const-Correct `edges` Overloads:** Never write `auto edges(const vertex_type& u) -> std::vector<edge_type>&` (returning a non-const reference from a `const vertex_type&` parameter discards `const` qualifiers). Any `edges` overload accepting `const vertex_type&` must return `const std::vector<edge_type>&` (or `decltype(auto)`).
  
  4. **`load_edges(ERng&& erng, EProj eproj = {}) -> void`:**
     - Iterates over every element `elem` in `erng` and invokes `auto projected = std::invoke(eproj, elem);`.
     - Detect the shape of `projected` strictly using structural `if constexpr (requires { ... })` checks:
       - `if constexpr (requires { projected.source_id; projected.target_id; })` — extracts `src` and `dst` (plus `projected.value` when `!std::is_void_v<edge_value_type>` and `requires { projected.value; }`).
       - `else if constexpr (requires { std::get<0>(projected); std::get<1>(projected); })` — extracts `src` and `dst` via `std::get<0>` and `std::get<1>` (plus `std::get<2>` when `!std::is_void_v<edge_value_type>`).
     - Resizes `vertices_` to hold `std::max(src_idx, dst_idx) + 1`, sets `vertices_[src_idx].id_ = src` and `vertices_[dst_idx].id_ = dst`, and pushes the edge onto `vertices_[src_idx].edges_`.
     - Compute `std::size_t src_idx = static_cast<std::size_t>(src);`, `std::size_t dst_idx = static_cast<std::size_t>(dst);`, and `std::size_t max_idx = std::max(src_idx, dst_idx);`. If `max_idx >= vertices_.size()`, call `vertices_.resize(max_idx + 1);` **before** indexing `vertices_[src_idx]` or `vertices_[dst_idx]` (never reference `idx` inside `load_edges`).
     - **Directedness:** Always insert the projected edge strictly once (`src -> dst`) without duplicating reverse edges.
- **CPO Customization Hooks (Member Functions Inside `dynamic_adjacency_graph`):**
  - `auto vertices() -> std::vector<vertex_type>&` and `auto vertices() const -> const std::vector<vertex_type>&`
  - `auto edges(vertex_type& u) -> std::vector<edge_type>&` and `auto edges(const vertex_type& u) const -> const std::vector<edge_type>&`
  - `auto edges(vertex_id_type uid) -> std::vector<edge_type>&` and `auto edges(vertex_id_type uid) const -> const std::vector<edge_type>&`
  - `auto vertex_id(const vertex_type& u) const noexcept -> vertex_id_type` (returns `u.id()`)
  - `auto target_id(const edge_type& e) const noexcept -> vertex_id_type` (returns `e.target_id()`)
  - `auto source_id(const edge_type& e) const noexcept -> vertex_id_type` (returns `e.source_id()`)
  - `auto vertex_value(vertex_type& u) -> vertex_value_type&` and `const` overload (enabled via `requires (!std::is_void_v<vertex_value_type>)`)
  - `auto edge_value(edge_type& e) -> edge_value_type& and auto edge_value(const edge_type& e) const -> const edge_value_type& returning e.value() (enabled via requires (!std::is_void_v<edge_value_type>);` do not generate any other edge_value overloads).
  - `auto size() const noexcept -> std::size_t` (returns `vertices_.size()`)
  - `auto operator[](vertex_id_type uid) -> vertex_type&` and `const` overload.
### Container & CPO Edge Traversal Rules
  1. **Const-Correct Accessors (C++23/2c Compatible):**
    - Never return a non-const `std::vector<edge_type>&` from an accessor taking `const vertex_type&`.
    - Always generate paired `const` and non-`const` overloads—or use C++23 deducing `this` (`decltype(auto) edges(this auto&& self, auto&& u) { return std::forward_like<decltype(u)>(u.edges()); }`)—so `const` vertex descriptors return `const` edge ranges.
  2. **Neighbor Proxy Unpacking (`graph::neighbor_info`):**
    - Range-based `for` loops over `graph::edges(g, u)` yield `graph::neighbor_info` proxies rather than raw edge descriptors.
    - Always unpack neighbor vertices using C++23 structured bindings (`for (auto&& [neighbor_id, edge] : graph::edges(g, u))`) or read `.target_id` directly; never pass a `neighbor_info` proxy to `graph::target_id(g, edge_ref)`.

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the complete, self-contained C++26 header enclosed within header guard defines (`#ifndef STRATUM_GRAPH_DYNAMIC_ADJACENCY_GRAPH_HPP` / `#define STRATUM_GRAPH_DYNAMIC_ADJACENCY_GRAPH_HPP` / `#endif`). Do not include conversational remarks before or after the block.
