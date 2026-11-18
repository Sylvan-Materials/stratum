# Stratum Module Synthesis Contract: Graph Views & Filtered Adaptors

You are the Stratum Automation Code Generator. Translate this specification into a single, production-grade, header-only C++26 component (`graph/views/graph_views.hpp`) adhering strictly to Stratum's design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- **Standard Headers & Core Includes:** Include `"graph/graph.hpp"`, `"graph/container/dynamic_adjacency_graph.hpp"`, `<concepts>`, `<cstddef>`, `<functional>`, `<iterator>`, `<ranges>`, `<tuple>`, `<type_traits>`, and `<utility>` explicitly at the top of the file.
- **Explicit Types & Trailing Returns:** Declare concrete variable types explicitly inside function bodies. Reserve `auto` strictly for trailing return placeholders (`-> decltype(auto)` or `-> auto`) and concept-constrained return types. Never write `const auto` before a function name, and never place `auto` inside template argument lists or `requires(...)` parameter lists.
- **Structured Binding Support on View Elements:** Every view descriptor struct (`vertex_info`, `neighbor_info`, `edge_info`) must support C++17/26 structured bindings (`auto [id, u] = ...` and `auto [uid, vid, uv] = ...`) by providing `std::tuple_size` and `std::tuple_element` specializations in `namespace std` plus a `template <std::size_t I> constexpr auto get() const -> decltype(auto)` member function.
- **Standard Range Compliance:** All view return types must model `std::ranges::forward_range` (or `std::ranges::input_range` / `std::ranges::view`) so range-based `for` loops and `<ranges>` algorithms work seamlessly.

## 2. Target Component Archetype (`namespace graph::views` & `namespace graph`)
- **Purpose:** Non-owning, zero-allocation range views (`vertexlist`, `incidence`, `neighbors`, `edgelist`) and branch-pruning graph adaptor (`filtered_graph`) matching `stdgraph/P1709` and `amet`'s traversal requirements.

## 3. Exact Top-to-Bottom Declaration Sequence (Mandatory)

### Part A: View Descriptor Types inside `namespace graph`
1. **`vertex_info<VId, V>`:**
   - Stores `VId id{}; V vertex{};` (with a `void` partial specialization when `V` is `void` storing only `VId id{};`).
   - Supports structured bindings: `get<0>()` returns `id`, `get<1>()` returns `vertex`.
2. **`neighbor_info<VId, E>`:**
   - Stores `VId target_id{}; E edge{};` (with a `void` partial specialization when `E` is `void` storing only `VId target_id{};`).
   - Supports structured bindings: `get<0>()` returns `target_id`, `get<1>()` returns `edge`.
3. **`edge_info<VId, E>`:**
   - Stores `VId source_id{}; VId target_id{}; E edge{};` (with a `void` partial specialization when `E` is `void` storing `VId source_id{}; VId target_id{};`).
   - Supports structured bindings: `get<0>()` returns `source_id`, `get<1>()` returns `target_id`, `get<2>()` returns `edge`.
4. **Specialize `std::tuple_size` and `std::tuple_element` in `namespace std`** immediately after declaring `vertex_info`, `neighbor_info`, and `edge_info`.

### Part B: Range Views inside `namespace graph::views`
1. **`vertexlist(G&& g)`:**
   - Iterates over `graph::vertices(g)` and yields `vertex_info` elements containing `{graph::vertex_id(g, u), u}` (or `{graph::vertex_id(u), u}` if unary).
   - Provide an overload `vertexlist(G&& g, VVF&& vvf)` that projects vertex values alongside IDs.
2. **`incidence(G&& g, const U& u)`:**
   - Iterates over `graph::edges(g, u)` and yields `neighbor_info` elements containing `{graph::target_id(g, uv), uv}`.
   - Provide an overload `incidence(G&& g, vertex_id_t<G> uid)` that looks up `u` by `uid` and yields outgoing `neighbor_info` elements.
3. **`neighbors(G&& g, const U& u)`:**
   - Yields the target vertex descriptors or target IDs adjacent to `u` across `graph::edges(g, u)`.
4. **`edgelist(G&& g)`:**
   - Flattens `graph::vertices(g)` and `graph::edges(g, u)` (for example via `std::views::join` or a lightweight forward range/iterator) to yield `edge_info` elements `{source_id, target_id, uv}` across the entire graph.

### Part C: Branch-Pruning Adaptor inside `namespace graph` (and aliased into `namespace graph::adaptors`)
1. **`filtered_graph<G, VPred, EPred>`:**
   - Wraps a reference/pointer to an underlying graph `G&`, a vertex predicate `VPred` (defaulting to a predicate returning `true`), and an edge predicate `EPred` (defaulting to a predicate returning `true`).
   - Exposes `vertices()` filtered by `VPred` (`std::views::filter`) and `edges(u)` filtered by `EPred` (`std::views::filter`) so `filtered_graph` itself satisfies `graph::vertex_list_graph`, `graph::incidence_graph`, and `amet::viewable_incidence_graph` transparently without copying the underlying graph.
   - Provide deduction guides `filtered_graph(G&, VPred, EPred)` and `filtered_graph(G&, EPred)`.

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the complete, self-contained C++26 header enclosed within header guard defines (`#ifndef STRATUM_GRAPH_VIEWS_HPP` / `#define STRATUM_GRAPH_VIEWS_HPP` / `#endif`). Do not include conversational remarks before or after the block.
