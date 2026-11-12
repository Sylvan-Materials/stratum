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

## 2. Target Component Archetype
- Namespace: Enclose all code elements inside the `stratum` root namespace.
- Data Structure: A standard Adjacency List container representing a Directed Graph.
- Internal Mechanics: Store vertices inside a sequential continuous vector. Map neighbor lists as sub-vectors storing target vertex descriptor markers.

## 2. Custom Point Object (CPO) & Concept Architecture (Mandatory)
- **Zero Concrete Type Assumptions:** Never reference, forward-declare, or `static_assert` against concrete user types (such as `abstract_vertex_record`). The library header must remain completely agnostic to user-defined vertex and edge structs.
- **Niebloid CPO Pattern:** Implement all customization points as `inline constexpr` Niebloid function objects inside `namespace stratum::cpo` (and lift them into `namespace stratum` via `using`).
- **Two-Tier CPO Dispatch Priority:** Each CPO (e.g., `vertex_id`, `neighbors`) must evaluate customization in strict priority order using `if constexpr` and `requires` expressions:
  1. **ADL / `tag_invoke` Customization:** Check if a valid `tag_invoke(cpo_tag, args...)` overload is found via Argument-Dependent Lookup.
  2. **Structural Member Fallback:** If no `tag_invoke` overload exists, check for a valid member function or data member (e.g., `v.id()` or `v.id` for `vertex_id`; `g.neighbors(v)` for `neighbors`).
  3. **Hard Concept Failure:** If neither customization path is satisfied, constrain `operator()` with a `requires` clause so invalid calls fail cleanly via SFINAE/concept diagnostics rather than failing inside the function body.
- **Graph Theory Concepts:** Define explicit C++20/26 `concept` definitions in `namespace stratum::concepts` built directly on top of the CPOs:
  - `vertex_identifiable<V>`: Requires `stratum::vertex_id(v)` to be well-formed and return an `std::integral` type.
  - `adjacency_list<G, V>`: Requires `vertex_identifiable<V>` and requires `stratum::neighbors(g, v)` to return a `std::ranges::forward_range` whose elements satisfy `vertex_identifiable`.
- **Safe Container Invariants:** Any reference `AdjacencyList` implementation included in the header must store vertices using associative or dynamically resized storage keyed by `stratum::vertex_id(v)` so non-contiguous or high-index vertex IDs never trigger out-of-bounds memory access.

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the raw C++ header enclosed within header guard defines. Do not include conversational remarks or explanations before or after the block.