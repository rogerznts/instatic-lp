---
name: instatic-setup
description: Connect an AI client to an Instatic instance over MCP and troubleshoot the connection. Use when the user wants to set up, connect, configure or debug the Instatic MCP (mcp__instatic__* tools missing, only 2 tools listed, "0 capabilities", "open the workspace" errors, 401s), create an Instatic access token, or asks "conectar o instatic", "configurar o MCP do instatic".
---

# Instatic setup

Instatic (https://github.com/CoreBunch/Instatic) is a self-hosted visual CMS that is itself an MCP server. Every instance exposes one endpoint:

```text
https://<host>/_instatic/mcp
```

## 1. Check whether it is already connected

Look for tools named `mcp__instatic__*` (or `mcp__<whatever-name>__get_context`). If they exist, call `get_context` — it is headless and reports whether the Site editor and Content workspace are connected. If it works, skip to step 4.

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

## 3. Register the server (Claude Code)

Never paste the token into a file that gets committed. Give the user the command to run themselves (suggest the `!` prefix so it runs in-session), with their host and token:

```sh
claude mcp add instatic --transport http --scope user \
  https://<host>/_instatic/mcp \
  --header "Authorization: Bearer imcp_pat_…"
```

- `--scope user` makes it available in every project; `--scope local` (default) only in the current directory. Use `--scope project` only if the team shares an instance and each member supplies their own token via `${INSTATIC_TOKEN}` in `.mcp.json`.
- The tool list is built on connect: after adding or changing capabilities, restart the session / reconnect with `/mcp`.

For Codex (`~/.codex/config.toml`), use an HTTP MCP entry with the same URL and `Authorization: Bearer` header, read from an environment variable.

## 4. Understand the two execution modes

This is the most common source of confusion:

- **Headless tools** work with no browser open: `get_context`, `site_list_*` (documents, breakpoints, modules, post types, loop sources), `site_read_styles`, `content_list_*` / `content_get_*` / `content_search_documents`, `media_upload`, `site_publish`.
- **Browser tools** (all page-tree, HTML/CSS, token, page lifecycle, content mutation and code-asset tools, plus `site_read_document` and `site_render_snapshot`) are relayed to the **connection owner's open editor**. The user must have the Instatic editor open in a browser, logged in as the same user that created the token — the **Site** workspace for site tools, the **Content** workspace for content tools. Otherwise the tool returns an "open the workspace" error.

When a browser tool fails with that error, ask the user to open `https://<host>/admin` in the right workspace and keep the tab open (a backgrounded tab is fine), then retry.

## 5. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| No `mcp__instatic__*` tools | Server not registered in this scope, or session not restarted | `claude mcp list`; re-add with the right `--scope`; reconnect |
| 401 Unauthorized | Token revoked/expired, or missing `Bearer ` prefix | Create a new token; check the header |
| Only ~2 tools listed / connector shows "0 capabilities" | Known bug in some Instatic versions: `capabilities_json` stored double-encoded | See below |
| Write tool: "open the workspace" | Editor not open as the token owner | Step 4 |
| Tool missing although capability granted | `ai.tools.write` not granted (every mutating tool needs it) | Recreate the token with it |
| Hosted connector can't reach the server | Instance on localhost/LAN/HTTP | Hosted clients need public HTTPS; use a personal token locally |

**"0 capabilities" fix** (self-hosted, Postgres; confirm with the operator before touching a database):

```sql
UPDATE ai_mcp_connectors
   SET capabilities_json = (capabilities_json #>> '{}')::jsonb
 WHERE jsonb_typeof(capabilities_json) = 'string';
```

Then reconnect the MCP client. Re-saving the connector in the UI can re-break it; the fix is idempotent. Upgrading Instatic is the durable fix.
