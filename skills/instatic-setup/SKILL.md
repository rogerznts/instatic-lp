---
name: instatic-setup
description: Connect an AI client to an Instatic instance over MCP and troubleshoot the connection. Use when the user wants to set up, connect, configure or debug the Instatic MCP (mcp__instatic__* tools missing, fewer than the 53 expected tools, only 2 tools listed, "0 capabilities", "open the workspace" errors, 401s), write an `mcpServers` / mcp-remote config for it, create an Instatic access token, or asks "conectar o instatic", "configurar o MCP do instatic", "quais ferramentas do instatic estão disponíveis".
---

# Instatic setup

Instatic (https://github.com/CoreBunch/Instatic) is a self-hosted visual CMS that is itself an MCP server. Every instance exposes one endpoint:

```text
https://<host>/_instatic/mcp
```

## 1. Check whether it is already connected

Look for tools named `mcp__instatic__*` (or `mcp__<whatever-name>__get_context`). If they exist, call `get_context` — it is headless and reports whether the Site editor and Content workspace are connected. If it works, check the tool count (step 4) and skip to step 5.

## 2. Pick a connection mode

| Client | Mode | How |
|---|---|---|
| Claude Code, Codex, Cursor, local bridges | Personal access token (`imcp_pat_…`) | Instatic → **AI → MCP → Create access token**. Choose name, expiry and capabilities, pass step-up. The token is shown **once**. |
| Claude Desktop / claude.ai custom connector | Hosted OAuth | Paste the Remote MCP URL in **Settings → Connectors**, leave client id/secret empty, approve in Instatic. Requires a public HTTPS origin. |

Capabilities the token needs (grant only what the task needs):

- Reading: `site.read`, `media.read`
- Editing pages: `ai.tools.write` + `site.structure.edit` (HTML, pages, templates, code assets), `site.style.edit` (CSS, classes, tokens), `site.content.edit` (copy/props only)
- Media upload: `ai.tools.write` + `media.write`
- Publishing: `ai.tools.write` + `pages.publish`
- CMS content: `content.create`, `content.edit.own` / `content.edit.any`, `content.publish.own` / `content.publish.any`

## 3. Register the server

Never paste the token into a file that gets committed. Give the user the command/config with their host and let them insert the token themselves.

### Claude Code (native HTTP)

Suggest the `!` prefix so it runs in-session:

```sh
claude mcp add instatic --transport http --scope user \
  https://<host>/_instatic/mcp \
  --header "Authorization: Bearer imcp_pat_…"
```

- `--scope user` makes it available in every project; `--scope local` (default) only in the current directory. Use `--scope project` only if the team shares an instance and each member supplies their own token via `${INSTATIC_TOKEN}` in `.mcp.json`.

### JSON `mcpServers` clients (Claude Desktop, Cursor, Windsurf, any stdio-only client)

Bridge with `mcp-remote`, which turns the remote HTTP endpoint into a local stdio server:

```json
"mcpServers": {
  "instatic": {
    "command": "npx",
    "args": [
      "-y",
      "mcp-remote@latest",
      "https://<host>/_instatic/mcp",
      "--transport",
      "http-only",
      "--header",
      "Authorization:${INSTATIC_MCP_AUTH}"
    ],
    "env": {
      "INSTATIC_MCP_AUTH": "Bearer imcp_pat_…"
    }
  }
}
```

- `--transport http-only`: Instatic speaks Streamable HTTP; skips the SSE fallback probe.
- `Authorization:${INSTATIC_MCP_AUTH}` has **no space** on purpose: some clients split args on spaces. `mcp-remote` expands the variable, so the `Bearer ` prefix and its space live in `env`.
- Needs Node/`npx` on the machine.

### Codex

`~/.codex/config.toml`, same bridge:

```toml
[mcp_servers.instatic]
command = "npx"
args = ["-y", "mcp-remote@latest", "https://<host>/_instatic/mcp", "--transport", "http-only", "--header", "Authorization:${INSTATIC_MCP_AUTH}"]
env = { INSTATIC_MCP_AUTH = "Bearer imcp_pat_…" }
```

The tool list is built on connect: after adding the server or changing the token's capabilities, restart the session / reconnect (`/mcp` in Claude Code).

## 4. Check tool availability

With **every** capability granted the server exposes **53 tools**:

- **Context and media (2):** `get_context`, `media_upload`.
- **Content (15):** reads `content_list_collections`, `content_get_collection_schema`, `content_list_documents`, `content_get_document`, `content_search_documents`, `content_list_media`, `content_list_users`; writes `content_create_document`, `content_set_document_field`, `content_set_document_fields` (both only on the active document), `content_set_document_status` (draft / scheduled / published), `content_set_document_author`, `content_delete_document` (trash, restorable); navigation `content_set_active_collection`, `content_set_active_document`.
- **Site (36):** pages and templates, documents, nodes (HTML), classes and CSS, design tokens, code assets, modules and loops, visual inspection (`site_render_snapshot`, `site_list_breakpoints`) and `site_publish`. Site edits stay in draft until `site_publish`.

Fewer tools = narrower token (or another Instatic version). Map each missing tool to its capability with the `instatic-lp` catalog (`references/tools.md`), recreate the token with it, reconnect. Grant only what the task needs; 53 is the ceiling, not a requirement.

## 5. Understand the two execution modes

This is the most common source of confusion:

- **Headless tools** (16) work with no browser open: `get_context`, `site_list_documents`, `site_list_breakpoints`, `site_list_modules`, `site_list_post_types`, `site_list_loop_sources`, `site_read_styles`, the 7 content reads (`content_list_*`, `content_get_*`, `content_search_documents`), `media_upload`, `site_publish`.
- **Browser tools** (37: all page-tree, HTML/CSS, token, page lifecycle, content write/navigation and code-asset tools, plus `site_read_document` and `site_render_snapshot`) are relayed to the **connection owner's open editor**. The user must have the Instatic editor open in a browser, logged in as the same user that created the token — the **Site** workspace for site tools, the **Content** workspace for content tools. Otherwise the tool returns an "open the workspace" error.

When a browser tool fails with that error, ask the user to open `https://<host>/admin` in the right workspace and keep the tab open (a backgrounded tab is fine), then retry.

## 6. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| No `mcp__instatic__*` tools | Server not registered in this scope, or session not restarted | `claude mcp list`; re-add with the right `--scope`; reconnect |
| 401 Unauthorized | Token revoked/expired, or missing `Bearer ` prefix | Create a new token; check the header (with `mcp-remote`, the `Bearer ` prefix goes in the env var) |
| `mcp-remote` fails to start / header ignored | No Node/`npx`, or a space inside the `--header` arg | Install Node; keep `Authorization:${INSTATIC_MCP_AUTH}` without spaces |
| Fewer than 53 tools, but more than ~2 | Token lacks some capabilities | Step 4 |
| Only ~2 tools listed / connector shows "0 capabilities" | Known bug in some Instatic versions: `capabilities_json` stored double-encoded | See below |
| Write tool: "open the workspace" | Editor not open as the token owner | Step 5 |
| Tool missing although capability granted | `ai.tools.write` not granted (every mutating tool needs it) | Recreate the token with it |
| Hosted connector can't reach the server | Instance on localhost/LAN/HTTP | Hosted clients need public HTTPS; use a personal token locally |

**"0 capabilities" fix** (self-hosted, Postgres; confirm with the operator before touching a database):

```sql
UPDATE ai_mcp_connectors
   SET capabilities_json = (capabilities_json #>> '{}')::jsonb
 WHERE jsonb_typeof(capabilities_json) = 'string';
```

Then reconnect the MCP client. Re-saving the connector in the UI can re-break it; the fix is idempotent. Upgrading Instatic is the durable fix.
