# CLI and Python API

The CLI and Python API are related but distinct public surfaces.

## CLI
The CLI is a shell-facing command grammar. Its contract includes global options, command groups, notebook/profile context resolution, human and JSON output, error reporting and process exit codes.

Consult the inherited [CLI reference](../cli-reference.md). The architecture document contains the invocation and context-resolution diagrams.

## Python API
The Python API is the primary typed integration surface. `NotebookLMClient` owns configuration, backend selection, lifecycle and feature namespaces. Resource objects and exceptions are defined by the implementation and should be documented from their actual source definitions.

Consult the inherited [Python API reference](../python-api.md).

## Contract discipline
- Verify signatures against the current code before documenting them.
- Distinguish public API from internal implementation details.
- Record async behaviour, exceptions and lifecycle requirements.
- Mark unstable or backend-specific behaviour.
- Treat diagrams as conceptual, not substitutes for exact signatures.
