# Stratum Module Synthesis Contract: Adjacency List & Vertex CPOs

You are the Stratum Automation Code Generator. Translate this specification into a single, header-only C++26 component adhering strictly to Stratum's design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- **Standard Headers:** Include `<concepts>`, `<cstdint>`, `<ranges>`, `<unordered_map>`, `<utility>`, and `<vector>` explicitly at the top of the file.
- **Explicit Types & Trailing Returns:** Declare variable types explicitly inside function bodies. Reserve `auto` strictly for trailing return types (`-> decltype(auto)`) and concept-constrained returns (`-> std::integral auto`). Never place `auto` inside template argument lists.
- **Single Flat Namespace:** Place all CPO functors, Niebloid instances, concepts, and `AdjacencyList` directly inside `namespace stratum` (no nested sub-namespaces).
- **Separate Niebloid Declarations:** Never write `inline constexpr struct foo`. Declare the functor struct first, then declare its `inline constexpr` instance variable immediately below the closing brace.
- **Concept Syntax:** Inside `requires { ... }` blocks, prefix nested concept checks with `requires`, and use `{ expr } -> ConceptName;` for return-type constraints (never `static_cast<ConceptName>`). Never pass a type name (such as `range_value_t`) as a value argument into a function call.

## 2. Target Component Archetype
- **Directed Graph Adjacency List:** A header-only `AdjacencyList` container driven by Custom Point Objects (CPOs) and C++26 concepts.
- **Associative Storage:** Store adjacency lists in a `std::unordered_map` keyed by vertex ID mapping to `std::vector<V>` so sparse or high-index vertex IDs never trigger out-of-bounds memory access.

## 3. Custom Point Object (CPO) & Concept Architecture (Mandatory)
- **Zero Concrete Type Assumptions:** Never reference or `static_assert` against concrete user types.
- **ADL Poison-Pill:** The very first declaration inside `namespace stratum` MUST be a zero-parameter `void tag_invoke();` function so unqualified `tag_invoke` calls inside `requires` clauses pass initial name lookup and dispatch via ADL. Never pass `vertex_id_fn` or `neighbors_fn` as parameters to this poison pill.
- **Constrained Overload Sets (No Circular Concepts, No Self-Calls):**
  - Inside `vertex_id_fn` and `neighbors_fn`, implement each customization tier as a separate templated `operator()` overload constrained by its own `requires` clause on the function signature (never an unconstrained `operator()`).
  - Never reference `vertex_identifiable` or `adjacency_list` inside `vertex_id_fn` or `neighbors_fn`—those concepts are declared *below* the functors. Constrain overloads strictly on raw expressions (`tag_invoke`, `.id()`, `.id`, or `.neighbors(v)`), negating `tag_invoke` on fallback overloads to prevent ambiguity.
  - Inside a function signature's trailing `requires` clause, never use `*this` (which is only valid inside the function body); pass a value-initialized tag instance (`vertex_id_fn{}` or `neighbors_fn{}`) as the first argument to `tag_invoke`. Inside the body, invoke `tag_invoke(*this, ...)` or the member access/method directly (never recursively call `stratum::vertex_id` or `stratum::neighbors`).
- **Mandatory Template Parameter Headers on Concepts:** Every `concept` declaration MUST begin with an explicit `template <...>` parameter list—never omit the `template` head or use `auto` inside a `requires(...)` parameter list.
- **Strict Top-to-Bottom Declaration Order inside `namespace stratum`:**
  1. **`vertex_id` CPO:** `vertex_id_fn` functor (overloads for `tag_invoke`, `v.id()`, and `v.id`) followed immediately by `inline constexpr vertex_id_fn vertex_id{};`.
  2. **`vertex_identifiable` Concept:** Explicitly declared as a `template <typename V>` concept requiring that invoking `stratum::vertex_id` on a `const V&` yields an `std::integral` type.
  3. **`neighbors` CPO:** `neighbors_fn` functor (overloads for `tag_invoke` and `g.neighbors(v)`) followed immediately by `inline constexpr neighbors_fn neighbors{};`.
  4. **`adjacency_list` Concept:** Explicitly declared as a `template <typename G, typename V>` concept requiring that `V` satisfies `vertex_identifiable`, `stratum::neighbors(g, v)` yields a `std::ranges::forward_range` when binding `const G& g` and `const V& v` directly, and a nested `requires vertex_identifiable<...>;` statement without curly braces confirms `std::ranges::range_value_t<decltype(stratum::neighbors(g, v))>` satisfies `vertex_identifiable`.
  5. **`AdjacencyList` Container:** Declare `AdjacencyList` as a single class template parameterized on `vertex_identifiable V` at the class level (never use templated member aliases inside a non-template class). Map vertex IDs to `std::vector<V>`, providing `add_edge(const V& source, const V& target) -> void` and a `const`-qualified `neighbors(const V& v) const -> const std::vector<V>&`. When `v` is absent from the map, return a reference to a function-local `static const` empty `std::vector<V>` so its storage duration outlives the call (never `return {};`).

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the raw C++ header enclosed within header guard defines (`#ifndef` / `#define` / `#endif`). Do not include conversational remarks or explanations before or after the block.
