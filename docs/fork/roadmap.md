# Fork roadmap

This roadmap captures the proposed work for the AndieKobbieTech fork. It deliberately separates documentation and maintenance foundations from features that require further reverse-engineering, implementation and evidence.

> **Issue tracker status:** GitHub Issues was disabled when this roadmap was prepared. These entries are issue-ready specifications, not claims that GitHub Issues have already been created. Enable Issues in repository settings, then create one issue per section below.

## Principles

- Preserve and track upstream rather than replacing it.
- Do not claim an export is complete unless the service inventory and export inventory can be compared with adequate evidence.
- Distinguish original uploaded bytes, indexed/extracted text, notes, conversations, metadata and generated artefacts.
- Record uncertainty explicitly. An inaccessible or unobservable resource is not proof of absence.
- Keep provenance, audit and AgentSON integrations optional and privacy-conscious.
- Make every architectural extension reviewable, tested and documented.

## Work packages

### R1 — Verified archive/export contract
**Priority:** P0 · **Type:** Research / design

Inventory the NotebookLM resource classes and the exact retrieval/export capability available for each. Define resumable exports and a resource-level status vocabulary: `available`, `unsupported`, `not_authorized`, `not_found`, `failed`, `not_tested`.

**Acceptance:** evidence-backed capability matrix; explicit distinction between original files and indexed text; tests for partial failure, empty results, pagination and resume; no promise to retrieve original bytes unless the service returns them.

### R2 — Provenance manifest and integrity verification
**Priority:** P0 · **Type:** Feature

Define a versioned manifest for each archive and resource. Include retrieval time, exporter version, resource identity/type, source reference, outcome, media type, relative path, byte length and SHA-256 of exact exported bytes.

**Acceptance:** deterministic serialization; verifier reports missing/changed/unexpected/unverifiable files; no secrets in the archive; corruption and truncation tests.

### R3 — Completeness and capability audit
**Priority:** P1 · **Type:** Feature

Compare available service inventories with archived resources and produce human-readable and machine-readable reports. Preserve unknown states and explain every omission.

**Acceptance:** reproducible report derived from manifest; machine-readable output; documented CI exit semantics; mixed-success and pagination tests.

### R4 — CLI, Python API, MCP and REST contracts
**Priority:** P0 · **Type:** Documentation / architecture

Maintain a model of the existing CLI grammar, Python client namespaces, REST routes and MCP tools, including their mapping to shared application operations. Document state transitions and resource entities.

**Acceptance:** diagrams match current code; experimental interfaces are labelled; future archive/provenance operations are clearly marked as proposals.

### R5 — Optional AgentSON event adapter
**Priority:** P2 · **Type:** Design / integration

Explore append-only typed JSONL events for archive and provenance operations. Keep AgentSON outside the core dependency graph.

**Acceptance:** event schema, redaction and privacy policy, ordering/partial-failure contract tests, schema versioning and opt-in integration.

### R6 — Upstream sync and divergence policy
**Priority:** P0 · **Type:** Maintenance

Fetch upstream on a schedule and open/update a PR against fork `main`. Never auto-merge. Document conflict handling and upstream commit provenance.

**Acceptance:** least-privilege workflow; no direct main push; PR includes upstream commit range; conflicts require a human.

### R7 — Sanitized regression fixtures
**Priority:** P1 · **Type:** Testing

Add synthetic or sanitized fixtures for source metadata, indexed text, notes, conversations, artifact listings/downloads, pagination and failure modes.

**Acceptance:** no credentials or personal data; deterministic tests; live authenticated E2E kept separate.

### R8 — Portable archive layout and restore feasibility
**Priority:** P1 · **Type:** Design / feature

Specify a collision-safe filesystem layout and distinguish restorable, re-importable, metadata-only and non-restorable resource types.

**Acceptance:** versioned layout, examples, path traversal/symlink protection, streaming/atomic writes and round-trip tests for supported resource types.

## Suggested delivery order

1. R4 documentation and data-model foundation.
2. R6 upstream maintenance automation.
3. R1 capability research and archive contract.
4. R2 manifest and integrity verifier.
5. R3 completeness audit.
6. R7 regression fixtures.
7. R8 portable layout and restore feasibility.
8. R5 optional AgentSON adapter.

## Out of scope without separate evidence and approval

- Circumventing Google authentication, authorization, quotas or access controls.
- Claiming access to every NotebookLM internal object.
- Treating indexed text as a byte-identical copy of the uploaded original.
- Silently retrying non-idempotent mutations or presenting uncertain writes as confirmed.
- Sending notebook content or credentials to third-party telemetry.
