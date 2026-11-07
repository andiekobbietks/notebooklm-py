# Architecture, state diagrams and data models

**Status:** Current-state model based on the fork's inherited code and reference documentation, with proposed archive extensions marked separately.

This document is a companion to [the upstream architecture guide](../architecture.md), [CLI reference](../cli-reference.md), [Python API reference](../python-api.md), [MCP guide](../mcp-guide.md) and [diagram catalogue](../diagrams/README.md). It does not replace those detailed references.

## 1. Runtime boundaries

The current architecture separates frontends from transport-neutral application operations and backend-specific transport implementations:

```mermaid
flowchart TB
    subgraph Frontends["Frontend adapters"]
      CLI["CLI · Click / Rich"]
      PY["Python API"]
      MCP["MCP · FastMCP"]
      REST["REST · FastAPI"]
    end
    APP["Transport-neutral application layer · _app/"]
    CLIENT["NotebookLMClient · typed feature namespaces"]
    RUNTIME["Client-owned neutral runtime"]
    SELECT{"Backend selected at construction"}
    WEB["Web backend · HTTP / batchexecute"]
    AND["Android backend · protobuf / gRPC"]
    GOOGLE["NotebookLM service"]
    CLI --> APP
    MCP --> APP
    REST --> APP
    PY --> CLIENT
    APP --> CLIENT
    CLIENT --> RUNTIME
    RUNTIME --> SELECT
    SELECT --> WEB
    SELECT --> AND
    WEB --> GOOGLE
    AND --> GOOGLE
```

**Boundary rules from the inherited architecture:**
- CLI, MCP and REST are adapters; they should not own raw RPC payload construction.
- The `_app/` layer owns transport-neutral plans, resolution, waits, retries, status projection and error classification.
- `NotebookLMClient` exposes typed feature namespaces and public data contracts.
- Backend choice is made at construction. Web and Android are separate transport assemblies.
- MCP and REST are optional/experimental extras; their public surface is not covered by the same compatibility guarantees as the core Python API.

## 2. CLI data model

The CLI is a command grammar and presentation adapter. It resolves command-line input and context, invokes shared operations, then renders human or JSON output.

```mermaid
flowchart LR
    ARGV["argv + environment"] --> GLOBAL["Global options"]
    GLOBAL --> PROFILE["Profile / storage / backend resolution"]
    PROFILE --> GROUP["Click command group"]
    GROUP --> PARSE["Typed command arguments"]
    PARSE --> CTX["CLI runtime / command context"]
    CTX --> PLAN["Application request / plan"]
    PLAN --> SERVICE["Client namespace / service"]
    SERVICE --> RESULT["Typed result or domain exception"]
    RESULT --> RENDER{"Output mode"}
    RENDER --> HUMAN["Rich / human-readable output"]
    RENDER --> JSON["Structured JSON envelope"]
    RESULT --> EXIT["Exit-code mapping"]
```

### CLI input and output entities

| Entity | Meaning | Examples / notes |
|---|---|---|
| Invocation | One process execution | `notebooklm [GLOBAL OPTIONS] COMMAND` |
| Global options | Cross-command configuration | `--profile`, `--storage`, `--backend`, `--json`, verbosity |
| Profile | Named credential/configuration context | Default profile or named profile |
| Notebook context | Target notebook resolution | Explicit `-n`, `NOTEBOOKLM_NOTEBOOK`, active context |
| Command | User-intent operation | `source add`, `ask`, `generate audio`, `download report` |
| Request / plan | Transport-neutral operation input | Validated identifiers and operation parameters |
| Result | Typed service outcome | Notebook, source, answer, artifact status, path, etc. |
| Error envelope | Stable machine-readable failure category | JSON `code`, message and relevant details |
| Process status | Shell control-flow signal | 0, 1, 2, 130; command-specific documented exceptions |

### Context resolution

```mermaid
stateDiagram-v2
    [*] --> ParseInvocation
    ParseInvocation --> ResolveProfile
    ResolveProfile --> LoadCredentials
    LoadCredentials --> ResolveNotebook: command needs notebook
    LoadCredentials --> Ready: command is profile/global
    ResolveNotebook --> ExplicitNotebook: -n / --notebook supplied
    ResolveNotebook --> EnvNotebook: NOTEBOOKLM_NOTEBOOK set
    ResolveNotebook --> ActiveContext: active notebook context exists
    ResolveNotebook --> MissingContext: none available
    ExplicitNotebook --> Ready
    EnvNotebook --> Ready
    ActiveContext --> Ready
    MissingContext --> ValidationError
    Ready --> Execute
    Execute --> Success
    Execute --> DomainError
    Execute --> Interrupted
    Success --> [*]
    DomainError --> [*]
    ValidationError --> [*]
    Interrupted --> [*]
```

The exact option precedence and command-specific exceptions remain governed by the CLI reference and implementation. Do not treat this high-level diagram as a substitute for those rules.

## 3. Python API data model

The primary public entry point is `NotebookLMClient`. It owns typed namespaces, configuration, lifecycle and backend assembly.

```mermaid
classDiagram
    class NotebookLMClient {
      +from_storage(...)
      +notebooks
      +sources
      +chat
      +artifacts
      +notes
      +labels
      +collections
      +research
      +sharing
      +settings
      +close()
    }
    class ClientConfig {
      +backend
      +runtime
      +transfers
    }
    class RuntimeOptions
    class TransferOptions
    class Notebook
    class Source
    class Note
    class Artifact
    class AskResult
    class NotebookNamespace
    class SourceNamespace
    class ChatNamespace
    class ArtifactNamespace
    NotebookLMClient --> ClientConfig : configured by
    NotebookLMClient --> NotebookNamespace : exposes
    NotebookLMClient --> SourceNamespace : exposes
    NotebookLMClient --> ChatNamespace : exposes
    NotebookLMClient --> ArtifactNamespace : exposes
    NotebookNamespace ..> Notebook : returns
    SourceNamespace ..> Source : returns
    ChatNamespace ..> AskResult : returns
    ArtifactNamespace ..> Artifact : returns
    ClientConfig --> RuntimeOptions
    ClientConfig --> TransferOptions
```

The class diagram is intentionally conceptual: namespace names and resource types are grounded in the public package, but method signatures and all fields belong in the versioned API reference and typed source definitions.

## 4. MCP and REST mapping

```mermaid
flowchart TB
    USER["Agent / API consumer"]
    subgraph Adapters["Transport adapters"]
      M["MCP tool call"]
      H["REST request"]
      C["CLI command"]
    end
    VALIDATE["Transport-specific validation and authentication"]
    NORMALIZE["Typed input / request normalization"]
    CORE["Shared _app/ operation"]
    NS["NotebookLMClient namespace"]
    OUT["Typed result / domain error"]
    PROJECT["Adapter-specific projection"]
    USER --> M
    USER --> H
    USER --> C
    M --> VALIDATE
    H --> VALIDATE
    C --> NORMALIZE
    VALIDATE --> NORMALIZE
    NORMALIZE --> CORE
    CORE --> NS
    NS --> OUT
    OUT --> PROJECT
    PROJECT --> USER
```

| Surface | Contract shape | Status / boundary |
|---|---|---|
| Python API | Async methods, typed options, dataclasses and exceptions | Primary library integration surface |
| CLI | Command grammar, human output, optional structured JSON and process exit codes | Shell/automation adapter |
| MCP | Tool names, JSON-like arguments and tool results | Optional extra; explicitly experimental |
| REST | HTTP methods/routes, JSON bodies, bearer authentication and HTTP status | Optional extra; explicitly experimental |

The MCP guide currently documents 38 tools. Treat that count as a versioned snapshot, not a permanent contract. REST's OpenAPI/schema endpoints are disabled by default in the inherited server implementation; consult the route modules and server security boundary before relying on endpoint discovery.

## 5. Resource and state models

```mermaid
erDiagram
    NOTEBOOK ||--o{ SOURCE : contains
    NOTEBOOK ||--o{ NOTE : contains
    NOTEBOOK ||--o{ CONVERSATION : has
    CONVERSATION ||--o{ CHAT_TURN : contains
    NOTEBOOK ||--o{ ARTIFACT : produces
    SOURCE ||--o{ SOURCE_DERIVATION : may_yield
    NOTEBOOK ||--o{ RESEARCH_TASK : runs
    NOTEBOOK {
      string notebook_id PK
      string title
      string description
    }
    SOURCE {
      string source_id PK
      string notebook_id FK
      string source_type
      string display_name
      string processing_status
    }
    NOTE {
      string note_id PK
      string notebook_id FK
      string title
      string content
    }
    CONVERSATION {
      string conversation_id PK
      string notebook_id FK
    }
    CHAT_TURN {
      string turn_id PK
      string conversation_id FK
      string role
      datetime created_at
    }
    ARTIFACT {
      string artifact_id PK
      string notebook_id FK
      string artifact_type
      string generation_status
    }
    SOURCE_DERIVATION {
      string derivation_id PK
      string source_id FK
      string derivation_type
    }
    RESEARCH_TASK {
      string task_id PK
      string notebook_id FK
      string status
    }
```

This is a conceptual domain model, not a claim that every relationship is persisted by Google exactly as drawn. IDs, field availability and resource lifecycles are backend/service-dependent.

## 6. State diagrams for resource operations

### Source processing

```mermaid
stateDiagram-v2
    [*] --> Submitted
    Submitted --> Accepted
    Submitted --> Rejected
    Accepted --> Processing
    Processing --> Ready
    Processing --> Failed
    Processing --> Unknown
    Ready --> [*]
    Rejected --> [*]
    Failed --> [*]
    Unknown --> [*]
```

### Generated artifact

```mermaid
stateDiagram-v2
    [*] --> Requested
    Requested --> Queued
    Requested --> Rejected
    Queued --> Generating
    Generating --> Completed
    Generating --> Failed
    Generating --> TimedOut
    TimedOut --> Reconciliating
    Reconciliating --> Completed: confirmed complete
    Reconciliating --> Failed: confirmed failure
    Reconciliating --> Unknown: outcome cannot be confirmed
    Completed --> Downloadable
    Downloadable --> Downloaded
    Downloaded --> Verified
    Verified --> [*]
    Failed --> [*]
    Rejected --> [*]
    Unknown --> [*]
```

The `Unknown` and reconciliation states are important for non-idempotent operations: a timeout does not necessarily mean the remote operation did not happen. Consult the inherited operation-lifetime and unconfirmed-write contracts before implementing retries.

## 7. Proposed archive extension boundary

The archive/provenance layer should be an optional consumer of the existing client rather than a rewrite of its transport internals:

```mermaid
flowchart LR
    NLM["NotebookLMClient / supported APIs"] --> COLLECT["Resource collectors"]
    COLLECT --> CLASSIFY["Capability and outcome classifier"]
    CLASSIFY --> NORMALIZE["Archive resource record"]
    NORMALIZE --> WRITE["Atomic archive writer"]
    WRITE --> HASH["SHA-256 over exported bytes"]
    HASH --> MANIFEST["Versioned manifest"]
    MANIFEST --> AUDIT["Completeness / integrity verifier"]
    AUDIT --> REPORT["Human + JSON report"]
    MANIFEST --> EVENTS["Optional AgentSON JSONL adapter"]
```

The extension must not claim original-file recovery when only indexed text is available. Detailed proposed fields and status semantics are in [archive and provenance model](./archive-provenance-model.md).
