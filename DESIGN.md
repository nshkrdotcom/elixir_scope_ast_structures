# Design Document: ElixirScope.AST.Structures (elixir_scope_ast_structures)

## 1. Purpose & Vision

**Summary:** Defines the core data structures for representing Abstract Syntax Trees (ASTs), Control Flow Graphs (CFGs), Data Flow Graphs (DFGs), and Code Property Graphs (CPGs) within ElixirScope. Also includes structures for enhanced module/function data and various analysis results.

**(Greatly Expanded Purpose based on your existing knowledge of ElixirScope and CPG features):**

The `elixir_scope_ast_structures` library serves as the canonical schema definition for all static analysis artifacts produced and consumed by ElixirScope. Its fundamental purpose is to provide well-defined, robust, and expressive Elixir structs for representing the multifaceted nature of source code, from its syntactic form (AST) to its control flow (CFG), data flow (DFG), and the unified Code Property Graph (CPG).

This library aims to:
*   **Standardize Static Representations:** Offer a consistent set of structs for AST nodes, CFG nodes/edges, DFG nodes/edges (including SSA concepts), and CPG nodes/edges. This ensures interoperability between the `elixir_scope_ast_repo` (producer) and other consumer libraries.
*   **Enable Rich Analysis:** Design structures that can hold detailed metadata, analysis results (complexity, patterns, security flags), and cross-references (like AST Node IDs, CPG IDs) necessary for deep code understanding.
*   **Support CPG Vision:** Explicitly define structures for CPGs, accommodating the types of nodes (AST, CFG, DFG origins), edges (control, data, syntax, inter-procedural, semantic), and properties required by the advanced CPG algorithms (`CPGMath`, `CPGSemantics`) and AI features.
*   **Define Enhanced Data Containers:** Provide `EnhancedModuleData.t()` and `EnhancedFunctionData.t()` structs that aggregate raw ASTs with their derived CFGs, DFGs, CPGs, and various analysis metrics.
*   **Encapsulate Analysis Results:** Define structs for storing results from various analyses (complexity, security, quality, path analysis, etc.).
*   **Facilitate Serialization:** Ensure these structures can be efficiently serialized (e.g., for caching or storage by `elixir_scope_ast_repo`).

The data structures defined here are the very "property" in "Code Property Graph." They are what the `elixir_scope_ast_repo` will build and what tools and AI assistants interacting via `TidewaveScope` will ultimately query to understand code deeply. For example, a CPG node struct might include fields for its type (e.g., `:ast_call`, `:cfg_decision`), its text representation, incoming/outgoing control/data/syntax edges, associated `ast_node_id`, and computed properties like "centrality_score" or "is_taint_source".

This library will enable:
*   The `elixir_scope_ast_repo` to construct and store consistent static analysis artifacts.
*   The `elixir_scope_correlator` to link runtime events (via `ast_node_id`) to specific elements within these static structures.
*   The `elixir_scope_ai` library to consume rich, structured code representations for its analysis and prediction models.
*   `TidewaveScope` MCP tools to request and display detailed information about code structure and properties.

## 2. Key Responsibilities

This library is responsible for:

*   Defining Elixir structs for:
    *   Basic AST Node elements (if a custom representation beyond Elixir's quoted form is needed, though likely we'll augment the standard AST).
    *   **CFG Components:** `CFGData.t()`, `CFGNode.t()`, `CFGEdge.t()`.
    *   **DFG Components:** `DFGData.t()`, `DFGNode.t()`, `DFGEdge.t()`, `VariableVersion.t()`, `Definition.t()`, `Use.t()`, `PhiNode.t()`.
    *   **CPG Components:** `CPGData.t()`, `CPGNode.t()` (with variants for AST-derived, CFG-derived, DFG-derived nodes), `CPGEdge.t()` (with variants for different edge types like `AST`, `CF`, `DF`, `REACHES`, `CALLS`, `CONTAINS`, etc.).
    *   **Enhanced Module & Function Data:** `EnhancedModuleData.t()`, `EnhancedFunctionData.t()`.
    *   **Supporting Structures:** `ScopeInfo.t()`, `ComplexityMetrics.t()`, `PathAnalysis.t()`, `TypeInfo.t()`, `PatternMatch.t()`, `SecurityVulnerability.t()`, `OptimizationHint.t()`, etc., covering all analysis outputs and metadata.
*   Providing `@typedoc` and `@type` definitions for all public structs and their fields.
*   Ensuring these data structures are self-contained or rely only on `elixir_scope_utils` or basic Elixir types.
*   Defining common enumerations or constants related to these structures (e.g., node types, edge types).

## 3. Key Modules & Structure

This library will primarily consist of modules defining structs. Organization can be by graph type or feature.

### Proposed File Tree:

```
elixir_scope_ast_structures/
├── lib/
│   └── elixir_scope/
│       └── ast/
│           ├── structures/              # Main directory for struct definitions
│           │   ├── ast_node.ex          # (If a custom AST node wrapper is needed)
│           │   ├── cfg_data.ex          # Defines CFGData, CFGNode, CFGEdge
│           │   ├── dfg_data.ex          # Defines DFGData, DFGNode, DFGEdge, VariableVersion, etc.
│           │   ├── cpg_data.ex          # Defines CPGData, CPGNode, CPGEdge
│           │   ├── enhanced_module_data.ex
│           │   ├── enhanced_function_data.ex
│           │   ├── scope_info.ex
│           │   ├── complexity_metrics.ex
│           │   ├── path_analysis.ex     # And LoopAnalysis, BranchCoverage, etc.
│           │   ├── security_analysis_structs.ex # For SecurityRisk, TaintFlow etc.
│           │   ├── quality_analysis_structs.ex  # For CodeSmell etc.
│           │   ├── variable_data.ex
│           │   ├── supporting_structures.ex # For MacroData, TypespecData etc. from original
│           │   └── node_mappings.ex     # For NodeMappings, QueryIndexes etc. from original CPGData
│           └── legacy_structures.ex # (Optional) For ModuleData, FunctionData if needed for migration
├── mix.exs
├── README.md
├── DESIGN.MD
└── test/
    ├── test_helper.exs
    └── elixir_scope/
        └── ast/
            └── structures_test.exs # Tests struct definitions and basic properties
```

**(Greatly Expanded - Module/Structure Description):**
*   **`ElixirScope.AST.Structures.CFGData` (and related `CFGNode`, `CFGEdge`):** These will define the structure for Control Flow Graphs, including nodes for basic blocks, decision points (if, case, cond), entry/exit points, and edges representing control flow transitions (sequential, conditional true/false, exception). Nodes will link back to AST Node IDs.
*   **`ElixirScope.AST.Structures.DFGData` (and related `DFGNode`, `DFGEdge`, `VariableVersion`, `Definition`, `Use`, `PhiNode`):** These define Data Flow Graphs, potentially in SSA form. `DFGNode`s represent operations or variable states. `VariableVersion` captures SSA versions. Edges show data dependencies. `PhiNode`s handle merging variable versions at control flow joins.
*   **`ElixirScope.AST.Structures.CPGData` (and related `CPGNode`, `CPGEdge`):** This is the core.
    *   `CPGNode.t()`: Will be a flexible struct, possibly with a `:type` field (e.g., `:ast_call`, `:ast_literal`, `:cfg_basic_block`, `:dfg_variable_def`, `:interprocedural_call_site`) and a `:properties` map. It must store the original `ast_node_id` if derived from an AST node. It will also store computed metrics from `CPGMath` and semantic tags from `CPGSemantics`.
    *   `CPGEdge.t()`: Will also have a `:type` (e.g., `:ast_parent_child`, `:cfg_successor`, `:dfg_reaches`, `:cpg_call_invocation`, `:cpg_data_influence`, `:cpg_semantic_link`) and a `:properties` map.
*   **`ElixirScope.AST.Structures.EnhancedModuleData` and `EnhancedFunctionData`**: These structs will serve as containers, holding the raw AST for a module/function, along with references or embedded instances of its CFG, DFG, and CPG, plus aggregated complexity metrics and analysis results.
*   **`ElixirScope.AST.Structures.ComplexityMetrics`**: A struct to hold various complexity scores (cyclomatic, cognitive, Halstead, custom CPG-based scores like path complexity).
*   Other modules will define structs for specific analysis results like `PathAnalysis`, `SecurityVulnerability`, `CodeSmell`, etc.

## 4. Public API (Conceptual)

This library's public API is primarily the set of defined structs and their associated `@type` definitions. There might be utility functions for constructing or validating these structs, but no major operational APIs.

*   Access to all defined struct modules (e.g., `ElixirScope.AST.Structures.CPGNode`).
*   Access to all `@type t()` definitions for these structs.

## 5. Core Data Structures

This library *is* the definition of core data structures. Examples include:

*   **`ElixirScope.AST.Structures.CPGNode.t()` (Conceptual Example):**
    ```elixir
    defmodule ElixirScope.AST.Structures.CPGNode do
      @typedoc "A node in the Code Property Graph."
      @type t :: %__MODULE__{
              id: String.t(),                       # Unique CPG Node ID
              type: atom(),                         # e.g., :ast_call, :cfg_block, :dfg_def, :semantic_entity
              ast_node_id: String.t() | nil,        # Link to original AST node, if applicable
              label: String.t() | nil,              # Human-readable label (e.g., function name, variable name)
              code_snippet: String.t() | nil,       # Relevant code snippet
              line_start: non_neg_integer() | nil,
              line_end: non_neg_integer() | nil,
              properties: map(),                    # Arbitrary key-value properties (e.g., {name: "my_var"}, {operator: "+"}), including CPGMath/CPGSemantics results
              # Edges might be stored separately or referenced
              # incoming_edges: list(String.t()), # List of CPGEdge IDs
              # outgoing_edges: list(String.t())  # List of CPGEdge IDs
            }
      defstruct [:id, :type, :ast_node_id, :label, :code_snippet, :line_start, :line_end, :properties]
    end
    ```

*   **`ElixirScope.AST.Structures.CPGEdge.t()` (Conceptual Example):**
    ```elixir
    defmodule ElixirScope.AST.Structures.CPGEdge do
      @typedoc "An edge in the Code Property Graph."
      @type t :: %__MODULE__{
              id: String.t(),                       # Unique CPG Edge ID
              from_node_id: String.t(),
              to_node_id: String.t(),
              type: atom(),                         # e.g., :ast_child_of, :control_flow, :data_flow_reaches, :call_graph, :refers_to
              label: String.t() | nil,              # e.g., "IF_TRUE", "ARGUMENT_1"
              properties: map()                     # Arbitrary key-value properties
            }
      defstruct [:id, :from_node_id, :to_node_id, :type, :label, :properties]
    end
    ```

*   (And all others as listed in Key Responsibilities, drawing from your existing `Repomix` and CPG design documents.)

## 6. Dependencies

This library will depend on the following ElixirScope libraries:

*   `elixir_scope_utils` (potentially for common types or utility functions if any are used in struct defaults or simple validations).

It will primarily define types and structs, having minimal runtime logic.

## 7. Role in TidewaveScope & Interactions

Within the `TidewaveScope` ecosystem, the `elixir_scope_ast_structures` library will:

*   Provide the schema for all static analysis data generated by `elixir_scope_ast_repo`.
*   Define the structure of data queried by `elixir_scope_correlator` when linking runtime events to static code.
*   Define the input format for AI models in `elixir_scope_ai` that consume static code representations.
*   Be used by `TidewaveScope`'s MCP tools when they request or display detailed structural information about the codebase.
*   The `elixir_scope_compiler` will consume AST-related struct definitions if it needs to understand specific properties of the AST it's transforming (beyond just the quoted form).

## 8. Future Considerations & CPG Enhancements

*   **Graph Database Compatibility:** As CPGs grow, if a dedicated graph database is considered, these structs might need to evolve to map more easily to graph DB schemas (e.g., properties becoming more standardized).
*   **Versioning of Structures:** If structures change significantly, a versioning field within the structs or a migration path might be needed for persisted data.
*   **Inter-procedural Data Structures:** Clear structures for representing inter-procedural call graph edges, summary edges for data flow, and context sensitivity will be critical and defined here.
*   **Semantic Overlay Structures:** As `CPGSemantics` matures, it might introduce new "virtual" node or edge types, or complex property structures, which would be defined in this library.

## 9. Testing Strategy

*   **Unit Tests:** For each struct definition:
    *   Test creation with valid default or example data.
    *   Test that fields have the correct expected types (can be partially checked with Dialyzer if typespecs are complete).
    *   If any structs have simple helper functions (e.g., `CPGNode.get_property(node, key)`), test these.
*   **No complex logic to test here:** The primary role is definition. The usage and population of these structs will be tested in the libraries that use them (e.g., `elixir_scope_ast_repo` tests will assert it creates valid `CPGData` structures).
*   **Dialyzer:** Extensive use of `@typedoc` and `@type` will allow Dialyzer to catch inconsistencies in how other libraries use these structures.
