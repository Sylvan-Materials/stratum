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
- **ADL Poison-Pill:** The very first declaration inside `namespace stratum` MUST be a zero-parameter `void tag_invoke();` function. Never pass any arguments or types to this declaration.
- **Strict Linear Compilation Sequence:** Emit every declaration in this exact top-to-bottom sequence. Every concept and functor must be fully defined before any subsequent item references it:
  1. **First (functor struct):** Define `struct vertex_id_fn` containing three separate, templated `constexpr auto operator()` overloads. Each overload must use a double-keyword trailing `requires requires` clause to form a valid constraint expression block:
     - Overload A: Takes `(const V& v)` and carries a trailing clause written exactly as `requires requires { tag_invoke(std::declval<vertex_id_fn>(), v); }`. It returns `tag_invoke(*this, v);`.
     - Overload B: Takes `(const V& v)` and carries a trailing clause written exactly as `requires requires { v.id(); }`. It returns `v.id();`.
     - Overload C: Takes `(const V& v)` and carries a trailing clause written exactly as `requires requires { v.id; }`. It returns `v.id;`.
  2. **Second (Niebloid instance):** Declare `inline constexpr vertex_id_fn vertex_id{};`.
  3. **Third (Concept):** Define `template <typename V> concept vertex_identifiable = requires(const V& v) { { stratum::vertex_id(v) } -> std::integral; };`.
  4. **Fourth (functor struct):** Define `struct neighbors_fn` containing two separate, templated `constexpr auto operator()` overloads. Each overload must use a double-keyword trailing `requires requires` clause to form a valid constraint expression block:
     - Overload A: Takes `(const G& g, const V& v)` and carries a trailing clause written exactly as `requires requires { tag_invoke(std::declval<neighbors_fn>(), g, v); }`. It returns `tag_invoke(*this, g, v);`.
     - Overload B: Takes `(const G& g, const V& v)` and carries a trailing clause written exactly as `requires requires { g.neighbors(v); }`. It returns `g.neighbors(v);`.
  5. **Fifth (Niebloid instance):** Declare `inline constexpr neighbors_fn neighbors{};`.
    6. **Sixth (Concept):** Define `template <typename G, typename V> concept adjacency_list = stratum::vertex_identifiable<V> && requires(const G& g, const V& v) { { stratum::neighbors(g, v) } -> std::ranges::forward_range; requires stratum::vertex_identifiable<std::ranges::range_value_t<decltype(stratum::neighbors(g, v))>>; };`.
  7. **Seventh (Class Template):** Define `template <stratum::vertex_identifiable V> class AdjacencyList {` (place `stratum::vertex_identifiable` directly inside the `template <...>` angle brackets—never write `requires` between `AdjacencyList` and `{`). Store elements inside a `std::unordered_map` keyed by `std::uint64_t`. Provide `add_edge(const V& source, const V& target) -> void` and a `const`-qualified `neighbors(const V& v) const -> const std::vector<V>&` method. Inside `neighbors`, if `v` is absent from the map, declare a local `static const std::vector<V> empty_vec{};` and return it (never `return {};`). End the class with `};`.


## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the raw C++ header enclosed within header guard defines (`#ifndef` / `#define` / `#endif`). Do not include conversational remarks or explanations before or after the block.
