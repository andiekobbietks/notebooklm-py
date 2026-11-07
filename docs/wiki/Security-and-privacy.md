# Security and privacy

- Treat NotebookLM session credentials and tokens as secrets.
- Never write cookies, bearer tokens, refresh tokens or authorization headers to archives or logs.
- Keep notebook content local by default; any external transmission must be explicit and documented.
- Use synthetic/sanitised fixtures in tests and documentation.
- Make AgentSON/event integrations opt-in and redact content by default.
- Do not bypass Google authentication, authorization, quotas or access controls.
- Report security concerns privately using the repository's supported security reporting channel, if configured.
