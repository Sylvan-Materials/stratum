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
Emit this exact namespace transition and specialization block immediately after `edge_info<VId, void>;`:
```cpp
} // namespace graph

namespace std {
template <typename VId, typename V> struct tuple_size<graph::vertex_info<VId, V>> : integral_constant<size_t, 2> {};
template <typename VId> struct tuple_size<graph::vertex_info<VId, void>> : integral_constant<size_t, 1> {};
template <typename VId, typename E> struct tuple_size<graph::neighbor_info<VId, E>> : integral_constant<size_t, 2> {};
template <typename VId> struct tuple_size<graph::neighbor_info<VId, void>> : integral_constant<size_t, 1> {};
template <typename VId, typename E> struct tuple_size<graph::edge_info<VId, E>> : integral_constant<size_t, 3> {};
template <typename VId> struct tuple_size<graph::edge_info<VId, void>> : integral_constant<size_t, 2> {};

template <size_t I, typename VId, typename V> struct tuple_element<I, graph::vertex_info<VId, V>> {
    using type = decltype(declval<graph::vertex_info<VId, V>>().template get<I>());
};
template <size_t I, typename VId> struct tuple_element<I, graph::vertex_info<VId, void>> {
    using type = decltype(declval<graph::vertex_info<VId, void>>().template get<I>());
};
template <size_t I, typename VId, typename E> struct tuple_element<I, graph::neighbor_info<VId, E>> {
    using type = decltype(declval<graph::neighbor_info<VId, E>>().template get<I>());
};
template <size_t I, typename VId> struct tuple_element<I, graph::neighbor_info<VId, void>> {
    using type = decltype(declval<graph::neighbor_info<VId, void>>().template get<I>());
};
template <size_t I, typename VId, typename E> struct tuple_element<I, graph::edge_info<VId, E>> {
    using type = decltype(declval<graph::edge_info<VId, E>>().template get<I>());
};
template <size_t I, typename VId> struct tuple_element<I, graph::edge_info<VId, void>> {
    using type = decltype(declval<graph::edge_info<VId, void>>().template get<I>());
};
} // namespace std
```

### Part C: Range Views inside `namespace graph::views`
Open `namespace graph::views {` and define these range views using `std::views::transform` (never `std::transform_view`, and never reference nonexistent traits like `graph::vertex_value_t` or `graph::edge_value_t`—use CTAD `graph::vertex_info{...}`, `graph::neighbor_info{...}`, and `graph::edge_info{...}` or `graph::vertex_t<G>` / `graph::edge_t<G>`):
1. **`template <typename G> auto vertexlist(G&& g) -> decltype(auto)`:**
   - Returns `graph::vertices(g) | std::views::transform([&g](const auto& u) { return graph::vertex_info{graph::vertex_id(g, u), u}; })`.
2. **`template <typename G, typename VVF> auto vertexlist(G&& g, VVF&& vvf) -> decltype(auto)`:**
   - Explicitly declare `template <typename G, typename VVF>` above the signature. Returns `graph::vertices(g) | std::views::transform([&g, vvf = std::forward<VVF>(vvf)](const auto& u) { return graph::vertex_info{graph::vertex_id(g, u), std::invoke(vvf, g, u)}; })`.
3. **`template <typename G, typename U> auto incidence(G&& g, const U& u) -> decltype(auto)`:**
   - Returns `graph::edges(g, u) | std::views::transform([&g](const auto& e) { return graph::neighbor_info{graph::target_id(g, e), e}; })`.
4. **`template <typename G> auto incidence(G&& g, graph::vertex_id_t<G> uid) -> decltype(auto)`:**
   - Returns `graph::edges(g, uid) | std::views::transform([&g](const auto& e) { return graph::neighbor_info{graph::target_id(g, e), e}; })`.
5. **`template <typename G, typename U> auto neighbors(G&& g, const U& u) -> decltype(auto)`:**
   - Returns `graph::edges(g, u) | std::views::transform([&g](const auto& e) { return graph::target_id(g, e); })`.
6. **`template <typename G> auto edgelist(G&& g) -> decltype(auto)`:**
   - Never pass arguments to `std::views::join`. First pipe `graph::vertices(g)` into `std::views::transform([&g](const auto& u) { return graph::edges(g, u) | std::views::transform([&g, &u](const auto& e) { return graph::edge_info{graph::vertex_id(g, u), graph::target_id(g, e), e}; }); })`, and then pipe that result into `| std::views::join`.
Close `namespace graph::views` (`} // namespace graph::views`).


### Part D: Branch-Pruning Adaptor inside `namespace graph` & `namespace graph::adaptors`
Open `namespace graph {` and define:
1. **`template  class filtered_graph`:**
   - Wraps a pointer/reference to `G` along with vertex predicate `VPred` and edge predicate `EPred` (defaulting to always-true predicates when omitted).
   - Provides `vertices()` filtered by `VPred` and `edges(u)` filtered by `EPred` so `filtered_graph` satisfies `graph::vertex_list_graph`, `graph::incidence_graph`, and `amet::viewable_incidence_graph`.
   - Provide deduction guides and alias `using graph::filtered_graph;` inside `namespace graph::adaptors`.
   - Do not declare `using edge_id_type` or reference nonexistent `graph::edge_id_t` inside `filtered_graph` (use `graph::vertex_id_t<G>`, `graph::vertex_t<G>`, and `graph::edge_t<G>` only).

Close `namespace graph` (`} // namespace graph`) after the deduction guides, then open `namespace graph::adaptors { using graph::filtered_graph; } // namespace graph::adaptors` (never write an unqualified `using filtered_graph;` outside a namespace).


## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the complete, self-contained C++26 header enclosed within header guard defines (`#ifndef STRATUM_GRAPH_VIEWS_HPP` / `#define STRATUM_GRAPH_VIEWS_HPP` / `#endif`). Do not include conversational remarks before or after the block.
