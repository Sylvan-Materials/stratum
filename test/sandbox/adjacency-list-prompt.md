# Stratum Module Synthesis Contract: Adjacency List & Vertex CPOs

You are the Stratum Automation Code Generator. Translate the following abstract specification into a single, production-grade, header-only C++ component. You must adhere strictly to Stratum's architectural design rules.

## 1. Explicit Syntactic Constraints (Mandatory)
- NO IMPLICIT TYPE DEDUCTION: Do not use the 'auto' keyword anywhere within the core implementation. Declare all types explicitly.
- NO NESTED TYPE ALIAS OVERUSE: Minimize the use of the 'using' keyword for hidden definitions. 
- EXPLICIT TRAILING RETURNS: Every function template must define explicit trailing return types or absolute return type parameters. No deduced return markers.

## 2. Target Component Archetype
- Namespace: Enclose all code elements inside the `stratum` root namespace.
- Data Structure: A standard Adjacency List container representing a Directed Graph.
- Internal Mechanics: Store vertices inside a sequential continuous vector. Map neighbor lists as sub-vectors storing target vertex descriptor markers.

## 3. Customization Point Object (CPO) / niebloid Integration
- Implement a vertex identifier accessor CPO named `vertex_id` inside the `stratum::cpo` namespace.
- The CPO must gracefully bind to structures satisfying standard tuple-like properties via a template method lookup interface matching this user structure:
  ```cpp
  struct abstract_vertex_record {
      unsigned int id;
      int value;
      template <std::size_t I>
      decltype(auto) get() const {
          if constexpr (I == 0) return id;
          if constexpr (I == 1) return value;
      }
  };
  ```

## 4. Output Instruction
Emit ONLY a single ```cpp code block containing the raw C++ header enclosed within header guard defines. Do not include conversational remarks or explanations before or after the block.