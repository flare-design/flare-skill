# Workflows

## Agent-Generated Image To Canvas

Use this when the user wants Claude, Codex, or another agent/client to generate or obtain an image and place it in Flare.

1. Use the available client-side image generation or retrieval capability to create or obtain the bitmap.
2. Keep or convert the result into a local image file. If the client produced a data URL or base64 string, decode it and write it to disk before touching Flare MCP.
3. Identify `projectId` from the project URL or MCP project list.
4. Read `get_live_canvas_context` when placement should respect current viewport or selection.
5. Call `create_image_upload_session`, then binary-upload the local file into Assets with the returned `uploadUrl` and `uploadToken`. The upload response returns `assetId` and an `asset` object.
6. Call `insert_asset_image` with the returned `assetId`. Use public URL import only when the image is already hosted at a public HTTPS URL.
7. Default to root canvas layer. Use `anchorNodeId` for placement, not `parentId`, unless explicit.
8. Verify with `get_canvas_snapshot` and, if visible, the browser.

Plain image requests like `/flare 生成一张照片` should use this workflow by default. Do not switch to Flare backend generation unless the user explicitly asks for the Flare/canvas generation backend.

Example insertion arguments after binary upload:

```json
{
  "projectId": "project-id",
  "assetId": "asset-id-returned-by-upload",
  "name": "Agent glass cube",
  "x": 0,
  "y": 0,
  "width": 512,
  "height": 512
}
```

## Flare Backend Generation

Use this only when the user explicitly wants to test or use Flare's own generation backend.

1. Call `create_generation_job` with the requested image/video prompt and model options.
2. Poll `get_generation_job` until complete or failed.
3. If the output is not automatically present in Assets/canvas, import a public output URL with `insert_generated_image`, or save output bytes to a local file, create an upload session, binary-upload them, then `insert_asset_image`.
4. Verify the asset, canvas node, and job status.

Do not use this workflow for plain "生成图片/照片/插图" requests, or phrases like "Codex 内生图", "Claude 自己生成", or "不要调用画布生图"; use the agent-generated image workflow instead.

## Existing Asset To Canvas

1. Use `list_assets` to find the target asset, filtering by kind or source type when useful.
2. Call `insert_asset_image` or `insert_asset_video`.
3. Use `x/y/width/height` for exact placement, or `anchorNodeId + placement` for relative placement.
4. Verify node creation.

## Save Media To Assets

For image bytes or data URLs:

Write the image to a local file, then binary-upload the file with the MCP/client upload endpoint.
Use `create_image_upload_session` for local files unless the MCP client has a built-in binary upload helper that already handles auth.

For public HTTPS URLs:

```text
save_image_asset_from_url
```

After saving, insert separately only if the user asked for placement.

## Motion Design For A Selected Board

1. Call `get_live_canvas_context`.
2. Resolve the selected frame/artboard id. If selection is missing or not a frame, ask the user to select one or provide `frameId`.
3. Read the canvas snapshot if layer ids or existing motion matter.
4. Design a concise motion plan: composition timing, layer-level animation roles, and whether to replace existing motion.
5. Call `apply_motion_design`.
6. Verify by reading the snapshot or motion data, then report what changed.

Motion design should be semantic and scene-aware. Avoid generic animations on every layer. Use layer purpose, visual hierarchy, and narrative order.

## Render Job

1. Read the project and motion/timeline state if needed.
2. Call `create_render_job` with the project id or explicit render snapshot.
3. Poll `get_render_job`.
4. Report render status, output URLs, and any failure reason.

## Canvas Editing

1. Read the current snapshot.
2. Prefer specialized tools for simple edits.
3. Use `apply_canvas_patch` for batches.
4. Verify created, updated, or deleted node ids.
5. If the browser is open, visually inspect after the patch for placement mistakes.
