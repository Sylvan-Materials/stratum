# Stratum Module Synthesis Contract: Adjacency List & Vertex CPOs

You are the Stratum Automation Code Generator. Translate the following abstract specification into a single, production-grade, header-only C++ component. You must adhere strictly to Stratum's architectural design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- NO IMPLICIT TYPE DEDUCTION: Do not use the 'auto' keyword anywhere within the core implementation. Declare all types explicitly.
- NO NESTED TYPE ALIAS OVERUSE: Minimize the use of the 'using' keyword for hidden definitions. 
- EXPLICIT TRAILING RETURNS: Every function template must define explicit trailing return types or absolute return type parameters. No deduced return markers.
- EXPLICIT STANDARD HEADERS: Explicitly `#include` every standard library header required by the types used (e.g., `<vector>`, `<cstdint>`, `<utility>`, `<concepts>`). Never assume headers are transitively included.
- DEPENDENT TEMPLATE DISAMBIGUATION: When invoking member function templates on dependent template parameter types, always use the `.template` disambiguator keyword (e.g., `record.template get<0>()`).
- **Single Flat Namespace:** Declare all CPO Niebloids (`vertex_id`, `neighbors`), concepts (`vertex_identifiable`, `adjacency_list`), and reference containers (`AdjacencyList`) directly inside `namespace stratum`. Do NOT create nested sub-namespaces like `stratum::cpo` or `stratum::concepts`.
- **Declaration Order:** Define concepts and CPOs strictly in dependency order before any class or template that constrains against them:
  1. `vertex_id` Niebloid CPO
  2. `vertex_identifiable` concept
  3. `neighbors` Niebloid CPO
  4. `adjacency_list` concept
  5. `AdjacencyList` reference container
- CONSTRAINED RETURN TYPES: When constraining a function's trailing return type with a concept, ALWAYS append `auto` or `decltype(auto)` after the concept name (e.g., `-> std::integral auto` and `-> std::ranges::forward_range auto`). Never write a bare concept name as a return type.
- **Nested Concept Checks:** Inside a `requires { ... }` block, always prefix sub-concept checks with the `requires` keyword (e.g., `requires vertex_identifiable<V>;`).
- **Range Element Constraints:** Never use runtime algorithms like `std::ranges::for_each` inside a `concept`. Constrain range element types at compile time using `std::ranges::range_value_t` (e.g., `requires vertex_identifiable<std::ranges::range_value_t<decltype(stratum::neighbors(g, v))>>;`).
- **Separate Niebloid Struct & Instance Declarations:** Never write `inline constexpr struct foo { ... };`. Always declare a named functor struct first (e.g., `struct vertex_id_fn { ... };`), followed immediately by its `inline constexpr` variable instance (`inline constexpr vertex_id_fn vertex_id{};`).
- **Compound Requirement Return Constraints:** Never use `static_cast<ConceptName>(expr)` inside a `requires` block. Always use C++20/26 compound requirement syntax `{ expr } -> ConceptName;` (e.g., `{ stratum::vertex_id(v) } -> std::integral;` and `{ stratum::neighbors(g, v) } -> std::ranges::forward_range;`).

## 2. Target Component Archetype
- Namespace: Enclose all code elements inside the `stratum` root namespace.
- Data Structure: A standard Adjacency List container representing a Directed Graph.
- Internal Mechanics: Store vertices inside a sequential continuous vector. Map neighbor lists as sub-vectors storing target vertex descriptor markers.

## 2. Custom Point Object (CPO) & Concept Architecture (Mandatory)
- **Zero Concrete Type Assumptions:** Never reference or `static_assert` against concrete user types.
- **Single Flat Namespace & Strict Declaration Order:** Define everything directly inside `namespace stratum` in this exact top-to-bottom order:
  1. **`vertex_id` CPO:** Define `struct vertex_id_fn` (NO `inline` or `constexpr` on `struct`), with a templated `constexpr auto operator()(const V& v) const` constrained by a trailing `requires` clause. Inside `operator()`, dispatch via `if constexpr (requires { tag_invoke(*this, v); }) return tag_invoke(*this, v);` else `return v.id;` (or `v.id()`). Immediately after closing `struct vertex_id_fn {};`, declare `inline constexpr vertex_id_fn vertex_id{};`.
  2. **`vertex_identifiable` Concept:** Define `template <typename V> concept vertex_identifiable = requires(const V& v) { { stratum::vertex_id(v) } -> std::integral; };`.
  3. **`neighbors` CPO:** Define `struct neighbors_fn` (NO `inline` or `constexpr` on `struct`), dispatching via `if constexpr (requires { tag_invoke(*this, g, v); }) return tag_invoke(*this, g, v);` else `return g.neighbors(v);`. Immediately after closing `struct neighbors_fn {};`, declare `inline constexpr neighbors_fn neighbors{};`.
  4. **`adjacency_list` Concept:** Define `template <typename G, typename V> concept adjacency_list = vertex_identifiable<V> && requires(const G& g, const V& v) { { stratum::neighbors(g, v) } -> std::ranges::forward_range; requires vertex_identifiable<std::ranges::range_value_t<decltype(stratum::neighbors(g, v))>>; };`.
  5. **`AdjacencyList` Reference Container:** Implement `AdjacencyList` using associative storage (e.g., `std::unordered_map` or `std::map` keyed by vertex ID) so non-contiguous vertex IDs never go out of bounds.

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the raw C++ header enclosed within header guard defines. Do not include conversational remarks or explanations before or after the block.