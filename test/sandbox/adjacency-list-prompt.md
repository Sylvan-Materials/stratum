# Stratum Module Synthesis Contract: Adjacency List & Vertex CPOs

You are the Stratum Automation Code Generator. Translate this specification into a single, header-only C++26 component adhering strictly to Stratum's design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- **Standard Headers:** Include `<concepts>`, `<cstdint>`, `<ranges>`, `<unordered_map>`, `<utility>`, and `<vector>` explicitly at the top of the file.
- **Explicit Types & Trailing Returns:** Declare variable types explicitly inside function bodies. Reserve `auto` strictly for trailing return types (`-> decltype(auto)`) and concept-constrained returns (`-> std::integral auto`). Never place `auto` inside template argument lists.
- **Single Flat Namespace:** Place all CPO functors, Niebloid instances, concepts, and `AdjacencyList` directly inside `namespace stratum` (no nested sub-namespaces).
- **Separate Niebloid Declarations:** Never write `inline constexpr struct foo`. Declare the functor struct first, then declare its `inline constexpr` instance variable immediately below the closing brace.
- **Concept Syntax:** Inside `requires { ... }` blocks, prefix nested concept checks with `requires`, and use `{ expr } -> ConceptName;` for return-type constraints (never `static_cast<ConceptName>`). Never pass a type name (such as `range_value_t`) as a value argument into a function call.

## 2. Target Component Archetype
- **Directed Graph Adjacency List:** A header-only `AdjacencyList` container in `namespace stratum` driven by Custom Point Objects (CPOs) and C++26 concepts.
- **Associative Storage:** Store adjacency lists in a `std::unordered_map` keyed by vertex ID mapping to `std::vector<V>` so sparse or high-index vertex IDs never trigger out-of-bounds memory access.

## 3. Custom Point Object (CPO) & Concept Architecture (Mandatory)
- **Zero Concrete Type Assumptions:** Never reference or `static_assert` against concrete user types.
- **ADL Poison-Pill:** Immediately after opening `namespace stratum`, declare `void tag_invoke();` so unqualified `tag_invoke` calls inside `requires` clauses pass initial name lookup and dispatch via ADL.
- **Constrained Overload Sets (No Circular Concepts, No Self-Calls):**
  - Inside `vertex_id_fn` and `neighbors_fn`, implement each customization tier as a separate templated `operator()` overload constrained by its own `requires` clause on the function signature (never an unconstrained `operator()`).
  - Never reference `vertex_identifiable` or `adjacency_list` inside `vertex_id_fn` or `neighbors_fn`—those concepts are declared *below* the functors. Constrain overloads strictly on raw expressions (`tag_invoke`, `.id()`, `.id`, or `.neighbors(v)`), negating `tag_invoke` on fallback overloads to prevent ambiguity.
  - Never call `stratum::vertex_id` or `stratum::neighbors` inside `vertex_id_fn::operator()` or `neighbors_fn::operator()`; invoke `tag_invoke(*this, ...)` or the member access/method directly.
- **Strict Top-to-Bottom Declaration Order inside `namespace stratum`:**
  1. **`vertex_id` CPO:** `vertex_id_fn` functor (overloads for `tag_invoke(*this, v)`, `v.id()`, and `v.id`) followed immediately by `inline constexpr vertex_id_fn vertex_id{};`.
  2. **`vertex_identifiable` Concept:** Constrains `V` so `stratum::vertex_id(v)` yields an `std::integral` type.
  3. **`neighbors` CPO:** `neighbors_fn` functor (overloads for `tag_invoke(*this, g, v)` and `g.neighbors(v)`) followed immediately by `inline constexpr neighbors_fn neighbors{};`.
  4. **`adjacency_list` Concept:** Constrains `G` and `V` so `V` satisfies `vertex_identifiable`, `stratum::neighbors(g, v)` yields a `std::ranges::forward_range`, and a nested `requires` check confirms `std::ranges::range_value_t<decltype(stratum::neighbors(g, v))>` satisfies `vertex_identifiable`.
  5. **`AdjacencyList` Container:** Class template `AdjacencyList` constrained on `vertex_identifiable V`, mapping vertex IDs to `std::vector<V>`, with `add_edge(const V& source, const V& target) -> void` and a `const`-qualified `neighbors(const V& v) const -> const std::vector<V>&`. When `v` is absent from the map, return a reference to a function-local `static const` empty `std::vector<V>` so its storage duration outlives the call (never `return {};`).
- **• Non-Template Niebloid Structs:** Declare vertex_id_fn and neighbors_fn as non-template struct types (never put template <...> on the struct itself). Only their operator() overloads are function templates.
- **Enforce Exact Line-by-Line Sequence:** Emit declarations in this exact top-to-bottom order so no symbol is referenced before it exists:
	1. void tag_invoke();
	2. struct vertex_id_fn { ... }; followed immediately by inline constexpr vertex_id_fn vertex_id{};
	3. concept vertex_identifiable
	4. struct neighbors_fn { ... }; followed immediately by inline constexpr neighbors_fn neighbors{};
	5. concept adjacency_list
	6. template <vertex_identifiable V> class AdjacencyList

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the raw C++ header enclosed within header guard defines (`#ifndef` / `#define` / `#endif`). Do not include conversational remarks or explanations before or after the block.
