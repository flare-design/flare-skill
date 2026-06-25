# MCP Setup

## Environments

Use the production Flare environment unless the user explicitly provides a different endpoint.

- App: `https://app.flare.design`
- MCP: `https://mcp.flare.design/mcp`
- Auth: `https://auth.flare.design`

## Required Scopes

Request only the scopes needed for the task:

- Project read: `projects:read`
- Canvas read/write: `canvas:read`, `canvas:write`
- Asset read/write: `assets:read`, `assets:write`
- Generation jobs: `generation:read`, `generation:create`
- Render jobs: `render:read`, `render:create`

For normal agent-side canvas workflows, the recommended set is:

```text
projects:read canvas:read canvas:write assets:read assets:write
```

On the Flare authorization page, these agent-recommended scopes should be selected by default. Generation and rendering scopes should remain unselected by default because `generation:create` and `render:create` can consume Flares, render allowance, or plan usage.

Request generation scopes only when the user explicitly asks to use Flare backend generation or to inspect Flare generation jobs. Request render scopes only for render/export tasks.

## Connection Check

Before giving up on an operation, verify:

1. The MCP server is connected for the same environment as the app URL.
2. `tools/list` includes the expected Flare tools.
3. OAuth is authorized for the same Flare account/workspace as the browser session.
4. The project id in the URL matches the `projectId` passed to MCP tools.

## Skill Update Check

When `check_client_setup` is exposed, call it once before the first visible canvas write in a conversation:

```json
{
  "client": "codex",
  "skillName": "flare",
  "installedSkillVersion": "0.1.22"
}
```

Use the actual client name when known, such as `codex`, `claude-code`, `chatgpt`, or `cursor`.

- If `updateRequired` is true, stop before editing and ask the user to run the returned `updateCommand`.
- If `updateAvailable` is true but `updateRequired` is false, mention the returned `updateCommand` once, then continue unless the current task depends on the newer workflow.
- If the tool is missing from an older MCP server, continue with the available tools and do not invent a version check.

## Expected Tool Families

The Flare MCP server should expose tools for:

- Client setup: `check_client_setup`
- Projects: `list_projects`, `create_project`, `get_project`
- Assets: `list_assets`, `create_image_upload_session`, `get_image_upload_endpoint`, `save_image_asset_from_url`
- Generated image insertion: `insert_agent_generated_image`, `insert_codex_generated_image`, `insert_generated_image`. Prefer `insert_agent_generated_image`; `insert_codex_generated_image` is a compatibility alias for older clients and servers.
- Canvas reads: `get_canvas_snapshot`, `get_live_canvas_context`, `get_image_annotation_context`, `export_project_snapshot`
- Canvas writes: `insert_asset_image`, `insert_asset_video`, `apply_canvas_patch`, `insert_text`, `insert_rect`, `create_frame`, `update_text`, `set_layer_style`, `reorder_nodes`, `group_nodes`, `delete_nodes`
- Generation jobs: `create_generation_job`, `get_generation_job`, `list_generation_jobs`
- Motion and timeline: `apply_motion_design`, `add_audio_track`, `add_caption_track`
- Render jobs: `create_render_job`, `get_render_job`, `list_render_jobs`

If a tool is missing, adapt to the available list and report the missing capability clearly.

## Binary Image Upload

For local image files, use the MCP/client binary upload path rather than JSON base64.

- Preferred tool: `create_image_upload_session`
- Fallback/legacy tool: `get_image_upload_endpoint`; use its returned `uploadToken` when `create_image_upload_session` is not exposed in the current client's cached tool list.
- Upload URL: use the exact `uploadUrl` returned by the tool; on current hosted environments this is the MCP resource URL (`/mcp`) with an image `Content-Type`.
- Method: `POST`
- Required headers: `Authorization: Bearer <uploadToken>`, `Content-Type: image/*`, `x-flare-file-size`
- Useful headers: `x-flare-filename`, `x-flare-source-model`, `x-flare-source-client`, `x-flare-generation-prompt`, `x-flare-generation-model`, `x-flare-generation-tool`
- Body: raw file bytes

Do not look for or expose the MCP OAuth access token in shell commands. The `uploadToken` returned by `create_image_upload_session` or `get_image_upload_endpoint` is short-lived and scoped to binary image upload.

For data URLs or base64 image data, write the bytes to a local image file first, then use the same upload-session flow.

After upload, use the returned `assetId` with `insert_asset_image`.
