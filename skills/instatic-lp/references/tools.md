# Instatic MCP tool catalog

Source: CoreBunch/Instatic `main` @ f92e8dc (2026-09-13), matched against a live instance with every capability granted.

## Availability

A connection with **all** capabilities sees **53 tools** in three areas:

| Area | Count | Tools |
|---|---|---|
| Context and media | 2 | `get_context`, `media_upload` |
| Content (collections and documents) | 15 | 7 reads + 8 writes/navigation |
| Site (pages, nodes, CSS, publishing) | 36 | 6 reads (headless) + `site_publish` + 29 editor tools |

The server builds the list on connect and filters it by the token's capabilities (`requiredCapabilities` is ANY-OF; every mutating tool also needs `ai.tools.write`). Fewer than 53 means a narrower token or a different Instatic version, not a broken server. A tool that is not listed cannot be called: check the tables below for the capability it needs, recreate the token, reconnect.

**Execution:**

- `server` (16 tools): headless, works with no browser open.
- `browser` (37 tools): relayed to the token owner's open editor, **Site** workspace for `site_*`, **Content** workspace for `content_*` writes. Otherwise: "open the workspace" error.

Tool names are bare here; clients prefix them with the server name (`mcp__instatic__site_insert_html`).

## Context and media (server)

| Tool | Capability | Use |
|---|---|---|
| `get_context` | any of `site.read`, `pages.edit`, `content.manage`, `data.*.tables.read` | Are the Site and Content editors connected? Which templates wrap pages. Call first. |
| `media_upload` | `media.write` | Upload an image from base64 `data` or an https `sourceUrl` → `id` + `publicPath` for `<img src>`. Missing on older Instatic versions. |

## Content: collections and documents

### Reads (server)

| Tool | Capability | Use |
|---|---|---|
| `content_list_collections`, `content_get_collection_schema` | any of `data.custom.tables.read/manage`, `data.system.tables.read/manage` | Collections and their field schema (read the schema before writing fields). |
| `content_list_documents`, `content_get_document`, `content_search_documents` | any of `content.create`, `content.edit.own/any`, `content.publish.own/any`, `content.manage` | Entries of a collection. |
| `content_list_media` | `media.read` | Existing media assets. |
| `content_list_users` | `users.manage` | Author ids for `content_set_document_author`. |

### Writes and navigation (browser, Content workspace)

| Tool | Capability | Use |
|---|---|---|
| `content_create_document` | `content.create` | New entry; leaves it **active**, so create-then-fill needs no extra call. |
| `content_set_document_field`, `content_set_document_fields` | any of `content.edit.own/any`, `content.manage` | Write one / several fields. **Only on the active document**: call `content_set_active_document` first or the write is refused. Body is Markdown. |
| `content_set_document_status` | any of `content.publish.own/any` | `draft`, `scheduled` (needs `scheduledAt`, ISO) or `published`. |
| `content_set_document_author` | any of `content.edit.any`, `content.manage` | Reassign author (ids from `content_list_users`). |
| `content_delete_document` | any of `content.edit.own/any`, `content.manage` | Moves to trash; restorable. |
| `content_set_active_document`, `content_set_active_collection` | — | Switch what the user sees in the editor. |

## Site: pages, nodes, CSS and publishing

Site edits land in the **draft**; nothing is public until `site_publish`.

### Reads (server)

| Tool | Capability | Use |
|---|---|---|
| `site_list_documents` | `site.read` | Pages, templates, visual components + ids and template config. |
| `site_read_styles` | any site capability or `pages.edit` | The Framework panel as CSS: font tokens + color / `--text-*` / `--space-*` variables under `/* === Design tokens === */`, then author classes and rules. `format:"summary"` = class names + tokens each uses; `className` = one class. Read before any page work. |
| `site_list_breakpoints` | any site capability or `pages.edit` | Breakpoint ids and widths for `@media` and snapshots. |
| `site_list_modules` | `site.read` | Available node modules. |
| `site_list_post_types` | `site.read` | Post-type slugs for `postTypes` templates. |
| `site_list_loop_sources` | `site.read` | Source/table ids and `{currentEntry.*}` tokens for `<instatic-loop>`. |

### Documents (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_read_document` | `site.read` | Annotated HTML + CSS (`uid` per node). Paginated via `pageInfo.nextPart`. |
| `site_open_document` | `site.read` | Visibly switch the canvas (before snapshotting another document). |

### Pages and templates (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_add_page` | `site.structure.edit` | New page → `pageId` + `rootNodeId`. Call once. |
| `site_duplicate_page`, `site_rename_page`, `site_delete_page` | `site.structure.edit` | Lifecycle. Slug `index` = homepage. Last page cannot be deleted. |
| `site_set_page_template`, `site_clear_page_template` | `site.structure.edit` | Turn a page into an `everywhere` / `postTypes` template (needs one `<instatic-outlet>`). |

### Nodes / HTML (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_insert_html` | `site.structure.edit` | Insert HTML + `<style>` under `parentId`. Returns `nodeIds` and `created`. |
| `site_get_node_html` | — | HTML for one subtree (read before replacing). |
| `site_replace_node_html` | `site.structure.edit` | Rebuild a node's children. |
| `site_update_node_props` | `site.content.edit` or `site.structure.edit` | Change text/attributes of a node. |
| `site_move_node`, `site_duplicate_node`, `site_delete_node` | `site.structure.edit` | Tree operations. |
| `site_rename_node` | `site.content.edit` or `site.structure.edit` | Rename a node in the layers panel. |

### Classes and CSS (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_apply_css` | `site.style.edit` | CSS on its own. `operation`: `merge` / `remove-properties` / `replace` / `delete`. |
| `site_assign_class`, `site_remove_class` | `site.style.edit` | Attach/detach classes. |

### Design tokens (browser) — site-wide framework writes

These rewrite the Framework panel (Colors, Type, Space) for the **whole site**; every page restyles. Read the framework with `site_read_styles`; call these only when the user explicitly asks to change it or the framework is empty and they approved creating it.

| Tool | Capability | Use |
|---|---|---|
| `site_set_color_tokens` | `site.style.edit` | Colors → `var(--<slug>)`. |
| `site_set_font_tokens` | `site.style.edit` | Typefaces (`googleFamily` installs a web font). |
| `site_set_type_scale` | `site.style.edit` | `--text-*` scale. |
| `site_set_spacing_scale` | `site.style.edit` | `--space-*` scale. |

### Code assets (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_list_code_assets`, `site_read_code_asset` | `site.read` | Inspect scripts/stylesheets. |
| `site_write_code_asset`, `site_patch_code_asset` | `site.structure.edit` | Add/patch behaviour (`src/scripts/...`); npm deps via `dependencies`. |
| `site_inspect_code_runtime` | `site.read` | Confirm scripts apply to the page with the right timing. |

### Visual inspection (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_render_snapshot` | — | Geometry, overflow/visibility warnings, image status, computed styles (+ screenshot on some clients). `breakpointId`, `nodeId`. |

### Publishing (server)

| Tool | Capability | Use |
|---|---|---|
| `site_publish` | `pages.publish` | Publish the **entire** site draft to the public site. Once, only on request. |
