# Validation

## General Verification

After every write, verify through at least one authoritative source, except plain image Fast Path and annotation-revision autoInsert flows that explicitly stop after a successful upload response:

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
- The agent/client image generation path was used for plain image/photo/illustration wording in any language.
- A fresh bitmap was generated for the request; no existing Flare asset was reused as a substitute unless the user explicitly asked for reuse/import or the operation was recovering the exact file generated in the current attempt.
- No image file was passed as base64 or data URL in MCP JSON.
- Local file upload used `create_image_upload_session`, fallback `get_image_upload_endpoint`, or a client upload helper, not a searched/exposed MCP OAuth token.
- Local generated image data was binary-uploaded into Assets before canvas insertion.
- Asset was saved with generation/source provenance where supported.
- Canvas image node exists.
- Fast Path uses `autoInsert` when available and stops after the upload response confirms insertion.
- Image is a root layer unless user explicitly requested `parentId`.
- `x/y` are center scene coordinates.
- The inserted image is inside the current/default visible viewport unless the user requested exact offscreen coordinates.
- Browser refresh/collab-sync checks are only for recovery, not the default Fast Path.

For annotated image revisions:

- A fresh revised bitmap was generated with the agent/client image generation or image-edit capability.
- The revision was not faked with canvas shapes/text, ordinary layer edits, existing asset reuse, or deterministic local image-processing scripts.
- The generation/edit prompt explicitly says to change only the annotated points, regions, or text requests.
- Unannotated composition, subject identity, background, lighting, camera angle, color palette, and style are preserved unless the user explicitly requested a global redesign.
- The revised image is placed beside the original using `suggestedAutoInsert` or fallback `suggestedPlacement`.
- The annotation-revision path stops after the upload response confirms `autoInsert` unless recovery or explicit verification is needed.

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
6. If an image node exists but is offscreen, move the existing node into view instead of regenerating or re-uploading.

Avoid repeated generation jobs when the failure is actually insertion, asset indexing, or browser refresh.

## Common Mistakes

- Calling `create_generation_job` for agent-generated images.
- Reusing an old matching asset instead of generating a new bitmap for a plain generation request.
- Passing image files as `dataUrl` or `dataBase64` instead of writing a local file and binary-uploading it.
- Searching shell environment, config files, browser storage, or logs for MCP OAuth tokens.
- Falling back to Flare app UI upload when an agent-generated local file should use `create_image_upload_session` or `get_image_upload_endpoint`.
- Treating a generic "generate/create" word in any language as Flare backend generation without explicit backend wording.
- Passing `parentId` because a frame is selected, causing generated media to disappear inside or behind an artboard.
- Treating top-left coordinates as `x/y`; Flare expects center coordinates.
- Re-uploading an existing asset instead of using `insert_asset_image`.
- Regenerating an image when the only problem is offscreen placement.
- Letting an annotated image revision drift into a full-image redesign when the user only marked local changes.
- Applying motion without resolving the selected frame.
- Assuming a browser screenshot is authoritative when MCP snapshot says the collab state differs.
