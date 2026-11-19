# Stratum Module Synthesis Contract: Graph Views & Filtered Adaptors

You are the Stratum Automation Code Generator. Translate this specification into a single, production-grade, header-only C++26 component (`graph/views/graph_views.hpp`) adhering strictly to Stratum's design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- **Standard Headers & Core Includes:** Include `"graph/graph.hpp"`, `"graph/container/dynamic_adjacency_graph.hpp"`, `<concepts>`, `<cstddef>`, `<functional>`, `<iterator>`, `<ranges>`, `<tuple>`, `<type_traits>`, and `<utility>` explicitly at the top of the file.
- **Explicit Types & Trailing Returns:** Declare concrete variable types explicitly inside function bodies. Reserve `auto` strictly for trailing return placeholders (`-> decltype(auto)` or `-> auto`) and concept-constrained return types. Never write `const auto` before a function name, and never place `auto` inside template argument lists or `requires(...)` parameter lists.
- **Top-Level Struct Templates Only:** Declare `vertex_info`, `neighbor_info`, and `edge_info` directly at namespace scope inside `namespace graph` using explicit `template <...>` headers—never wrap them inside an outer non-template struct.
- **Namespace Hygiene:** Always close `namespace graph` (`} // namespace graph`) before opening `namespace std`, and close `namespace std` (`} // namespace std`) before opening `namespace graph::views`. Never open `namespace std` inside `namespace graph`.

## 2. Target Component Archetype (`namespace graph::views` & `namespace graph`)
- **Purpose:** Non-owning range views (`vertexlist`, `incidence`, `neighbors`, `edgelist`) and branch-pruning graph adaptor (`filtered_graph`) matching `stdgraph/P1709` and `amet`'s traversal requirements.

## 3. Exact Top-to-Bottom Declaration Sequence (Mandatory)

### Part A: View Descriptor Struct Templates inside `namespace graph`
Open `namespace graph {` and declare these three struct templates directly at namespace scope:
1. **`template <typename VId, typename V = void> struct vertex_info`:**
   - Primary template stores `VId id{}; V vertex{};` and defines `template <std::size_t I> constexpr auto get() const -> decltype(auto)` returning `id` when `I == 0` and `vertex` when `I == 1`.
   - Partial specialization `template <typename VId> struct vertex_info<VId, void>` stores `VId id{};` (`VId` is never `void`) and defines `template <std::size_t I> constexpr auto get() const -> decltype(auto)` returning `id` when `I == 0`.
2. **`template <typename VId, typename E = void> struct neighbor_info`:**
   - Primary template stores `VId target_id{}; E edge{};` with `get<0>()` returning `target_id` and `get<1>()` returning `edge`.
   - Partial specialization `template <typename VId> struct neighbor_info<VId, void>` stores `VId target_id{};` with `get<0>()` returning `target_id`.
3. **`template <typename VId, typename E = void> struct edge_info`:**
   - Primary template stores `VId source_id{}; VId target_id{}; E edge{};` with `get<0>()` returning `source_id`, `get<1>()` returning `target_id`, and `get<2>()` returning `edge`.
   - Partial specialization `template <typename VId> struct edge_info<VId, void>` stores `VId source_id{}; VId target_id{};` with `get<0>()` returning `source_id` and `get<1>()` returning `target_id`.
4. **Close `namespace graph` (`} // namespace graph`)** immediately after `edge_info<VId, void>;`.

### Part B: Structured Binding Traits inside `namespace std`
Open `namespace std {` at global scope (outside `namespace graph`) and specialize `tuple_size` and `tuple_element` for:
- `graph::vertex_info<VId, V>` (size 2) and `graph::vertex_info<VId, void>` (size 1)
- `graph::neighbor_info<VId, E>` (size 2) and `graph::neighbor_info<VId, void>` (size 1)
- `graph::edge_info<VId, E>` (size 3) and `graph::edge_info<VId, void>` (size 2)
Inside every `tuple_element` specialization, use `.template get<I>()` with the `.template` keyword. Close `namespace std` (`} // namespace std`) immediately after.

### Part C: Range Views inside `namespace graph::views`
Open `namespace graph::views {` and define:
1. **`vertexlist(G&& g)`** and **`vertexlist(G&& g, VVF&& vvf)`:** Yields `graph::vertex_info` elements across `graph::vertices(g)`.
2. **`incidence(G&& g, const U& u)`** and **`incidence(G&& g, graph::vertex_id_t<G> uid)`:** Yields `graph::neighbor_info` elements across `graph::edges(g, u)`.
3. **`neighbors(G&& g, const U& u)`:** Yields target vertices or target IDs adjacent to `u` across `graph::edges(g, u)`.
4. **`edgelist(G&& g)`:** Yields `graph::edge_info` elements `{source_id, target_id, uv}` across all vertices and incident edges in `g`.
Close `namespace graph::views` (`} // namespace graph::views`).

### Part D: Branch-Pruning Adaptor inside `namespace graph` & `namespace graph::adaptors`
Open `namespace graph {` and define:
1. **`template  class filtered_graph`:**
   - Wraps a pointer/reference to `G` along with vertex predicate `VPred` and edge predicate `EPred` (defaulting to always-true predicates when omitted).
   - Provides `vertices()` filtered by `VPred` and `edges(u)` filtered by `EPred` so `filtered_graph` satisfies `graph::vertex_list_graph`, `graph::incidence_graph`, and `amet::viewable_incidence_graph`.
   - Provide deduction guides and alias `using graph::filtered_graph;` inside `namespace graph::adaptors`.
Close `namespace graph` (`} // namespace graph`).

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the complete, self-contained C++26 header enclosed within header guard defines (`#ifndef STRATUM_GRAPH_VIEWS_HPP` / `#define STRATUM_GRAPH_VIEWS_HPP` / `#endif`). Do not include conversational remarks before or after the block.
