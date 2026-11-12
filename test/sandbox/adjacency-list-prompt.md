# Stratum Module Synthesis Contract: Adjacency List & Vertex CPOs

You are the Stratum Automation Code Generator. Translate the following abstract specification into a single, production-grade, header-only C++26 component. You must adhere strictly to Stratum's architectural design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- MINIMAL EXPLICIT HEADERS: Start the file with #include <concepts>, #include <cstdint>, #include <ranges>, #include <unordered_map>, #include <utility>, and #include <vector>
- EXPLICIT TYPE DECLARATIONS: Declare concrete variable types explicitly inside function bodies. Reserve `auto` strictly for deduced trailing return types (`constexpr auto operator()(...) -> decltype(auto)`) and concept-constrained return types (`-> std::integral auto`).
- NO NESTED TYPE ALIAS OVERUSE: Minimize the use of the `using` keyword for hidden definitions.
- EXPLICIT TRAILING RETURNS: Every function template must define explicit trailing return types (e.g., `-> decltype(auto)` or `-> void`).
- EXPLICIT STANDARD HEADERS: Explicitly `#include` every standard library header required by the types used (e.g., `<concepts>`, `<ranges>`, `<unordered_map>`, `<vector>`, `<cstdint>`, `<utility>`). Never assume headers are transitively included.
- DEPENDENT TEMPLATE DISAMBIGUATION: When invoking member function templates on dependent template parameter types, always use the `.template` disambiguator keyword (e.g., `record.template get<0>()`).
- SINGLE FLAT NAMESPACE: Declare all CPO Niebloids (`vertex_id`, `neighbors`), concepts (`vertex_identifiable`, `adjacency_list`), and reference containers (`AdjacencyList`) directly inside `namespace stratum`. Do NOT create nested sub-namespaces like `stratum::cpo` or `stratum::concepts`.
- CONSTRAINED RETURN TYPES: When constraining a function's trailing return type with a concept, ALWAYS append `auto` or `decltype(auto)` after the concept name (e.g., `-> std::integral auto` and `-> std::ranges::forward_range auto`). Never write a bare concept name as a return type.
- NESTED CONCEPT CHECKS: Inside a `requires { ... }` block, always prefix sub-concept checks with the `requires` keyword (e.g., `requires vertex_identifiable<V>;`).
- RANGE ELEMENT CONSTRAINTS: Never use runtime algorithms like `std::ranges::for_each` inside a `concept`. Constrain range element types at compile time using `std::ranges::range_value_t` (e.g., `requires vertex_identifiable<std::ranges::range_value_t<decltype(stratum::neighbors(g, v))>>;`).
- SEPARATE NIEBLOID STRUCT & INSTANCE DECLARATIONS: Never write `inline constexpr struct foo { ... };`. Always declare a named functor struct first (e.g., `struct vertex_id_fn { ... };`), followed immediately by its `inline constexpr` variable instance (`inline constexpr vertex_id_fn vertex_id{};`).
- COMPOUND REQUIREMENT RETURN CONSTRAINTS: Never use `static_cast<ConceptName>(expr)` inside a `requires` block. Always use C++20/26 compound requirement syntax `{ expr } -> ConceptName;` (e.g., `{ stratum::vertex_id(v) } -> std::integral;` and `{ stratum::neighbors(g, v) } -> std::ranges::forward_range;`).
- BALANCED BRACES IN `if constexpr`: Always wrap every branch of `if constexpr (...) { ... } else if constexpr (...) { ... } else { ... }` in explicit curly braces `{}` without prematurely closing `operator()`.

## 2. Target Component Archetype
- Namespace: Enclose all code elements inside the `stratum` root namespace.
- Data Structure: A header-only Directed Graph Adjacency List container (`AdjacencyList`) driven by Custom Point Objects (CPOs) and C++26 concepts.
- Internal Mechanics: Key vertex storage associatively by vertex ID (`std::unordered_map`) mapping each vertex ID to a `std::vector` of adjacent vertex descriptors so sparse or non-contiguous vertex IDs never trigger out-of-bounds access.

## 3. Custom Point Object (CPO) & Concept Architecture (Mandatory)
- **Zero Concrete Type Assumptions:** Never reference or `static_assert` against concrete user types.
- **ADL Poison-Pill Declaration:** Immediately after opening `namespace stratum`, declare `void tag_invoke();` before defining any CPO structs so unqualified `requires { tag_invoke(*this, ...); }` expressions pass initial name lookup and dispatch via ADL.
- **Mandatory SFINAE Constraints on CPO Signatures:** Every Custom Point Object call operator (operator()) MUST declare a trailing requires clause on its function signature—placed after const and before the opening brace {—that checks a disjunction (||) of valid customization expressions (ADL tag_invoke or member fallback). Because *this is out of scope in a function signature's trailing requires clause, pass a value-initialized instance of the CPO functor struct (such as vertex_id_fn{} or neighbors_fn{}) as the first argument to tag_invoke inside the signature's requires expression. Never leave a CPO call operator signature unconstrained.
- **Strict Top-to-Bottom Declaration Order:** Define everything directly inside `namespace stratum` in this exact sequence:
  1. **`vertex_id` CPO:** Define `struct vertex_id_fn` (NO `inline` or `constexpr` on `struct`), with a templated `constexpr auto operator()(const V& v) const -> decltype(auto)` constrained by a trailing `requires` clause checking that at least one dispatch path is valid. Inside `operator()`, dispatch using braced `if constexpr`:
     - `if constexpr (requires { tag_invoke(*this, v); }) { return tag_invoke(*this, v); }`
     - `else if constexpr (requires { v.id(); }) { return v.id(); }`
     - `else { return v.id; }`
   Immediately after closing `struct vertex_id_fn {};`, declare `inline constexpr vertex_id_fn vertex_id{};`. When a const member function returns a const reference to a neighbor container and the queried vertex is absent, never return a temporary or braced initializer list. Instead, return a reference to a function-local static const empty container instance so its storage duration outlives the call.
  2. **`vertex_identifiable` Concept:** Define `template <typename V> concept vertex_identifiable = requires(const V& v) { { stratum::vertex_id(v) } -> std::integral; };`.
  3. **`neighbors` CPO:** Define `struct neighbors_fn` (NO `inline` or `constexpr` on `struct`), with a templated `constexpr auto operator()(const G& g, const V& v) const -> decltype(auto)` constrained by a trailing `requires` clause checking that at least one dispatch path is valid. Inside `operator()`, dispatch using braced `if constexpr`:
     - `if constexpr (requires { tag_invoke(*this, g, v); }) { return tag_invoke(*this, g, v); }`
     - `else { return g.neighbors(v); }`
     Immediately after closing `struct neighbors_fn {};`, declare `inline constexpr neighbors_fn neighbors{};`.
   Constrain the templated call operator of every Custom Point Object with a trailing requires clause verifying that at least one customization or member fallback path is valid for the given arguments. Never leave a CPO call operator unconstrained, so unsupported types fail cleanly during concept evaluation rather than triggering hard errors inside the function body.
  4. **`adjacency_list` Concept:** Define `template <typename G, typename V> concept adjacency_list = vertex_identifiable<V> && requires(const G& g, const V& v) { { stratum::neighbors(g, v) } -> std::ranges::forward_range; requires vertex_identifiable<std::ranges::range_value_t<decltype(stratum::neighbors(g, v))>>; };`.
  5. **`AdjacencyList` Reference Container:** Implement `template <vertex_identifiable V> class AdjacencyList` using `std::unordered_map` keyed by vertex ID (`decltype(stratum::vertex_id(std::declval<V>()))`) mapping to `std::vector<V>`, providing `add_edge(const V& source, const V& target) -> void` and a `const`-qualified `neighbors(const V& v) const -> const std::vector<V>&` method that safely returns a static empty `std::vector<V>` if `v` has no outgoing edges.

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the raw C++ header enclosed within header guard defines (`#ifndef` / `#define` / `#endif`). Do not include conversational remarks or explanations before or after the block.
