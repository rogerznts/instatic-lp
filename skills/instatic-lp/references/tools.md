# Instatic MCP tool catalog

Source: CoreBunch/Instatic `main` @ f92e8dc (2026-09-13). The live list is filtered by the connection's capabilities; if a tool listed here is missing, the token lacks its capability (every mutating tool also needs `ai.tools.write`) or the instance runs another version.

**Execution:** `server` = headless, works with no browser open. `browser` = relayed to the token owner's open editor (Site or Content workspace).

## Orientation and reads (headless)

| Tool | Capability | Use |
|---|---|---|
| `get_context` | site read | Editor/workspace connected? Which templates wrap pages. Call first. |
| `site_list_documents` | `site.read` | Pages, templates, visual components + ids and template config. |
| `site_read_styles` | `site.read` | Design tokens, classes, rules. |
| `site_list_breakpoints` | `site.read` | Breakpoint ids and widths for `@media` and snapshots. |
| `site_list_modules` | `site.read` | Available node modules. |
| `site_list_post_types` | `site.read` | Post-type slugs for `postTypes` templates. |
| `site_list_loop_sources` | `site.read` | Source/table ids and `{currentEntry.*}` tokens for `<instatic-loop>`. |
| `content_list_media` | `media.read` | Existing media assets. |
| `content_list_collections`, `content_get_collection_schema`, `content_list_documents`, `content_get_document`, `content_search_documents` | content read | CMS collections and entries. |
| `content_list_users` | `users.manage` | Author ids. |

## Page tree and HTML (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_read_document` | `site.read` | Annotated HTML + CSS (`uid` per node). Paginated via `pageInfo.nextPart`. |
| `site_get_node_html` | — | HTML for one subtree. |
| `site_open_document` | `site.read` | Visibly switch the canvas (before snapshotting another document). |
| `site_insert_html` | `site.structure.edit` | Insert HTML + `<style>` under `parentId`. Returns `nodeIds` and `created`. |
| `site_replace_node_html` | `site.structure.edit` | Rebuild a node's children. |
| `site_update_node_props` | `site.content.edit` | Change text/attributes of a node. |
| `site_move_node`, `site_duplicate_node`, `site_delete_node` | `site.structure.edit` | Tree operations. |
| `site_rename_node` | `site.content.edit` | Rename a node in the layers panel. |

## CSS and design tokens (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_apply_css` | `site.style.edit` | CSS on its own. `operation`: `merge` / `remove-properties` / `replace` / `delete`. |
| `site_assign_class`, `site_remove_class` | `site.style.edit` | Attach/detach classes. |
| `site_set_color_tokens` | `site.style.edit` | Colors → `var(--<slug>)`. |
| `site_set_font_tokens` | `site.style.edit` | Typefaces (`googleFamily` installs a web font). |
| `site_set_type_scale` | `site.style.edit` | `--text-*` scale. |
| `site_set_spacing_scale` | `site.style.edit` | `--space-*` scale. |

## Pages and templates (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_add_page` | `site.structure.edit` | New page → `pageId` + `rootNodeId`. Call once. |
| `site_duplicate_page`, `site_rename_page`, `site_delete_page` | `site.structure.edit` | Lifecycle. Slug `index` = homepage. Last page cannot be deleted. |
| `site_set_page_template`, `site_clear_page_template` | `site.structure.edit` | Turn a page into an `everywhere` / `postTypes` template (needs one `<instatic-outlet>`). |

## Code assets (browser)

| Tool | Capability | Use |
|---|---|---|
| `site_list_code_assets`, `site_read_code_asset` | `site.read` | Inspect scripts/stylesheets. |
| `site_write_code_asset`, `site_patch_code_asset` | `site.structure.edit` | Add/patch behaviour (`src/scripts/...`); npm deps via `dependencies`. |
| `site_inspect_code_runtime` | `site.read` | Confirm scripts apply to the page with the right timing. |

## Verification, media and publishing

| Tool | Execution | Capability | Use |
|---|---|---|---|
| `site_render_snapshot` | browser | — | Geometry, warnings, image status, computed styles (+ screenshot on some clients). `breakpointId`, `nodeId`. |
| `media_upload` | server | `media.write` | Upload image from https `sourceUrl` or base64 → `id` + `publicPath`. Newer versions only. |
| `site_publish` | server | `pages.publish` | Publish the **entire** site draft. Once, only on request. |

## Content writes (browser, Content workspace)

`content_create_document`, `content_set_document_field(s)`, `content_set_document_status`, `content_delete_document`, `content_set_document_author`, `content_set_active_document`, `content_set_active_collection`. Body is exchanged as Markdown.
