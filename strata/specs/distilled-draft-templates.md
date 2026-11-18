### A. Core Grammar & Syntax Rules
- **template-declaration**: Defines a family of classes, functions, variables, aliases for types, or concepts.
  - `template-head declaration`
  - `template-head concept-definition`

- **template-head**:
  - `template <template-parameter-list> [requires-clause]`

- **template-parameter-list**:
  - `template-parameter`
  - `template-parameter-list, template-parameter`

- **requires-clause**:
  - `requires constraint-logical-or-expression`

- **constraint-logical-or-expression**:
  - `constraint-logical-and-expression`
  - `constraint-logical-or-expression || constraint-logical-and-expression`

- **constraint-logical-and-expression**:
  - `primary-expression`
  - `constraint-logical-and-expression && primary-expression`

### B. Compiler Traps & Ill-Formed Constructs (Hard Prohibitions)
- A template-declaration shall declare or define a function, class, variable, member function, member class, member enumeration, static data member of a class template or nested within a class template, member template of a class or class template, friend-type-declaration, deduction-guide, or alias-declaration. It shall not be a linkage-specification.
- A template-declaration is a declaration. A variable template at class scope is a static data member template.
- In a function template declaration, the unqualified-id of the declarator-id shall be a name.
- A class or variable template declaration of a simple-template-id declares a partial specialization.
- In a template-declaration, explicit specialization or explicit instantiation, the init-declarator-list in the declaration shall contain at most one declarator. When such a declaration is used to declare a class template, no declarator is permitted.
- A specialization (explicit or implicit) of one template is distinct from all specializations of any other template.
- Default arguments for function templates and member functions of class templates are considered definitions for the purpose of template instantiation and must obey the one-definition rule.
- A template cannot have the same name as any other name bound in the same scope, except that a function template can share a name with using-declarators, a type, non-template functions, and/or function templates. Specializations do not reintroduce or bind names; their target scope is the target scope of the primary template.
- A local class, a local or block variable, or a friend function defined in a templated entity is a templated entity.

### C. Standard-Year Purity Rules (Anti-Drift)
- Legacy C++ template metaprogramming and type-trait patterns such as `__attribute__((deprecated))` are superseded by the facilities introduced in this chapter and must be avoided when these features apply.

### D. Feature-Test Macros & Header Requirements
- `<cstddef>`
- `<type_traits>`
- `None specified in excerpt`

