# Codex Workflow

Use this reference only when the agent is running inside Codex and the user wants Flare canvas work with visible live feedback.

## Codex Image Generation To Flare Canvas

Use this workflow when the user wants Codex to generate the image itself and place it in Flare.

Common Chinese triggers:

- `/flare 生成一张照片`
- `/flare 生成图片放到画布`
- `用你自己的生图能力`

1. Open or focus the in-app browser before generating or inserting. In Codex desktop, assume the user wants to watch live canvas changes by default; do not wait for the user to explicitly ask. Prefer the current Flare tab. If Flare is not logged in, ask the user to log in in that browser and wait before continuing. If no Flare project is open and no target URL or project id is known, call `list_projects` with `limit: 5` and `status: "active"`, show the recent project candidates, and wait for the user to choose before editing.
2. Call `check_client_setup` when exposed with `client: "codex"`, `skillName: "flare"`, and `installedSkillVersion: "0.1.13"`. If `updateRequired` is true, ask the user to run the returned `updateCommand` before continuing. If only `updateAvailable` is true, mention it once and continue when safe.
3. Load and follow the installed `$imagegen` skill when available.
4. Generate the image with Codex's image generation path, not Flare's `create_generation_job`.
5. Keep the generated bitmap as a local file. Inspect its dimensions and MIME type if available. Do not convert local files to base64 for MCP arguments. If the generation API returns a data URL or base64 string, decode and write it to a local image file before any Flare MCP call.
6. Identify `projectId` from the in-app browser URL, the user's explicit URL/id, or the selected recent project. Do not guess between multiple projects.
7. Read Flare MCP state with `get_live_canvas_context` and/or `get_canvas_snapshot`.
8. Call `create_image_upload_session` with the local file name, byte size, MIME type, and provenance. Use `sourceClient: "codex"`, keep legacy `sourceModel: "codex"` when accepted, put the actual image model in `generationModel`, put the prompt in `generationPrompt`, and put the generation surface in `generationTool` (for example `imagegen`) when known. If the current Codex thread only exposes `get_image_upload_endpoint`, call it and use its returned generic `uploadToken`.
9. Binary-upload the local file into Assets with the returned `uploadUrl` and `uploadToken`; use raw file bytes, `Content-Type`, `x-flare-file-size`, and optional `x-flare-filename`. Do not put the local image file, data URL, or base64 string into MCP JSON.
10. Call `insert_asset_image` with the returned `assetId`. Use root canvas placement by default; use `anchorNodeId` for placement and sizing, not `parentId`, unless explicit.
11. Verify by reading `get_canvas_snapshot` and visually checking the in-app browser. The browser verification is required in Codex desktop when the user expects live interaction.

Only use `insert_agent_generated_image` directly when the generated image is already at a public HTTPS URL. Never shrink or re-encode just to squeeze base64 through MCP JSON.

Codex-generated images should show up in Flare as generated media with an MCP binary upload ingest method. They should not be treated as ordinary user uploads, and they should not create a fake Flare backend generation job.

Do not run shell searches for OAuth, token, or MCP credentials. Codex shell does not need the MCP OAuth token; it needs the short-lived `uploadToken` returned by `create_image_upload_session` or `get_image_upload_endpoint`.

Do not use the Flare app UI upload flow as a fallback for agent-generated images. Use MCP upload sessions so Assets, provenance, and canvas insertion stay deterministic.

## Codex Annotated Image Revision

Use this workflow when the user says to revise an image from Flare canvas annotations, such as `按照批注改图`.

1. Open or focus the in-app browser before reading context. In Codex desktop, assume the user wants to watch live canvas changes by default; do not wait for the user to explicitly ask. Prefer the current Flare tab. If Flare is not logged in, ask the user to log in in that browser and wait before continuing. If no Flare project is open and no target URL or project id is known, call `list_projects` with `limit: 5` and `status: "active"`, show the recent project candidates, and wait for the user to choose before reading annotation context.
2. Call `check_client_setup` when exposed with `client: "codex"`, `skillName: "flare"`, and `installedSkillVersion: "0.1.13"`. If `updateRequired` is true, ask the user to run the returned `updateCommand` before continuing. If only `updateAvailable` is true, mention it once and continue when safe.
3. Call `get_image_annotation_context` with `projectId`. Pass `nodeId` or `annotationId` only when the user identified a specific image/session; otherwise let Flare resolve the active selection. The tool returns `annotatedImage` by default; set `includeAnnotatedImage: false` only when the client needs a smaller text-only response.
4. Treat `target.src` and `annotations` as the edit contract. Use `annotatedImage` as a visual reference, not as the only source of truth. Do not infer coordinates from a browser screenshot when the structured annotations are available.
5. Interpret annotations precisely:
   - `arrow.to` is the target-image point to revise.
   - `rect.bounds` and `ellipse.bounds` are target-image regions.
   - `text.x/y` is a target-image text anchor.
   - `arrow.from` and `labelPosition` are layout/context hints and may be outside the target image.
6. Generate the revised image with Codex image generation using the target image URL, structured annotations, optional `annotatedImage` composite preview, and original asset provenance. Do not call Flare `create_generation_job`.
7. Save the revised bitmap as a local file. If the image generation result is a data URL or base64 string, decode it to a local file before any MCP call.
8. Upload the local file through `create_image_upload_session` and raw binary upload. Include provenance fields: `sourceClient: "codex"`, `generationPrompt`, `generationModel`, `generationTool`, and a concise `generationNotes` referencing the annotation session.
9. Call `insert_asset_image` with the returned `assetId` and the `suggestedPlacement` from the annotation context so the revised image appears beside the original.
10. Verify through `get_canvas_snapshot` and the in-app browser.

## Browser Behavior

- Always open or focus the in-app browser before Flare canvas generation, insertion, annotation revision, motion, or other visible canvas edits in Codex desktop.
- Prefer the current in-app browser tab when it already shows `app.flare.design` or `app.staging.flare.design`; otherwise navigate to the target project URL when it is known.
- If no target project is clear, call `list_projects` with `limit: 5` and `status: "active"`, present the recent project names, ids, and updated times, then wait for the user to pick one. Do not infer the project from the first result.
- If Flare is not logged in, ask the user to log in in the in-app browser and wait. Continue only after the browser session is authenticated.
- Keep the in-app browser visible through verification so the user can watch the canvas update without having to ask.
- Do not log in as a different user, search for tokens, or bypass auth.
- After inserting the image, refresh or wait for collab sync only if the image is not visible.
- If the browser and MCP snapshot disagree, trust MCP for saved state but inspect browser sync before retrying.

## Forbidden Path

Do not call `create_generation_job` for Codex-side image generation unless the user explicitly says to use Flare backend/canvas generation or to test the Flare generation queue.
