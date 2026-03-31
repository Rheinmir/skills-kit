---
name: mcp-research
description: MCP researcher — fetch docs/API/spec from GitHub, DB, internal tools via MCP
allowed-tools: WebFetch, WebSearch, Bash
---

Query: $ARGUMENTS

**Goal: answer the query with minimum fetches. Targeted reads only — no broad crawling.**

**Sequence:**

1. **PARSE** — Extract from $ARGUMENTS:
   - Target source type: `github` | `url` | `mcp-tool` | `search`
   - Specific artifact: endpoint, schema, function, config key, page section
   - If source is ambiguous → ask before fetching

2. **LOCATE** — One targeted lookup to find the exact resource:
   - GitHub repo: fetch `https://api.github.com/repos/<owner>/<repo>/contents/<path>` or raw file URL
   - API spec: fetch OpenAPI/AsyncAPI root, then only the relevant path object
   - DB schema: use available MCP tool (e.g. list-tables → describe only relevant tables)
   - Unknown source: one WebSearch with `site:` scoped query, pick top result

3. **FETCH** — Read only what answers the query:
   - For large docs: fetch the section anchor/fragment if URL supports it
   - For OpenAPI specs: extract only the relevant `paths` + `components/schemas` entries
   - For GitHub files: prefer raw URL over HTML — less noise, fewer tokens
   - Stop after 2 fetches if answer is found

4. **SYNTHESIZE** — Output:
```
SOURCE:  <url or tool:resource>
FINDING: <direct answer to the query — facts, not narrative>
SCHEMA:  <relevant fields/types if applicable>
NOTES:   <caveats, version constraints, gotchas>
```

**Hard rules:**
- Never fetch a page just to find where the real page is — resolve first, then fetch
- No recursive crawling — if a link looks relevant, include it in NOTES, don't follow it
- Skip auth/login pages — note the access requirement and stop
- If MCP tool is available for the source (DB, internal API), prefer it over web fetch
- Max 3 fetches per query — if not resolved: report what was found + what's still missing
