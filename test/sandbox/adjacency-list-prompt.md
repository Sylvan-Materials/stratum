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
- **ADL Poison-Pill:** The very first declaration inside `namespace stratum` MUST be `void tag_invoke();`. Never pass any parameters to this declaration.
- **Strict top-to-bottom declaration order (No Forward References):**
  1. **`vertex_id_fn` Struct:** Define `struct vertex_id_fn` as a plain non-template struct. Inside it, define three distinct, standalone `constexpr auto operator()` overloads:
     - Overload A: Takes `const V& v` and is constrained strictly by `requires { tag_invoke(std::declval<vertex_id_fn>{}, v); }`. It returns `tag_invoke(std::declval<vertex_id_fn>{}, v);`.
     - Overload B: Takes `const V& v` and is constrained strictly by `requires { v.id(); }`. It returns `v.id();`.
     - Overload C: Takes `const V& v` and is constrained strictly by `requires { v.id; }`. It returns `v.id;`.
  2. **`vertex_id` Niebloid Object:** Immediately after closing the struct, declare `inline constexpr vertex_id_fn vertex_id{};`.
  3. **`vertex_identifiable` Concept:** Define `template <typename V> concept vertex_identifiable = requires(const V& v) { { stratum::vertex_id(v) } -> std::integral; };`.
  4. **`neighbors_fn` Struct:** Define `struct neighbors_fn` as a plain non-template struct. Inside it, define two distinct, standalone `constexpr auto operator()` overloads:
     - Overload A: Takes `const G& g, const V& v` and is constrained strictly by `requires { tag_invoke(std::declval<neighbors_fn>{}, g, v); }`. It returns `tag_invoke(std::declval<neighbors_fn>{}, g, v);`.
     - Overload B: Takes `const G& g, const V& v` and is constrained strictly by `requires { g.neighbors(v); }`. It returns `g.neighbors(v);`.
  5. **`neighbors` Niebloid Object:** Immediately after closing the struct, declare `inline constexpr neighbors_fn neighbors{};`. Ensure the entire AdjacencyList class definition template is properly terminated with a trailing semicolon (;) immediately following its final closing brace.
  6. **`adjacency_list` Concept:** Define `template <typename G, typename V> concept adjacency_list = stratum::vertex_identifiable<V> && requires(const G& g, const V& v) { { stratum::neighbors(g, v) } -> std::ranges::forward_range; } && stratum::vertex_identifiable<std::ranges::range_value_t<decltype(stratum::neighbors(std::declval<const G&>(), std::declval<const V&>()))>>;`.
  7. **`AdjacencyList` Container:** Declare `AdjacencyList` as a class template constrained directly inside the class-level template brackets (using `template <stratum::vertex_identifiable V> class AdjacencyList {`). Store elements in a `std::unordered_map` keyed by `std::uint64_t` or the vertex ID type, mapping to `std::vector<V>`. Provide a mutator named `add_edge(const V& source, const V& target) -> void`. Provide a `const`-qualified `neighbors(const V& v) const -> const std::vector<V>&` method. Inside this method, if `v` is absent from the storage map, declare a local function-static `static const std::vector<V> empty_vec{};` and return it (never return an empty brace list `{}`). Terminate the entire class definition block with a trailing semicolon (`;`) immediately following its final closing brace.


## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the raw C++ header enclosed within header guard defines (`#ifndef` / `#define` / `#endif`). Do not include conversational remarks or explanations before or after the block.
