---
name: flare
description: Operate Flare projects through MCP for AI agents and MCP clients, including project discovery, live canvas edits, asset library management, agent-provided generated media insertion, HTML-to-frame import, Flare backend AI generation jobs, motion design, audio/captions, and render jobs. Use when the user mentions Flare, flare.design, a Flare project URL, canvas/artboard/frame/layer edits, generating or adding images/photos to a Flare canvas, importing HTML into a canvas, saving media to Assets, designing motion for a selected board, or rendering/exporting a Flare project.
metadata:
  version: "0.1.24"
---

# Flare

## Overview

Use this skill as the operating guide for Flare from any capable AI agent or MCP client. Route user intent to the correct Flare MCP tool, preserve live canvas state, and verify complex changes in the project after meaningful edits. Plain agent-side image generation uses the Fast Path below and ends after successful insertion unless the user asks for verification or recovery is needed.

Flare MCP is the source of truth for project, canvas, asset, generation, motion, and render operations. Agent-side media generation is a separate capability: when an agent or another MCP client has already generated or obtained an image, upload the local file through Flare's MCP upload-session flow and insert the resulting asset instead of starting a Flare generation job.

## Reference Map

Read only the references needed for the current request:

- `references/mcp-setup.md`: MCP installation, auth, scopes, environments, and tool discovery.
- `references/tool-routing.md`: Which MCP tool to use for each user intent.
- `references/canvas-geometry.md`: Coordinate system, parenting, frames, placement, and live collaboration rules.
- `references/generation-models.md`: Flare backend generation models, request fields, option limits, and validation rules.
- `references/motion-presets.md`: Layer/text motion preset ids, timing payloads, and motion-design guidance.
- `references/shader-presets.md`: Shader preset ids, categories, params, blend modes, and shader insertion.
- `references/canvas-patch-schema.md`: Raw `apply_canvas_patch` operation types and field shapes.
- `references/render-export.md`: Render/export presets, project variants, and render job request shape.
- `references/assets-and-media.md`: Asset kinds, uploads, public URL imports, and media limits.
- `references/workflows.md`: End-to-end flows for generated images, assets, motion, render, and canvas edits.
- `references/codex.md`: Codex-specific image generation and in-app browser workflow.
- `references/validation.md`: Verification checklist, failure recovery, and debugging signals.

## Operating Loop

1. Resolve the target environment, MCP project id, and browser project URL before visible edits. Prefer MCP-returned `projectId` for tool calls and `projectUrl` for browser navigation. MCP responses may also include `projectRouteId`, the prefix-free browser route id. If the target is unclear, follow the Project Selection Pattern below before opening a browser tab or editing.
2. Confirm Flare MCP is connected. Use tool discovery when available; if the Flare MCP server is missing, follow `references/mcp-setup.md`.
3. Before the first visible canvas write in a conversation, call `check_client_setup` when the tool is exposed. Pass `skillName: "flare"` and `installedSkillVersion: "0.1.24"`; include the client name when known. If `updateRequired` is true, ask the user to run the returned `updateCommand` before continuing. If only `updateAvailable` is true, mention the returned `updateCommand` once and continue when the current workflow is still compatible.
4. Read state before writing except on the plain image Fast Path. For canvas work, prefer `get_live_canvas_context` and `get_canvas_snapshot`; for broader context, use `export_project_snapshot`.
5. Query dynamic capability tools before using option-heavy features: `list_generation_models`, `list_motion_presets`, `list_shader_presets`, `list_canvas_patch_operations`, `list_media_capabilities`, or `list_render_presets`.
6. Choose the highest-level safe tool. Prefer specialized tools like `get_image_annotation_context` for annotated image revision, `create_image_upload_session` with `autoInsert` plus binary upload for local agent-generated files, `insert_agent_generated_image` for public URLs, `insert_html` for HTML-to-frame imports, `insert_shader_layer`, `apply_motion_design`, or `create_render_job` before raw `apply_canvas_patch`.
7. Apply the change with center-origin scene coordinates. Avoid unintended parenting.
8. Verify with a snapshot, job status, asset list, or browser view except on the plain image Fast Path and annotation-revision autoInsert path. Report concrete ids and any remaining uncertainty.

## Plain Image Fast Path

Use this path for ordinary requests to generate/create a photo, image, picture, or illustration and place it on a Flare canvas, when the user did not explicitly ask to use Flare backend generation, replace an existing layer, revise annotations, or inspect results.

This section is complete for the hot path. Do not open or read any `references/*` file before generation or upload unless the request leaves the fast path or an MCP call fails.

1. Start the agent/client image generation immediately.
2. In parallel with image generation, resolve the target project and open/focus its `projectUrl` directly in the browser when browser control is available. In Codex desktop, use the in-app browser control path, not Chrome DevTools or the user's external Chrome, and keep it visible through upload/insertion. Use `check_client_setup` and Project Selection Pattern only as needed for setup or target resolution; do not read workflow/upload references first.
3. Do not read canvas snapshots, list/search existing assets, inspect similar layers, or check whether a matching image already exists.
4. When the local image file is ready, call `create_image_upload_session` with provenance and `autoInsert: { projectId }`, then binary-upload the raw file bytes to the returned `uploadUrl`. The upload response should include both `assetId` and `autoInsert`.
5. Do not pass `width` or `height` unless the user explicitly asked for resizing, fitting, filling, or cropping.
6. End after the upload response reports successful `autoInsert`. Do not call `insert_asset_image`, `get_canvas_snapshot`, refresh the browser, center the viewport, or visually verify. Call `insert_asset_image` only if `autoInsert` is unavailable, missing from the upload response, or failed.

## Project Selection Pattern

Use this only when the user has not provided a project URL/id and no Flare project is already open in the active browser/client context. Do not open `app.flare.design`, `/projects`, or the project list while selecting; resolve a concrete `projectUrl` first.

1. Call `list_projects` with `limit: 5` and `status: "active"` before opening the browser.
2. If the response includes `selectionHint`, follow it. Use interactive choices only when the client actually supports them; otherwise show `selectionHint.textFallback` or a numbered text list.
3. If there are no active projects and `create_project` is available, call `create_project`, then open the returned `projectUrl` directly before editing. If `create_project` is unavailable or permission is missing, ask the user to create or open a project.
4. If there is exactly one active project, use it by default and open its `projectUrl` directly before visible edits unless the user explicitly asked to choose or the surrounding context points to another project.
5. If there are multiple active projects, do not infer the first result. Present up to five choices with name, id, updated time, and `projectUrl`, then wait for the user to reply with a number, name, or project id. After the user chooses, open that exact `projectUrl` directly.

## Project Id And URL Mapping

- MCP `projectId` values are canonical storage ids and normally include the `proj_` prefix.
- MCP responses may include `projectRouteId`, the prefix-free id used by browser routes.
- Flare editor browser URLs intentionally use `projectRouteId`: `proj_3f62a35f-...` opens at `https://app.flare.design/projects/3f62a35f-...`.
- When an MCP response includes `projectUrl`, use it directly for browser navigation. It should already be the prefix-free editor route.
- Prefer MCP-returned canonical `projectId` for tool calls. Current MCP servers also accept a prefix-free `/projects/{routeId}` value in `projectId` arguments and normalize it internally.
- When deriving a browser URL yourself, remove one leading `proj_` prefix before building `/projects/{routeId}`.

## Core Routing Rules

- **Plain "generate an image/photo" requests default to the Fast Path**: if the user asks to generate a picture/photo/illustration for Flare and does not explicitly say to use Flare backend generation, start agent/client image generation first and resolve/open the Flare project in parallel. Do not satisfy a generation request by reusing an existing Flare asset just because the prompt, name, or visual content looks similar. Do not search existing assets or inspect existing canvas images for ordinary generation. Existing assets may be reused only when the user explicitly asks to reuse/import an existing asset, or when recovering from a partially successful upload of the exact file generated in the current operation. If the result is a local file, call `create_image_upload_session` with provenance (`sourceClient`, `generationPrompt`, `generationModel`, `generationTool` when known) and `autoInsert: { projectId }`, then binary-upload it to Assets. Stop when the upload response confirms `autoInsert`; call `insert_asset_image` only when autoInsert is unavailable or failed. If the current client only exposes `get_image_upload_endpoint`, use its returned `uploadToken` and then call `insert_asset_image`.
- **The agent generated or obtained an image**: prefer a local file or public HTTPS URL. For local files, create an MCP upload session with `autoInsert` when a target project is known, then upload raw bytes; do not put base64 or data URLs in MCP JSON. If the image currently exists as a data URL or base64 string, save it to a local image file first, then upload that file. Use `insert_asset_image` only when `autoInsert` is unavailable or failed. Use `insert_agent_generated_image` only for public URLs. Do not call `create_generation_job`. Treat these assets as generated media entering Flare through MCP binary upload, not ordinary user uploads. When no exact `x/y` is required, omit `width` and `height` so Flare preserves the original image dimensions, and use smart placement or explicit visible coordinates so the inserted layer appears in the current/default viewport.
- **Revise an image from Flare annotations**: call `get_image_annotation_context` first. Treat `annotations` as the source of truth, `target.src` as the original image, and `annotatedImage` as visual reference only. Use arrow `to`, rect/ellipse `bounds`, and text `x/y` as 0..1 normalized target-image coordinates; use arrow `from` and `labelPosition` only as layout/context hints. This is a required image-generation/edit workflow: after reading annotation context, the next substantive action is image generation or image edit. Produce a fresh revised bitmap with the agent/client image generation or image-edit capability. Do not satisfy annotation revision by drawing canvas shapes/text, editing ordinary layers, reusing an existing asset, or using deterministic local scripts such as Pillow/canvas compositing as a substitute for generation. If no image generation/edit capability is available, say so instead of faking the edit. By default revise only the annotated points or regions and preserve unannotated composition, subject identity, background, lighting, camera angle, color palette, and style unless the user explicitly asks for a global redesign. Upload the generated local file with `create_image_upload_session` using the returned `suggestedAutoInsert` as `autoInsert`, then stop after the upload response confirms insertion beside the original. Do not call `insert_asset_image`, `get_canvas_snapshot`, or browser verification unless autoInsert fails, auth is missing, the user asks to verify, or the user says the result is not visible. Do not call `create_generation_job` unless the user explicitly asks for Flare backend generation.
- **Flare should generate media**: call `create_generation_job`, then poll `get_generation_job` or inspect `list_generation_jobs`, only when the user explicitly asks for Flare/canvas/backend generation or is testing that queue.
- **Need supported model/preset/options**: call the relevant `list_*` capability tool first. Treat these dynamic MCP responses as the source of truth, and use the reference files as fallback guidance.
- **Save user or agent media to Assets only**: create an upload session and binary-upload local files; call `save_image_asset_from_url` for public HTTPS URLs.
- **Insert an existing asset**: call `insert_asset_image`, `insert_asset_video`, or `add_audio_track` as appropriate.
- **Insert a shader/procedural background**: call `list_shader_presets`, then `insert_shader_layer`; use raw `insert_shader` only inside a controlled `apply_canvas_patch` batch.
- **Import HTML into the canvas**: call `insert_html` when the user provides or asks you to create HTML for the canvas. One HTML snippet or document becomes one editable root frame; supported text, simple boxes, and public HTTPS images become child layers. Use a browser or screenshot only as a layout QA aid before import; do not use screenshots as the import path unless the user explicitly needs pixel-perfect browser rendering.
- **Edit the live canvas**: use specialized insert/update/group/reorder/delete tools when possible; use `apply_canvas_patch` for controlled batches.
- **Design motion for a selected board/frame**: call `get_live_canvas_context`, resolve the target frame, then use `apply_motion_design`.
- **Render/export**: call `create_render_job`, then poll with `get_render_job`.

## HTML Import Quality

When creating HTML for `insert_html`, optimize for an editable Flare frame, not only for a nice browser screenshot. Flare's importer preserves common text, boxes, borders, fills, and public HTTPS images more reliably than complex browser effects.

- Choose an explicit frame size before writing markup. Use the user's requested size, or a conventional size for the artifact such as `1080x1440` for a vertical poster, `1920x1080` for a wide slide, or `1080x1080` for a square social post. Set `body { margin: 0; }` and make one root frame element with fixed `width`, `height`, `box-sizing: border-box`, and a deliberate background.
- Build the visual hierarchy first: a clear title area, primary message, supporting sections, and one obvious call to action when appropriate. Use generous padding, aligned edges, and a small set of type sizes rather than many unrelated sizes.
- Prefer simple, importer-friendly CSS: flex/grid for major structure, explicit `gap`/`padding`, solid fills, borders, rounded rectangles, and normal text blocks. Avoid relying on pseudo-elements, `filter`, `backdrop-filter`, blend modes, masks, `clip-path`, CSS animations, or deeply nested absolute positioning for essential content because these may not become clean editable layers.
- Use absolute positioning only for decorative or deliberately fixed elements. For information-heavy cards, prefer flex/grid so text expansion does not overlap badges, prices, chips, or buttons.
- Make text robust. Use a font stack that supports the content language, for example `Inter, -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif` for Chinese/English layouts. Manually break long hero titles or slogans, set sensible `line-height`, and verify that the longest words, Chinese phrases, salary ranges, and labels fit their containers.
- Use real visual assets when the request benefits from product, place, person, brand, mood, or scene imagery. If the user did not provide an image and web/image search is available, look for a subject-specific Unsplash image; prefer a direct public HTTPS image URL that can be used in HTML, and avoid generic stock filler that does not reinforce the message. For local, generated, or data-URL images, upload them through the asset workflow and insert them separately instead of embedding them in HTML.
- Avoid generic filler posters. Derive palette, iconography, copy emphasis, and section structure from the user's subject. A recruitment poster, for example, should usually include role, salary/range, location/work mode, experience level, benefits, responsibilities, requirements, and contact/apply information in a scannable hierarchy.
- Before calling `insert_html` for a nontrivial generated layout, render or inspect the HTML at the target dimensions when practical. Fix obvious browser-side issues first: clipped text, accidental scrollbars, overlapping chips, weak contrast, cramped margins, one-note color palettes, and decorative elements competing with the message.
- After import, verify the resulting frame with `get_canvas_snapshot` and, if visible, the browser. If Flare reflows or drops complex styling, repair with simpler editable layers or tell the user when pixel-perfect output would require rendering the HTML to an image and inserting that image instead.

## Canvas Defaults

- Treat canvas geometry as center-origin scene coordinates: `x/y` are the layer center, and `width/height` are unscaled layer size.
- For raw image-node patches, do not shrink `geometry.width/height` to resize a bitmap unless the user wants a crop. Preserve the image's intrinsic source `width/height` and use `scaleX/scaleY` for display sizing. Specialized insert tools should handle this when asset dimensions are known.
- Insert generated images as root canvas layers by default, not inside the selected frame/artboard.
- Pass `parentId` only when the user explicitly asks to place the layer inside a frame, group, or artboard.
- Use `anchorNodeId` and `placement` for relative positioning; do not assume this implies hierarchy.
- When inserting generated media without explicit user coordinates, keep it visible in the current/default viewport. On the Fast Path, do not verify this after insertion; if the user later says it is not visible, move the existing node into view with an update operation instead of regenerating or re-uploading.
- Do not pass `width` or `height` to `insert_asset_image` for a newly generated local image unless the user explicitly asks to resize it or to fit/fill a selected target. Omitting both dimensions preserves the original generated bitmap size.
- Preserve collaborator state and existing content. Read before editing and verify after writing except on the plain image Fast Path.

## Quality Bar

- Prefer user-visible, durable outputs: generated media should land in Assets and on the canvas when the user asks to place it.
- A created layer that is offscreen is an insertion/placement issue, not a generation failure. Inspect the node id and coordinates, then reposition that node.
- Keep generated image data as files. Do not convert local files to base64 for MCP arguments; if an image starts as base64 or data URL, write it to a local file before uploading.
- Do not search shell environment, config files, browser storage, or logs for the MCP OAuth access token. For local binary upload, use the short-lived `uploadToken` returned by `create_image_upload_session`.
- Keep provenance clear: agent-generated media should have generation/source metadata where the MCP tool supports it. Use `sourceClient` for the calling agent/client and `generationModel` for the actual image model; keep the prompt in `generationPrompt` instead of a note-only description.
- Avoid duplicate jobs or duplicate layers. If a prior operation partially succeeded, inspect assets, canvas nodes, and jobs before retrying.
- When a request is ambiguous between agent-side generation and Flare backend generation, default to agent-side generation for plain image/photo/illustration wording in any language. Use Flare backend generation only when the user explicitly mentions Flare, canvas, backend generation, generation jobs, or testing Flare's generation queue.
