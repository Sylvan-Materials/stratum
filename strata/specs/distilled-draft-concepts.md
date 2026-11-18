### A. Core Grammar & Syntax Rules

- **Concepts Library**: The library provides components for compile-time validation of template arguments and function dispatch based on type properties.
- **Language-Related Concepts**:
  - `same_as`: Checks if two types are the same, considering cv-qualification.
  - `derived_from`: Checks if one class is derived from another.
  - `convertible_to`: Checks if an expression can be implicitly and explicitly converted to a target type.
  - `common_reference_with`: Determines if two types share a common reference type.
  - `common_with`: Finds the common type of two types, considering cv-qualification.
- **Arithmetic Concepts**:
  - `integral`, `signed_integral`, `unsigned_integral`, `floating_point`: Define properties of integral and floating-point types.
- **Assignable Concepts**: Checks if an expression can be assigned to a lvalue reference.
- **Swappable Concepts**:
  - `swappable`: Checks if two objects can be swapped using the `ranges::swap` function.
  - `swappable_with`: Checks if two types are swappable with each other.
- **Destructible Concepts**: Ensures that an object type can be destroyed without throwing exceptions.
- **Constructible Concepts**:
  - `constructible_from`: Checks if a type can be constructed from given arguments.
  - `default_initializable`, `move_constructible`, `copy_constructible`: Define properties of constructors.

### B. Compiler Traps & Ill-Formed Constructs (Hard Prohibitions)

- **Equality-Preserving Expressions**: An expression is equality-preserving if it produces the same output for equal inputs.
- **Domain of Expression**: The domain of an expression includes all valid input values for which the expression must be well-defined.
- **Stability Requirement**: Equality-preserving expressions must produce stable results, meaning two evaluations with the same input objects should yield the same outputs absent explicit modification.
- **Modifying Inputs**: Expressions that modify their inputs in a manner observable to equality-preserving expressions are not allowed.

### C. Standard-Year Purity Rules (Anti-Drift)

- Legacy template metaprogramming and type-trait patterns such as `std::is_same`, `std::is_base_of`, etc., should be avoided when using the concepts library.
- The concepts library provides more modern, expressive ways to perform compile-time checks and dispatch based on type properties.

### D. Feature-Test Macros & Header Requirements

- **Header**: `<concepts>`
- **Feature-Test Macro**: `__cpp_concepts`

