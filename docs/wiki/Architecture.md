# Architecture

See the [architecture and data-model document](../fork/architecture-and-data-models.md) for Mermaid diagrams and the detailed conceptual model.

## High-level boundaries

- **CLI:** parses user intent, resolves profile/notebook context and renders results.
- **Python API:** primary typed library interface, centred on `NotebookLMClient` and its feature namespaces.
- **MCP:** agent-facing tool adapter; optional/experimental.
- **REST:** HTTP adapter; optional/experimental.
- **Application layer:** transport-neutral operations, plans, polling/waiting, retries and error classification.
- **Backends:** distinct Web and Android transport implementations.

Adapters should not duplicate backend RPC construction. New archive functionality should consume supported client operations rather than fork transport internals.

## State and data models

The architecture document includes:
- system component diagram;
- CLI invocation and context-resolution state diagram;
- Python client conceptual class diagram;
- MCP/REST mapping diagram;
- NotebookLM resource ER model;
- source-processing and generated-artifact state diagrams;
- proposed archive/provenance flow.
