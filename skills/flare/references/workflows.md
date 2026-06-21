# Workflows

## Agent-Generated Image To Canvas

Use this when the user wants Claude, Codex, or another agent/client to generate or obtain an image and place it in Flare.

1. Use the available client-side image generation or retrieval capability to create or obtain the bitmap. Obtain a `dataUrl`, `dataBase64 + mimeType`, or public HTTPS `imageUrl`.
2. Identify `projectId` from the project URL or MCP project list.
3. Read `get_live_canvas_context` when placement should respect current viewport or selection.
4. Call `insert_agent_generated_image`. If unavailable, fall back to `insert_codex_generated_image` or `insert_generated_image`.
5. Default to root canvas layer. Use `anchorNodeId` for placement, not `parentId`, unless explicit.
6. Verify with `get_canvas_snapshot` and, if visible, the browser.

Plain image requests like `/flare 生成一张照片` should use this workflow by default. Do not switch to Flare backend generation unless the user explicitly asks for the Flare/canvas generation backend.

Example arguments:

```json
{
  "projectId": "project-id",
  "dataUrl": "data:image/png;base64,...",
  "fileName": "agent-glass-cube.png",
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
3. If the output is not automatically present in Assets/canvas, use the generated output URL or bytes with `insert_generated_image`.
4. Verify the asset, canvas node, and job status.

Do not use this workflow for plain "生成图片/照片/插图" requests, or phrases like "Codex 内生图", "Claude 自己生成", or "不要调用画布生图"; use the agent-generated image workflow instead.

## Existing Asset To Canvas

1. Use `list_assets` to find the target asset, filtering by kind or source type when useful.
2. Call `insert_asset_image` or `insert_asset_video`.
3. Use `x/y/width/height` for exact placement, or `anchorNodeId + placement` for relative placement.
4. Verify node creation.

## Save Media To Assets

For image bytes or data URLs:

```text
save_image_asset
```

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
