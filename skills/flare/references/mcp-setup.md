# MCP Setup

## Environments

Use the user's current Flare environment unless they specify otherwise.

- Staging app: `https://app.staging.flare.design`
- Staging MCP: `https://mcp.staging.flare.design/mcp`
- Production MCP, when configured: `https://mcp.flare.design/mcp`

If the user is testing staging, connect the staging MCP endpoint. Mixing production MCP with a staging project URL will usually fail or operate on the wrong account/project.

## Required Scopes

Request only the scopes needed for the task:

- Project read: `projects:read`
- Canvas read/write: `canvas:read`, `canvas:write`
- Asset read/write: `assets:read`, `assets:write`
- Generation jobs: `generation:create`, plus read scope if the server exposes one
- Render jobs: render/create and read scopes if the server exposes them

For normal canvas generation workflows, the practical full set is:

```text
projects:read canvas:read canvas:write assets:read assets:write generation:create
```

Add render scopes only for render/export tasks.

## Connection Check

Before giving up on an operation, verify:

1. The MCP server is connected for the same environment as the app URL.
2. `tools/list` includes the expected Flare tools.
3. OAuth is authorized for the same Flare account/workspace as the browser session.
4. The project id in the URL matches the `projectId` passed to MCP tools.

## Expected Tool Families

The Flare MCP server should expose tools for:

- Projects: `list_projects`, `get_project`
- Assets: `list_assets`, `save_image_asset`, `save_image_asset_from_url`
- Generated image insertion: `insert_agent_generated_image`, `insert_codex_generated_image`, `insert_generated_image`. Prefer `insert_agent_generated_image`; `insert_codex_generated_image` is a compatibility alias for older clients and servers.
- Canvas reads: `get_canvas_snapshot`, `get_live_canvas_context`, `export_project_snapshot`
- Canvas writes: `insert_asset_image`, `insert_asset_video`, `apply_canvas_patch`, `insert_text`, `insert_rect`, `create_frame`, `update_text`, `set_layer_style`, `reorder_nodes`, `group_nodes`, `delete_nodes`
- Generation jobs: `create_generation_job`, `get_generation_job`, `list_generation_jobs`
- Motion and timeline: `apply_motion_design`, `add_audio_track`, `add_caption_track`
- Render jobs: `create_render_job`, `get_render_job`, `list_render_jobs`

If a tool is missing, adapt to the available list and report the missing capability clearly.
