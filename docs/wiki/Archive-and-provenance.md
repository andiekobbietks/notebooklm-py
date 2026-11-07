# Archive and provenance

**Status: proposed design, not a statement that full archive extraction is implemented.**

The proposed archive layer records what the current NotebookLM integration can actually retrieve and preserves the limits of that retrieval.

## Keep resource types separate
- original uploaded file bytes;
- NotebookLM-indexed or extracted text;
- source metadata;
- notes;
- conversations and chat turns;
- research task results;
- generated Studio artefacts.

Indexed text is not a byte-identical replacement for the original upload.

## Status vocabulary
`available`, `unsupported`, `not_authorized`, `not_found`, `failed`, `not_tested`, `unknown`.

An empty response or timeout is not proof that a resource does not exist.

## Integrity
The proposed manifest records resource identity/type, retrieval outcome/time, exporter/upstream version, media type, path, byte length and SHA-256 over the exact exported bytes.

A checksum verifies bytes relative to a manifest. It does not prove ownership, authorship, consent or licensing.

See [the full archive/provenance model](../fork/archive-provenance-model.md).
