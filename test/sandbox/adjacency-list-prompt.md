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

## 2. Target Component Archetype
- Namespace: Enclose all code elements inside the `stratum` root namespace.
- Data Structure: A header-only Directed Graph Adjacency List container (`AdjacencyList`) driven by Custom Point Objects (CPOs) and C++26 concepts.
- Internal Mechanics: Key vertex storage associatively by vertex ID (`std::unordered_map`) mapping each vertex ID to a `std::vector` of adjacent vertex descriptors so sparse or non-contiguous vertex IDs never trigger out-of-bounds access.

## 3. Custom Point Object (CPO) & Concept Architecture (Mandatory)
- **Zero Concrete Type Assumptions:** Never reference or `static_assert` against concrete user types.
- **ADL Poison-Pill Declaration:** Immediately after opening `namespace stratum`, declare a deleted or poison-pill `tag_invoke` function before defining any CPO structs so unqualified `tag_invoke` expressions inside `requires` clauses pass initial name lookup and dispatch via Argument-Dependent Lookup.
- **Constrained Overload Set Inside CPO Functors:** Inside each CPO functor struct (`vertex_id_fn` and `neighbors_fn`), do NOT use a single unconstrained `operator()` with internal `if constexpr` branching. Instead, implement each customization tier as a separate templated `operator()` overload whose signature carries its own `requires` constraint checking that specific expression:
  - **Highest-Priority Tier (ADL `tag_invoke`):** Constrained to match whenever an unqualified `tag_invoke` call taking the functor instance and target arguments is well-formed.
  - **Fallback Tiers (Structural Member Access):** Constrained to match whenever the fallback member method or member data field is well-formed *and* the higher-priority `tag_invoke` constraint is not satisfied (preventing overload ambiguity when a type defines both a member and a custom `tag_invoke`).
  - **Silent SFINAE Rejection:** When no overload's `requires` constraint is satisfied (such as for fundamental integers or unadapted containers), overload resolution fails cleanly via SFINAE without instantiating any function body.
- **No Circular Concept References in CPOs:** Never constrain vertex_id_fn::operator() using vertex_identifiable, and never constrain neighbors_fn::operator() using adjacency_list. Each CPO overload's requires clause must test only the raw expression it actually invokes (such as tag_invoke(...), v.id(), v.id, or g.neighbors(v)).
- **Strict Top-to-Bottom Declaration Order:** Define everything directly inside `namespace stratum` in this exact sequence:
  1. **`vertex_id` CPO:** Declare the `vertex_id_fn` functor struct (without `inline` or `constexpr` on `struct`) containing constrained `operator()` overloads for ADL `tag_invoke`, member method `id()`, and data member `id`. Immediately after the struct definition, declare the `inline constexpr` Niebloid instance `vertex_id`.
  2. **`vertex_identifiable` Concept:** Define the `vertex_identifiable` concept requiring that invoking `stratum::vertex_id` on a `const` vertex reference yields an integral type.
  3. **`neighbors` CPO:** Declare the `neighbors_fn` functor struct (without `inline` or `constexpr` on `struct`) containing constrained `operator()` overloads for ADL `tag_invoke` and member method `neighbors(v)`. Immediately after the struct definition, declare the `inline constexpr` Niebloid instance `neighbors`.
  4. **`adjacency_list` Concept:** Define the `adjacency_list` concept requiring that `V` satisfies `vertex_identifiable`, invoking `stratum::neighbors` on a `const` graph and `const` vertex yields a `std::ranges::forward_range`, and the range's element value type also satisfies `vertex_identifiable`.
  5. **`AdjacencyList` Reference Container:** Implement a class template `AdjacencyList` constrained on `vertex_identifiable V` using an associative map (`std::unordered_map`) keyed by vertex ID to store adjacency vectors (`std::vector<V>`). Provide a mutator to add directed edges and a `const`-qualified `neighbors` member function returning a `const` reference to the target vector. When a queried vertex is absent, return a reference to a function-local `static const` empty container instance so its storage duration outlives the call.

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the raw C++ header enclosed within header guard defines (`#ifndef` / `#define` / `#endif`). Do not include conversational remarks or explanations before or after the block.
