# MCP and REST

MCP and REST expose NotebookLM functionality through different transport contracts.

## MCP
MCP exposes named tools with structured arguments and results for agent clients. It is an optional/experimental surface; tool names and schemas can evolve.

Consult the inherited [MCP guide](../mcp-guide.md).

## REST
REST exposes HTTP routes, request/response bodies, authentication and HTTP status semantics. Treat it as a separate adapter, not as a direct mirror of every Python method. Consult the inherited server implementation and [REST/server reference](../server-api.md) where present.

## Mapping principle
Both adapters should normalize inputs and delegate to shared application/client operations. Transport-specific authentication, validation and response projection remain at the boundary.

## Security
Never expose session cookies, bearer tokens or raw authorization headers in tool results, logs, examples or generated documentation.
