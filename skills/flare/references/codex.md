# Codex Workflow

Use this reference only when the agent is running inside Codex and the user wants Flare canvas work with visible live feedback.

## Codex Image Generation To Flare Canvas

For requests like `/flare 生成一张照片`, `/flare 生成图片放到画布`, or "用你自己的生图能力":

1. Open or focus the current in-app browser tab when it shows Flare. If no Flare project is open, navigate to the target project URL before editing so the user can watch the canvas update.
2. Load and follow the installed `$imagegen` skill when available.
3. Generate the image with Codex's image generation path, not Flare's `create_generation_job`.
4. Keep the generated bitmap as a local file. Inspect its dimensions and MIME type if available. Do not convert local files to base64 for MCP arguments. If the generation API returns a data URL or base64 string, decode and write it to a local image file before any Flare MCP call.
5. Identify `projectId` from the in-app browser URL or MCP project list.
6. Read Flare MCP state with `get_live_canvas_context` and/or `get_canvas_snapshot`.
7. Use `get_image_upload_endpoint` when available, then binary-upload the local file into Assets. If the MCP client exposes its own file upload helper, use that helper; otherwise use the returned endpoint with raw file bytes, `Content-Type`, `x-flare-filename`, and `x-flare-file-size`.
8. Call `insert_asset_image` with the returned `assetId`. Use root canvas placement by default; use `anchorNodeId` for placement and sizing, not `parentId`, unless explicit.
9. Verify by reading `get_canvas_snapshot` and visually checking the in-app browser. The browser verification is required in Codex desktop when the user expects live interaction.

Only use `insert_agent_generated_image` directly when the generated image is already at a public HTTPS URL. Never shrink or re-encode just to squeeze base64 through MCP JSON.

## Browser Behavior

- Prefer the current in-app browser tab when it already shows `app.flare.design` or `app.staging.flare.design`.
- If the user says they want to watch the operation, open or focus the in-app browser before generating or inserting.
- Do not log in as a different user. If auth is missing, ask the user to log in, then continue.
- After inserting the image, refresh or wait for collab sync only if the image is not visible.
- If the browser and MCP snapshot disagree, trust MCP for saved state but inspect browser sync before retrying.

## Forbidden Path

Do not call `create_generation_job` for Codex-side image generation unless the user explicitly says to use Flare backend/canvas generation or to test the Flare generation queue.
