# BUILD REPORT — steppesales

Clean-room rebuild of `com.kzsaleshub/knowledge` v1.0.0
(remote-only MCP server at `https://kzsaleshub.com/mcp`),
built 2026-10-28 as a forwarding client. Package: `@reneturcios/steppesales`
0.1.0, registry name `io.github.CloudCorpRecords/steppesales`.

## Surface built

FULL surface: **all 6 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `search_knowledge` | `query` |
| `get_article` | `slug` |
| `search_playbook` | `query` |
| `search_cases` | `query` |
| `get_related_content` | `slug` |
| `list_categories` | *(none — no `required` key in the capture, served verbatim)* |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured (the capture's
per-tool `annotations` key was dropped at fixture slimming and is
not served). No inputSchema carries a `$schema` marker — none was
captured — and no tool carries `additionalProperties`. Property
shapes as captured: `search_knowledge` takes `query` (string),
`type` (string enum of the 10 captured content kinds) and `limit`
(integer, minimum 1, maximum 20, default 8); `get_article` takes
`slug` and `format` (enum `markdown` / `text`, default
`markdown`); `search_playbook` and `search_cases` take `query`
plus the same bounded `limit`; `get_related_content` takes `slug`
plus `limit`; `list_categories` carries empty `properties` and no
`required` key. All titles and descriptions are the captured
Russian text, served verbatim. The tool literals were generated
programmatically from
`harness/fixtures/kzsaleshub-tools-original.json` by the script
at `/tmp/laneb7-steppesales-scratch/gen.py` (parse +
re-serialise, spliced into `src/forward.ts` between the
TOOL_DEFS_GENERATED markers) — never retyped — and the
generator's own round-trip check confirmed the spliced block
deep-equals the fixture's 6 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers),
not McpServer+zod.

The whole surface is read-only search and fetch over one
published knowledge base (B2B/IT sales, marketing, operations
and AI material for Kazakhstan and Central Asia, in Russian).
No tool sends, submits, or carries personal data, so nothing
on the captured surface needed omitting.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, the `required` arrays (`query` / `slug` /
`query` / `query` / `slug`, and none for `list_categories`),
and the property shapes, including the `search_knowledge`
`type` enum, the bounded `limit` integers, and the
`get_article` `format` enum and default.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://kzsaleshub.com/mcp`.
- Env override `KZSALESHUB_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that
  no `Authorization` header (and no `X-API-Key` header) is sent
  with an empty environment, with a `KZSALESHUB_API_KEY`-style
  variable present, and with `AUTHORIZATION` / `API_KEY` /
  `KZSALESHUB_TOKEN` variables present.
- Error rendering `steppesales error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including `isError`
  and extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests,
  and no live call is claimed beyond the coordinator's
  tools/list capture.

## Deviations

**None.** The write-guard playbook was not needed: the
lockfile, `npm ci`, `tsc` and the test suite all ran normally
in this session via the direct `node_modules/.bin` binaries.

## Lockfile

Generated CLEAN Route 3 from birth, in a clean directory (no
`node_modules`, no pre-existing lockfile):
`npm install --package-lock-only --ignore-scripts --offline`
→ success. Verified: parses as JSON, `name` ==
`@reneturcios/steppesales`, exactly **128** package entries,
sha256 `20321a8b3cb7fba47034b0a9de8de6ca868f850cc0dc7116bba464b4c0590299`.
`npm ci --ignore-scripts --offline` then succeeded and the
lockfile sha256 was re-verified **unchanged** afterwards.

## server.json

- Description exactly: "Search a B2B sales and marketing knowledge base for Kazakhstan and Central Asia (in Russian)." — **93 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/steppesales:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/steppesales"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/steppesales`
  and its smoke test asserts the server name `steppesales`.

## Verification (run 2026-10-28, builder session)

1. `tsc --noEmit` (direct `node_modules/.bin/tsc`) → exit 0.
2. `tsc` build → exit 0; `dist/` produced normally.
3. `node dist/cli.js --version` → `0.1.0`.
4. Full test suite (`tsx --test test/*.test.ts`) → **69 tests,
   69 pass, 0 fail** (registration-integrity suite deep-equals
   the served tools/list against the fixture's 6 tools, in
   order, at both the registry level and through an
   in-memory-transport MCP client; plus per-tool schema shape
   checks — the `type` enum, bounded `limit` integers, the
   `format` enum/default, empty `list_categories` properties —
   session handling/reuse, env override, the
   no-Authorization-header assertions under every environment,
   verbatim argument forwarding for all 6 tools and result
   pass-through including upstream isError, SSE parsing, error
   rendering, MCP-layer validation, and CLI behaviour).
5. Parity (harness, tools-file mode vs
   `harness/fixtures/kzsaleshub-tools-original.json`, rebuild
   over stdio from
   `/home/hatch/workspace/mcp-rebuild-100/builds/steppesales/dist/cli.js`)
   → **verdict PASS, parity 100.0** (6/6 identical, 0 missing,
   0 extra, no schema diffs). Output in `parity-out/`.
6. Consistency sweep: `package.json` version = `server.json`
   top-level version = npm-entry version = OCI identifier tag
   = core `VERSION` = newest CHANGELOG heading (all `0.1.0`);
   `package.json` `mcpName` = `server.json` `name` = Dockerfile
   server-name label; all 19 factory files present.
