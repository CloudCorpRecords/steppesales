# steppesales

Selling into Kazakhstan and Central Asia has its own playbooks, vendors and hard-won lessons, and most of that knowledge sits in one published base: KZSalesHub. steppesales is a small forwarding client for the hosted KZSalesHub MCP service. The tool list is answered from your own machine using captured definitions, and each call travels upstream, with the service's answer handed back exactly as it arrived. The material itself is in Russian — B2B and IT sales, marketing, operations, startups and AI — and queries can be written in Russian or English, with morphology and synonyms taken into account upstream.

All six of the service's captured tools are exposed, and the whole surface is read-only: search and fetch over published content. Nothing here sends, submits or carries personal data.

## The tools

- **`search_knowledge`** — Search the entire public base with a free-text `query`: articles, systems (playbooks), frameworks, glossary terms, books, learning paths, courses and more. An optional `type` filters to one content kind, and `limit` caps the results (1–20, default 8). Each hit comes back with its title, original URL, type, category, date, relevance and an excerpt. The natural first call for any "what is known about X" question.
- **`get_article`** — One material in full: the current content in Markdown, with metadata and the URL to cite. Addressed by `slug` (a full page URL works too), with an optional `format` of `markdown` (default) or `text`.
- **`search_playbook`** — Search Sales OS and the other systems, training programmes and frameworks — step-by-step methodologies rather than individual articles. Takes a `query` and an optional `limit`.
- **`search_cases`** — Search the vendor, product and startup teardowns and the practical case studies inside the base's materials. Takes a `query` and an optional `limit`.
- **`get_related_content`** — What connects to a given material, addressed by `slug` (or URL): direct links out of its text and into it, shared topics, roles and technologies, grouped by type. Optional `limit`.
- **`list_categories`** — The shape of the base itself: topics, content types with counts, systems, roles, technologies and industries. Worth calling first to understand what the base holds before searching it. No arguments.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/steppesales mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "steppesales": {
      "command": "npx",
      "args": ["-y", "@reneturcios/steppesales", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/steppesales
docker run --rm ghcr.io/cloudcorprecords/steppesales search_knowledge '{"query":"prodazhi"}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
steppesales search_knowledge '{"query":"B2B prodazhi","type":"playbook"}'
steppesales get_article '{"slug":"nfr-licenziya-chto-mozhno"}'
steppesales search_playbook '{"query":"Sales OS"}'
steppesales search_cases '{"query":"vendor teardown"}'
steppesales get_related_content '{"slug":"nfr-licenziya-chto-mozhno"}'
steppesales list_categories '{}'
steppesales --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `KZSALESHUB_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://kzsaleshub.com/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The KZSalesHub registry record lists no headers for its remote, so steppesales never attaches an authentication header — every call goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `steppesales error:`.

## How forwarding works

The first call of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, steppesales repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The suite compares the served tool list against the captured surface tool for tool, walks the session handshake and its reuse, proves no `Authorization` header is sent under any environment, and confirms that arguments travel upstream unchanged and results return unchanged for each of the six tools.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
