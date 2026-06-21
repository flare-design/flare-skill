# Validation

## General Verification

After every write, verify through at least one authoritative source:

- Canvas node created/updated/deleted: `get_canvas_snapshot`
- Live selection or collaborator state: `get_live_canvas_context`
- Asset saved: `list_assets`
- Generation status: `get_generation_job`
- Render status: `get_render_job`
- Visual placement: browser screenshot when available

Report concrete ids: `projectId`, asset id, node id, job id, or render id.

## Generated Image Checklist

For agent-generated image insertion:

- No `create_generation_job` was called.
- The agent/client image generation path was used for plain "生成图片/照片/插图" wording.
- No image file was passed as base64 or data URL in MCP JSON.
- Local generated image data was binary-uploaded into Assets before canvas insertion.
- Asset was saved with generation/source provenance where supported.
- Canvas image node exists.
- Image is a root layer unless user explicitly requested `parentId`.
- `x/y` are center scene coordinates.
- The browser shows the image after refresh or collab sync.

## Motion Checklist

For motion design:

- Target frame/artboard id is known.
- Existing motion was preserved or replaced intentionally.
- Layer animations match layer purpose.
- Composition duration/fps/poster settings are explicit or sensible.
- Result can be read back from snapshot or project export.

## Failure Recovery

If an operation appears to fail:

1. Check whether it partially succeeded before retrying.
2. For image insertion, inspect both Assets and canvas nodes.
3. For generation/render, inspect job status and error message.
4. For live canvas issues, compare snapshot vs browser state and refresh stale collab context.
5. Retry with idempotent names or explicit ids only when safe.

Avoid repeated generation jobs when the failure is actually insertion, asset indexing, or browser refresh.

## Common Mistakes

- Calling `create_generation_job` for agent-generated images.
- Passing image files as `dataUrl` or `dataBase64` instead of writing a local file and binary-uploading it.
- Treating the Chinese word "生成" as Flare backend generation without explicit backend wording.
- Passing `parentId` because a frame is selected, causing generated media to disappear inside or behind an artboard.
- Treating top-left coordinates as `x/y`; Flare expects center coordinates.
- Re-uploading an existing asset instead of using `insert_asset_image`.
- Applying motion without resolving the selected frame.
- Assuming a browser screenshot is authoritative when MCP snapshot says the collab state differs.
