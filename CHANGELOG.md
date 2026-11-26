# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted KZSalesHub MCP server, a published knowledge base of B2B and IT sales, marketing, operations and AI material for Kazakhstan and Central Asia, written in Russian. All six captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `search_knowledge`, `get_article`, `search_playbook`, `search_cases`, `get_related_content`, `list_categories`. The whole surface is read-only published-content search and fetch: no sends, no submissions, no personal data.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `steppesales error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `KZSALESHUB_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
