# Codex Workflow

Use this reference only when the agent is running inside Codex and the user wants Flare canvas work with visible live feedback.

## Codex Image Generation To Flare Canvas

For requests like `/flare 生成一张照片`, `/flare 生成图片放到画布`, or "用你自己的生图能力":

1. Load and follow the installed `$imagegen` skill when available.
2. Generate the image with Codex's image generation path, not Flare's `create_generation_job`.
3. Keep or obtain a `dataUrl`, `dataBase64 + mimeType`, local generated image converted to data URL, or public HTTPS image URL.
4. Use the in-app browser context when available to identify the current Flare project URL and `projectId`.
5. Read Flare MCP state with `get_live_canvas_context` and/or `get_canvas_snapshot`.
6. Call `insert_agent_generated_image` with the generated image payload or URL.
7. Default to root canvas layer. Use `anchorNodeId` for placement and sizing, not `parentId`, unless explicit.
8. Verify by reading `get_canvas_snapshot` and visually checking the in-app browser so the user can watch the result appear.

## Browser Behavior

- Prefer the current in-app browser tab when it already shows `app.flare.design` or `app.staging.flare.design`.
- Do not log in as a different user. If auth is missing, ask the user to log in, then continue.
- After inserting the image, refresh or wait for collab sync only if the image is not visible.
- If the browser and MCP snapshot disagree, trust MCP for saved state but inspect browser sync before retrying.

## Forbidden Path

Do not call `create_generation_job` for Codex-side image generation unless the user explicitly says to use Flare backend/canvas generation or to test the Flare generation queue.
