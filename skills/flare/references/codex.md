# Codex Workflow

Use this reference only when the agent is running inside Codex and the user wants Flare canvas work with visible live feedback.

## Codex Image Generation to Flare Canvas

Use this workflow when the user wants Codex to generate the image itself and place it in Flare.

Typical intent patterns in any language:

- The user asks Codex or the current agent to create an image, photo, or illustration and place it in Flare.
- The user asks to use the agent/client's own image generation capability.
- The user asks not to use Flare, canvas, or backend generation.

1. Resolve the target `projectUrl` before opening or changing the in-app browser. In Codex desktop, assume the user wants to watch live canvas changes by default; do not wait for the user to explicitly ask. If a project id or URL is known, use `https://app.flare.design/projects/{projectId}` or the returned `projectUrl`. If no Flare project is open and no target URL or project id is known, use the Project Selection Behavior below before browser navigation. Once `projectUrl` is known, open or focus the in-app browser at that exact URL. Prefer an existing current Flare project tab only when it is already the target project. Do not open `app.flare.design`, `/projects`, the project list page, or project cards as an intermediate step. If browser-control tools are not available in the current Codex surface, say that explicitly before writing through MCP and verify with MCP snapshots instead. If Flare is not logged in, ask the user to log in in that browser and wait before continuing.
2. Call `check_client_setup` when exposed with `client: "codex"`, `skillName: "flare"`, and `installedSkillVersion: "0.1.20"`. If `updateRequired` is true, ask the user to run the returned `updateCommand` before continuing. If only `updateAvailable` is true, mention it once and continue when safe.
3. Load and follow the installed `$imagegen` skill when available.
4. Generate a fresh bitmap with Codex's image generation path, not Flare's `create_generation_job`. Do not skip generation and reuse an existing Flare asset just because a similar prompt, name, or image already exists. Reuse an existing asset only when the user explicitly asks for reuse/import, or when recovering from a partially successful upload of the exact file generated in this operation.
5. Keep the generated bitmap as a local file. Inspect its dimensions and MIME type if available. Do not convert local files to base64 for MCP arguments. If the generation API returns a data URL or base64 string, decode and write it to a local image file before any Flare MCP call.
6. Identify `projectId` from the in-app browser URL, the user's explicit URL/id, or the selected recent project. Do not guess between multiple projects.
7. Read Flare MCP state with `get_live_canvas_context` and/or `get_canvas_snapshot`.
8. Call `create_image_upload_session` with the local file name, byte size, MIME type, and provenance. Use `sourceClient: "codex"`, keep legacy `sourceModel: "codex"` when accepted, put the actual image model in `generationModel`, put the prompt in `generationPrompt`, and put the generation surface in `generationTool` (for example `imagegen`) when known. If the current Codex thread only exposes `get_image_upload_endpoint`, call it and use its returned generic `uploadToken`.
9. Binary-upload the local file into Assets with the returned `uploadUrl` and `uploadToken`; use raw file bytes, `Content-Type`, `x-flare-file-size`, and optional `x-flare-filename`. Do not put the local image file, data URL, or base64 string into MCP JSON.
10. Call `insert_asset_image` with the returned `assetId`. Omit `width` and `height` by default so Flare preserves the generated bitmap's original dimensions. Pass dimensions only when the user explicitly asks to resize, fill, crop, or match a selected target. Use root canvas placement by default; use `anchorNodeId` for placement, not `parentId`, unless explicit. If the user did not give exact coordinates, keep the image in the current/default visible viewport by choosing placement or `x/y`, not by shrinking the image.
11. Verify by reading `get_canvas_snapshot` and visually checking the in-app browser. The browser verification is required in Codex desktop when browser control is available. If the node exists but is not visible, update that existing node's placement or the browser viewport; do not scale it down, regenerate, or upload a duplicate image.

Only use `insert_agent_generated_image` directly when the generated image is already at a public HTTPS URL. Never shrink or re-encode just to squeeze base64 through MCP JSON.

Codex-generated images should show up in Flare as generated media with an MCP binary upload ingest method. They should not be treated as ordinary user uploads, and they should not create a fake Flare backend generation job.

For generation requests, matching existing Assets are not a substitute for generation. They can be useful for recovery or reference, but the user expects a new generated bitmap unless they explicitly asked to reuse an existing asset.

Do not run shell searches for OAuth, token, or MCP credentials. Codex shell does not need the MCP OAuth token; it needs the short-lived `uploadToken` returned by `create_image_upload_session` or `get_image_upload_endpoint`.

Do not use the Flare app UI upload flow as a fallback for agent-generated images. Use MCP upload sessions so Assets, provenance, and canvas insertion stay deterministic.

## Codex Annotated Image Revision

Use this workflow when the user asks to revise an image from Flare canvas annotations, regardless of language.

1. Resolve the target `projectUrl` before opening or changing the in-app browser. In Codex desktop, assume the user wants to watch live canvas changes by default; do not wait for the user to explicitly ask. If a project id or URL is known, use `https://app.flare.design/projects/{projectId}` or the returned `projectUrl`. If no Flare project is open and no target URL or project id is known, use the Project Selection Behavior below before browser navigation. Once `projectUrl` is known, open or focus the in-app browser at that exact URL. Prefer an existing current Flare project tab only when it is already the target project. Do not open `app.flare.design`, `/projects`, the project list page, or project cards as an intermediate step. If browser-control tools are not available in the current Codex surface, say that explicitly before writing through MCP and verify with MCP snapshots instead. If Flare is not logged in, ask the user to log in in that browser and wait before continuing.
2. Call `check_client_setup` when exposed with `client: "codex"`, `skillName: "flare"`, and `installedSkillVersion: "0.1.20"`. If `updateRequired` is true, ask the user to run the returned `updateCommand` before continuing. If only `updateAvailable` is true, mention it once and continue when safe.
3. Call `get_image_annotation_context` with `projectId`. Pass `nodeId` or `annotationId` only when the user identified a specific image/session; otherwise let Flare resolve the active selection. The tool returns `annotatedImage` by default; set `includeAnnotatedImage: false` only when the client needs a smaller text-only response.
4. Treat `target.src` and `annotations` as the edit contract. Use `annotatedImage` as a visual reference, not as the only source of truth. Do not infer coordinates from a browser screenshot when the structured annotations are available.
5. Interpret annotations precisely:
   - `arrow.to` is the target-image point to revise.
   - `rect.bounds` and `ellipse.bounds` are target-image regions.
   - `text.x/y` is a target-image text anchor.
   - `arrow.from` and `labelPosition` are layout/context hints and may be outside the target image.
6. Generate the revised image with Codex image generation using the target image URL, structured annotations, optional `annotatedImage` composite preview, and original asset provenance. In the image prompt, state that this is a local edit: only change the annotated points, regions, or text requests; preserve unannotated composition, subject identity, background, lighting, camera angle, color palette, and style unless the user explicitly asks for a global redesign. Do not call Flare `create_generation_job`.
7. Save the revised bitmap as a local file. If the image generation result is a data URL or base64 string, decode it to a local file before any MCP call.
8. Upload the local file through `create_image_upload_session` and raw binary upload. Include provenance fields: `sourceClient: "codex"`, `generationPrompt`, `generationModel`, `generationTool`, and a concise `generationNotes` referencing the annotation session.
9. Call `insert_asset_image` with the returned `assetId` and the `suggestedPlacement` from the annotation context so the revised image appears beside the original. Keep the suggested size only when it represents the target/original image size; otherwise omit `width` and `height` to preserve the revised bitmap dimensions.
10. Verify through `get_canvas_snapshot` and the in-app browser.

## Browser Behavior

- After the target `projectUrl` is resolved, always open or focus the in-app browser before Flare canvas generation, insertion, annotation revision, motion, or other visible canvas edits in Codex desktop.
- When the target project is not already known, resolve `projectUrl` through MCP first. Do not open any Flare browser page until the exact `projectUrl` is known or created.
- If browser-control tools are unavailable, say that explicitly before writing through MCP instead of implying live browser verification happened.
- Prefer the current in-app browser tab only when it already shows the target `app.flare.design/projects/{projectId}` page; otherwise navigate directly to the target project URL when it is known.
- Never open `app.flare.design`, `/projects`, or the projects list as a stepping stone. Use MCP project URLs instead of clicking project cards.
- If no target project is clear, use the Project Selection Behavior below before opening the browser.
- If Flare is not logged in, ask the user to log in in the in-app browser and wait. Continue only after the browser session is authenticated.
- Keep the in-app browser visible through verification so the user can watch the canvas update without having to ask.
- Do not log in as a different user, search for tokens, or bypass auth.
- After inserting the image, refresh or wait for collab sync only if the image is not visible.
- If the snapshot shows the created node but the browser still does not show it, treat this as a placement issue. Move the existing node into the visible viewport with a canvas update instead of repeating generation, upload, or shrinking the image.
- If the browser and MCP snapshot disagree, trust MCP for saved state but inspect browser sync before retrying.

## Project Selection Behavior

- Call `list_projects` with `limit: 5` and `status: "active"` before opening the browser.
- If the response includes `selectionHint`, follow it. If Codex exposes interactive choices in the current surface, use those choices. Otherwise render the projects as a numbered text list.
- If one active project is returned, use it by default unless the user explicitly asked to choose or the context conflicts, then open its `selectedProjectUrl` or choice `projectUrl` directly.
- If multiple active projects are returned, wait for the user to pick by number, name, or project id. Do not choose the first project automatically. After selection, open that choice's `projectUrl` directly.
- If no active project is returned and `create_project` is available, call `create_project`, then open the returned `projectUrl` directly. If `create_project` is unavailable or permission is missing, ask the user to create or open a Flare project.

## Forbidden Path

Do not call `create_generation_job` for Codex-side image generation unless the user explicitly says to use Flare backend/canvas generation or to test the Flare generation queue.
