### A. Core Grammar & Syntax Rules

- **Operators and Expressions:**
  - `begin`, `end`, `cbegin`, `cend`, `rbegin`, `rend`, `crbegin`, `crend`
  - `size`, `reserve_hint`, `ssize`

- **Declarations:**
  - `range` concept
  - `borrowed_range` concept
  - `iterator_t`, `sentinel_t`, `const_iterator_t`, `const_sentinel_t`, `range_difference_t`, `range_size_t`, `range_value_t`, `range_reference_t`, `range_const_reference_t`, `range_rvalue_reference_t`, `range_common_reference_t`
  - `sized_range` concept
  - `approximately_sized_range` concept
  - `view` concept
  - `output_range` concept, `input_range` concept, `forward_range` concept, `bidirectional_range` concept, `random_access_range` concept, `contiguous_range` concept, `common_range` concept, `viewable_range` concept, `constant_range` concept

### B. Compiler Traps & Ill-Formed Constructs (Hard Prohibitions)

- **Ill-formed constructs:**
  - If `E` is an rvalue and `enable_borrowed_range<remove_cv_t<T>>` is `false`, `ranges::begin(E)` and `ranges::end(E)` are ill-formed.
  - If `T` is an array type and `remove_all_extents_t<T>` is an incomplete type, `ranges::begin(E)` and `ranges::end(E)` are ill-formed with no diagnostic required.
  - If `T` is an array of unknown bound, `ranges::end(E)` is ill-formed.

### C. Standard-Year Purity Rules (Anti-Drift)

- **Legacy patterns superseded:**
  - Avoid using legacy template metaprogramming or type-trait patterns that are replaced by the facilities in this chapter.

### D. Feature-Test Macros & Header Requirements

- **Feature-test macros:**
  - None specified in excerpt

- **Headers:**
  - `<ranges>`
  - `<compare>` (see `ref{compare.syn}`)
  - `<initializer_list>` (see `ref{initializer.list.syn}`)
  - `<iterator>` (see `iterator.synopsis`)

