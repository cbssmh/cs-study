1. Title

Interactive Refactoring in gopls

2. Source

Author / Organization: Hongxiang Jiang / The Go Project (The Go Blog)
Link: https://go.dev/blog/interactive-refactoring
Date: 2026-10-16

3. One-line Summary

The Go team introduced interactive refactoring in gopls and proposed an open Language Server Protocol (LSP) extension (co-designed with the Dart team) that enables language servers to drive dynamic, multi-step parameter collection dialogs across diverse editors without breaking editor-agnostic decoupling.

4. Key Points

• The Parameterless LSP Bottleneck: Historically, nearly all code transformations in the Language Server Protocol functioned as zero-parameter calls where the server had to infer user intent solely from cursor position and enclosing syntax trees; single-parameter symbol renaming was the lone exception requiring a bespoke `PrepareRename` protocol extension.
• The Binary Compromise in Tooling: When syntactic context was insufficient to determine transformation intent, language servers were forced into an unsatisfactory compromise: apply a rigid, one-size-fits-all default (e.g., unconditionally generating camelCase JSON struct tags) or withhold the refactoring feature entirely.
• Rejection of Editor-Specific Workarounds: While individual extensions previously hacked custom dialogs directly into VS Code, doing so violated the fundamental LSP premise by creating fragmented, non-portable capabilities that left terminal editors like Neovim and Emacs behind.
• Open Cross-Language Standard: In collaboration with the Dart/Flutter team, Go engineers formalized an open standard (microsoft/language-server-protocol#1164) defining a stateless, server-driven form protocol for collecting user parameters before computing workspace text edits.
• Dynamic Struct Tag Customization: Upgrades struct tag manipulation in gopls v0.23.0 to allow developers to select arbitrary tag combinations (e.g., `json,yaml,db`), pick naming conventions (camelCase, snake_case, PascalCase), or selectively delete specific tag keys while preserving others.
• Context-Aware Interface Implementation: Enables developers to implement interfaces on concrete types via dynamic workspace and standard-library symbol search (auto-completing interface names like `maphash.Hasher`) and recursively prompts for concrete type arguments when implementing generic interfaces.
• Automated Cross-Package Declaration Relocation: Introduces a preview refactoring that moves top-level declarations across files or packages via an interactive file picker, automatically moving private dependent helpers, updating all workspace references, and cleaning up unused imports in both source and destination packages.
• Self-Contained Test Harness ("Slay the Dragon"): Gopls v0.24.0 includes a playful, multi-step interactive code action triggered on variables named `dragonSlayer`, serving as an end-to-end verification harness for third-party editor client implementations.
• Expanding to Non-Edit Server Commands: Because interactivity is abstracted at the generic LSP command layer, the same protocol foundation will power future complex operations such as "Change Signature" parameter reordering, interactive identifier conflict resolution, and configuring benchmark/test runs with profiling flags.

5. Deep Dive (Structured Understanding)

Problem
The Language Server Protocol (LSP) revolutionized developer tooling by decoupling language intelligence from editor frontends through standard JSON-RPC. However, code actions were designed with a critical structural limitation: they are parameterless functions. When a user invokes a code action, the server receives only the document URI and cursor range. While sufficient for simple fixes (e.g., adding a missing import), complex architectural refactorings inherently require user-supplied parameters: what naming convention to apply to struct tags, which interface to implement, what type arguments to bind to generics, or what destination package a function should be moved to. Because the protocol lacked a standard mechanism for a server to prompt the user, language servers either forced rigid defaults or relied on bespoke, proprietary editor plugins that degraded the experience on non-VS Code editors like Neovim and Emacs.

Approach
The Go team (collaborating with the Dart team and the LSP community) designed an interactive code action extension for the LSP (microsoft/language-server-protocol#1164). Rather than having the editor hardcode language-specific dialogs, the server returns an interactive command descriptor defining a sequence of input steps (e.g., text inputs, dropdown selections, symbol pickers). The client editor renders native dialogs matching its platform UI and returns the captured parameters to the server. Gopls then executes the transformation as a standard workspace text edit. The implementation debuted in gopls v0.23.0/v0.24.0 and vscode-go v0.56.0, with proof-of-concept integrations demonstrated for terminal editors.

Key Insight
1. Server-Driven Protocol Independence: True editor-agnostic tooling requires that UI forms and validation logic reside in the language server, while the editor acts strictly as an unopinionated rendering agent. This preserves the "1 × M" efficiency of the LSP: a single refactoring feature implemented in gopls immediately functions across VS Code, Neovim, Emacs, and Helix.
2. AST Dependency Graphing for Code Movement: Moving declarations across package boundaries is fundamentally more complex than text relocation. Gopls performs reachability analysis on the abstract syntax tree (AST) to identify unexported helper functions, types, and constants tightly coupled to the moved symbol, relocating them together while rewriting package references and imports across the entire workspace.
3. Generalization Beyond Code Edits: Modeling interactivity as a general command extension rather than a text-edit hack allows the protocol to serve runtime configuration tasks—such as interactively selecting CPU/memory profiling flags or debugger attachment parameters directly from the language server.

Result / Impact
Unlocks advanced, multi-step refactoring capabilities (Move Declaration, Implement Generic Interface, Change Signature) for Go developers across all LSP-compliant editors, establishing a cross-industry open standard that modernizes code manipulation across multiple programming language ecosystems.

6. Why It Matters

• Platform Engineering & Developer Experience: Addresses the long-standing usability gap between monolithic, proprietary IDEs (like JetBrains IntelliJ/GoLand) and modular open-source LSP editors, bringing rich, interactive refactorings to terminal-centric developer workflows.
• Open Standards & Tooling Architecture: Demonstrates how open-source language maintainers (Go and Dart) can collaboratively push universal protocol standards (LSP) forward to eliminate proprietary extension fragmentation.
• Software Engineering & Maintainability: Reduces the friction and human error associated with large-scale architectural refactoring—such as moving domain models into separate packages or modifying struct serialization tags across large codebases.
• AI Engineering & Coding Agents: Provides standardized, machine-verifiable parameter schemas that AI coding agents and automated refactoring harnesses can inspect and invoke programmatically to perform safe, syntax-aware code modifications.

7. Critical Analysis

• Client Adoption and Ecosystem Lag: While gopls and `vscode-go` support interactive refactorings today, widespread developer benefit depends on upstream acceptance into the core Microsoft LSP specification and subsequent implementation across Neovim, Emacs (eglot/lsp-mode), Sublime Text, and Helix. Until client implementations mature, non-VS Code users may see disabled or degraded code actions.
• Network Latency and Turn-Around in Remote Workspaces: For remote development environments (SSH remotes, devcontainers, cloud IDEs), multi-step interactive refactorings introduce multiple network round-trips for dynamic symbol auto-completion, which could introduce noticeable UI stutter on high-latency connections.
• Edge Cases in Complex AST Moves: Moving declarations that participate in cyclic dependencies, complex build tags (`//go:build`), or architecture-specific assembly files requires sophisticated static analysis. In large, legacy enterprise codebases, automated dependency extraction risks breaking subtle implicit package invariants that compiler error checkers must catch post-refactor.

8. Connections

• Language Server Protocol (LSP) Specification: Directly extends the JSON-RPC foundation established by Microsoft, addressing the protocol's inability to handle parameterized code actions beyond `textDocument/prepareRename`.
• JetBrains Refactoring Engine: Replicates the sophisticated, interactive refactoring dialogs historically exclusive to proprietary IDEs (IntelliJ, GoLand) within an open, decoupled architecture.
• Go Abstract Syntax Tree (AST) & `golang.org/x/tools/go/analysis`: Leverages Go's robust standard tooling infrastructure to parse, inspect, type-check, and rewrite source files with mathematical precision.

9. Keywords

• gopls
• Language Server Protocol (LSP)
• Interactive Refactoring
• AST Transformation
• Struct Tags
• Interface Implementation
• Move Declaration
• Developer Tooling

10. TL;DR

• The Go team introduced interactive refactoring in gopls, powered by a new open standard proposed for the Language Server Protocol in collaboration with Dart.
• Bridges a major LSP architectural gap by enabling language servers to prompt users for multi-step parameters (naming conventions, target packages, generic type arguments) before executing workspace text edits.
• Shipped in gopls v0.24.0, unlocking long-awaited capabilities like dynamic struct tag customization, interface method generation, and cross-package declaration moves across all supported editors.
