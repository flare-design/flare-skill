# Codex Workflow

Use this reference only when the agent is running inside Codex and the user wants Flare canvas work with visible live feedback.

## Codex Image Generation to Flare Canvas

Use this workflow when the user wants Codex to generate the image itself and place it in Flare.

Typical intent patterns in any language:

- The user asks Codex or the current agent to create an image, photo, or illustration and place it in Flare.
- The user asks to use the agent/client's own image generation capability.
- The user asks not to use Flare, canvas, or backend generation.

1. Start Codex image generation immediately. In parallel, resolve the target `projectUrl` and open or focus the Codex in-app browser at that exact URL. In Codex desktop, assume the user wants to watch live canvas changes by default; do not wait for the user to explicitly ask. Use the in-app browser control path, not Chrome DevTools or the user's external Chrome. If an MCP response includes `projectUrl`, use it directly. MCP responses may also include canonical `projectId` for tool calls and prefix-free `projectRouteId` for browser routes. If only a full MCP `projectId` is known, build the browser URL by removing one leading `proj_` prefix, for example `proj_3f62...` becomes `https://app.flare.design/projects/3f62...`. If no Flare project is open and no target URL or project id is known, use the Project Selection Behavior below before browser navigation. Prefer an existing current Flare project tab only when it is already the target project. Do not open `app.flare.design`, `/projects`, the project list page, or project cards as an intermediate step. If Flare is not logged in, ask the user to log in in that browser and wait before upload/insertion.
2. Call `check_client_setup` when exposed with `client: "codex"`, `skillName: "flare"`, and `installedSkillVersion: "0.1.24"`. If `updateRequired` is true, ask the user to run the returned `updateCommand` before continuing. If only `updateAvailable` is true, mention it once and continue when safe.
3. Load and follow the installed `$imagegen` skill when available.
4. Generate a fresh bitmap with Codex's image generation path, not Flare's `create_generation_job`. Do not skip generation and reuse an existing Flare asset just because a similar prompt, name, or image already exists. Do not list/search Flare assets or inspect canvas layers for similar existing images. Reuse an existing asset only when the user explicitly asks for reuse/import, or when recovering from a partially successful upload of the exact file generated in this operation.
5. Keep the generated bitmap as a local file. Do not copy it to the workspace or inspect dimensions unless the user asked for a local backup or explicit sizing. Do not convert local files to base64 for MCP arguments. If the generation API returns a data URL or base64 string, decode and write it to a local image file before any Flare MCP call.
6. Identify `projectId` from the in-app browser URL, the user's explicit URL/id, or the selected recent project. Do not guess between multiple projects.
7. Call `create_image_upload_session` with the local file name, byte size, MIME type, provenance, and `autoInsert: { projectId }`. Use `sourceClient: "codex"`, keep legacy `sourceModel: "codex"` when accepted, put the actual image model in `generationModel`, put the prompt in `generationPrompt`, and put the generation surface in `generationTool` (for example `imagegen`) when known. If the current Codex thread only exposes `get_image_upload_endpoint`, call it and use its returned generic `uploadToken`.
8. Binary-upload the local file into Assets with the returned `uploadUrl` and `uploadToken`; use raw file bytes, `Content-Type`, `x-flare-file-size`, and optional `x-flare-filename`. Do not put the local image file, data URL, or base64 string into MCP JSON.
9. Stop after the upload response confirms `autoInsert`. Do not call `insert_asset_image`, `get_canvas_snapshot`, refresh the browser, center the viewport, or visually verify unless the user explicitly asks, an MCP call failed, auth is missing, or the user says the result is not visible.

Only use `insert_agent_generated_image` directly when the generated image is already at a public HTTPS URL. Never shrink or re-encode just to squeeze base64 through MCP JSON.

Codex-generated images should show up in Flare as generated media with an MCP binary upload ingest method. They should not be treated as ordinary user uploads, and they should not create a fake Flare backend generation job.

For generation requests, matching existing Assets are not a substitute for generation. Do not search for them on the fast path. They can be useful for recovery or reference, but the user expects a new generated bitmap unless they explicitly asked to reuse an existing asset.

Do not run shell searches for OAuth, token, or MCP credentials. Codex shell does not need the MCP OAuth token; it needs the short-lived `uploadToken` returned by `create_image_upload_session` or `get_image_upload_endpoint`.

Do not use the Flare app UI upload flow as a fallback for agent-generated images. Use MCP upload sessions so Assets, provenance, and canvas insertion stay deterministic.

## Codex Annotated Image Revision

Use this workflow when the user asks to revise an image from Flare canvas annotations, regardless of language.

1. Resolve the target `projectUrl` before opening or changing the in-app browser. In Codex desktop, assume the user wants to watch live canvas changes by default; do not wait for the user to explicitly ask. Use the Codex in-app browser control path, not Chrome DevTools or the user's external Chrome. If an MCP response includes `projectUrl`, use it directly. MCP responses may also include canonical `projectId` for tool calls and prefix-free `projectRouteId` for browser routes. If only a full MCP `projectId` is known, build the browser URL by removing one leading `proj_` prefix, for example `proj_3f62...` becomes `https://app.flare.design/projects/3f62...`. If no Flare project is open and no target URL or project id is known, use the Project Selection Behavior below before browser navigation. Once `projectUrl` is known, open or focus the in-app browser at that exact URL. Prefer an existing current Flare project tab only when it is already the target project. Do not open `app.flare.design`, `/projects`, the project list page, or project cards as an intermediate step. If browser-control tools are not available in the current Codex surface, say that explicitly before writing through MCP before continuing. If Flare is not logged in, ask the user to log in in that browser and wait before continuing.
2. Call `check_client_setup` when exposed with `client: "codex"`, `skillName: "flare"`, and `installedSkillVersion: "0.1.24"`. If `updateRequired` is true, ask the user to run the returned `updateCommand` before continuing. If only `updateAvailable` is true, mention it once and continue when safe.
3. Call `get_image_annotation_context` with `projectId`. Pass `nodeId` or `annotationId` only when the user identified a specific image/session; otherwise let Flare resolve the active selection. The tool returns `annotatedImage` by default; set `includeAnnotatedImage: false` only when the client needs a smaller text-only response.
4. Treat `target.src` and `annotations` as the edit contract. Use `annotatedImage` as a visual reference, not as the only source of truth. Do not infer coordinates from a browser screenshot when the structured annotations are available.
5. Interpret annotations precisely:
   - `arrow.to` is the target-image point to revise.
   - `rect.bounds` and `ellipse.bounds` are target-image regions.
   - `text.x/y` is a target-image text anchor.
   - `arrow.from` and `labelPosition` are layout/context hints and may be outside the target image.
6. Generate a fresh revised bitmap with Codex image generation or image-edit capability using the target image URL, structured annotations, optional `annotatedImage` composite preview, and original asset provenance. This step is mandatory for "按批注修改" / annotation revision; after `get_image_annotation_context`, the next substantive action should be image generation or image edit. Do not use Python/Pillow/local drawing scripts, canvas shapes/text, ordinary layer edits, or existing asset reuse as a substitute. If Codex image generation/edit is not available, say that the requested revision cannot be completed in this workflow instead of faking it. In the image prompt, state that this is a local edit: only change the annotated points, regions, or text requests; preserve unannotated composition, subject identity, background, lighting, camera angle, color palette, and style unless the user explicitly asks for a global redesign. Do not call Flare `create_generation_job`.
7. Save the revised bitmap as a local file. If the image generation result is a data URL or base64 string, decode it to a local file before any MCP call.
8. Upload the local file through `create_image_upload_session` and raw binary upload. Include provenance fields: `sourceClient: "codex"`, `generationPrompt`, `generationModel`, `generationTool`, and a concise `generationNotes` referencing the annotation session. Pass `autoInsert` from `suggestedAutoInsert` returned by `get_image_annotation_context` so the revised image appears beside the original.
9. Stop after the upload response confirms `autoInsert`. Do not call `insert_asset_image`, `get_canvas_snapshot`, refresh the browser, center the viewport, or visually verify unless the user explicitly asks, an MCP call failed, auth is missing, or the user says the result is not visible.

## Browser Behavior

- On the Codex image-generation fast path, start image generation immediately and open/focus the target browser project in parallel. After the target `projectUrl` is resolved, open or focus the Codex in-app browser before insertion when browser control is available. Do not use Chrome DevTools or the user's external Chrome as a substitute for the in-app browser.
- When the target project is not already known, resolve `projectUrl` through MCP first. Do not open any Flare browser page until the exact `projectUrl` is known or created.
- Browser `projectUrl` route ids omit one leading `proj_` prefix. Prefer the MCP-returned canonical `projectId` for tool calls and the MCP-returned `projectUrl` for browser navigation. Current MCP servers also accept a prefix-free `/projects/{routeId}` value in `projectId` arguments and normalize it internally.
- If browser-control tools are unavailable, continue through MCP after saying browser control is unavailable. Do not add MCP snapshot verification just to compensate on the fast path.
- Prefer the current in-app browser tab only when it already shows the target `app.flare.design/projects/{projectId}` page; otherwise navigate directly to the target project URL when it is known.
- Never open `app.flare.design`, `/projects`, or the projects list as a stepping stone. Use MCP project URLs instead of clicking project cards.
- If no target project is clear, use the Project Selection Behavior below before opening the browser.
- If Flare is not logged in, ask the user to log in in the in-app browser and wait. Continue only after the browser session is authenticated.
- Keep the in-app browser visible through insertion so the user can watch the canvas update without having to ask.
- Do not log in as a different user, search for tokens, or bypass auth.
- After inserting a fast-path or annotation-revision image, do not refresh, wait for sync, read a snapshot, or center the viewport. If the user later says the image is not visible, treat it as a placement/sync recovery task and move the existing node into view instead of repeating generation, upload, or shrinking the image.
- If the browser and MCP snapshot disagree during a non-fast-path task, trust MCP for saved state but inspect browser sync before retrying.

## Project Selection Behavior

- Call `list_projects` with `limit: 5` and `status: "active"` before opening the browser.
- If the response includes `selectionHint`, follow it. If Codex exposes interactive choices in the current surface, use those choices. Otherwise render the projects as a numbered text list.
- If one active project is returned, use it by default unless the user explicitly asked to choose or the context conflicts, then open its `selectedProjectUrl` or choice `projectUrl` directly.
- If multiple active projects are returned, wait for the user to pick by number, name, or project id. Do not choose the first project automatically. After selection, open that choice's `projectUrl` directly.
- If no active project is returned and `create_project` is available, call `create_project`, then open the returned `projectUrl` directly. If `create_project` is unavailable or permission is missing, ask the user to create or open a Flare project.

## Forbidden Path

Do not call `create_generation_job` for Codex-side image generation unless the user explicitly says to use Flare backend/canvas generation or to test the Flare generation queue.
