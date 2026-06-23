---
name: flare
description: Operate Flare projects through MCP for AI agents and MCP clients, including project discovery, live canvas edits, asset library management, agent-provided generated media insertion, HTML-to-frame import, Flare backend AI generation jobs, motion design, audio/captions, and render jobs. Use when the user mentions Flare, flare.design, a Flare project URL, canvas/artboard/frame/layer edits, generating or adding images/photos to a Flare canvas, importing HTML into a canvas, saving media to Assets, designing motion for a selected board, or rendering/exporting a Flare project.
metadata:
  version: "0.1.18"
---

# Flare

## Overview

Use this skill as the operating guide for Flare from any capable AI agent or MCP client. Route user intent to the correct Flare MCP tool, preserve live canvas state, and verify changes in the project after each meaningful edit.

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

1. Identify the environment and project. If the user is already on a Flare project URL, use that project id. If the user gave a project URL or id, use it. If the target is unclear, follow the Project Selection Pattern below before editing.
2. Confirm Flare MCP is connected. Use tool discovery when available; if the Flare MCP server is missing, follow `references/mcp-setup.md`.
3. Before the first visible canvas write in a conversation, call `check_client_setup` when the tool is exposed. Pass `skillName: "flare"` and `installedSkillVersion: "0.1.18"`; include the client name when known. If `updateRequired` is true, ask the user to run the returned `updateCommand` before continuing. If only `updateAvailable` is true, mention the returned `updateCommand` once and continue when the current workflow is still compatible.
4. Read state before writing. For canvas work, prefer `get_live_canvas_context` and `get_canvas_snapshot`; for broader context, use `export_project_snapshot`.
5. Query dynamic capability tools before using option-heavy features: `list_generation_models`, `list_motion_presets`, `list_shader_presets`, `list_canvas_patch_operations`, `list_media_capabilities`, or `list_render_presets`.
6. Choose the highest-level safe tool. Prefer specialized tools like `get_image_annotation_context` for annotated image revision, `create_image_upload_session` plus binary upload plus `insert_asset_image` for local agent-generated files, `insert_agent_generated_image` for public URLs, `insert_html` for HTML-to-frame imports, `insert_shader_layer`, `apply_motion_design`, or `create_render_job` before raw `apply_canvas_patch`.
7. Apply the change with center-origin scene coordinates. Avoid unintended parenting.
8. Verify with a snapshot, job status, asset list, or browser view. Report concrete ids and any remaining uncertainty.

## Project Selection Pattern

Use this only when the user has not provided a project URL/id and no Flare project is already open in the active browser/client context.

1. Call `list_projects` with `limit: 5` and `status: "active"`.
2. If the response includes `selectionHint`, follow it. Use interactive choices only when the client actually supports them; otherwise show `selectionHint.textFallback` or a numbered text list.
3. If there are no active projects, ask the user to open or create one.
4. If there is exactly one active project, state that it will be used by default and continue unless the user explicitly asked to choose or the surrounding context points to another project.
5. If there are multiple active projects, do not infer the first result. Present up to five choices with name, id, and updated time, then wait for the user to reply with a number, name, or project id.

## Core Routing Rules

- **Plain "generate an image/photo" requests default to agent-side generation**: if the user asks to generate a picture/photo/illustration for Flare and does not explicitly say to use Flare backend generation, use the agent/client image generation capability first. Do not satisfy a generation request by reusing an existing Flare asset just because the prompt, name, or visual content looks similar. Existing assets may be reused only when the user explicitly asks to reuse/import an existing asset, or when recovering from a partially successful upload of the exact file generated in the current operation. If the result is a local file, call `create_image_upload_session` with provenance (`sourceClient`, `generationPrompt`, `generationModel`, `generationTool` when known), binary-upload it to Assets with the returned upload token, then insert it with `insert_asset_image`. If the current client only exposes `get_image_upload_endpoint`, use its returned `uploadToken`.
- **The agent generated or obtained an image**: prefer a local file or public HTTPS URL. For local files, create an MCP upload session or use the generic upload token returned by `get_image_upload_endpoint`, upload raw bytes, and then `insert_asset_image`; do not put base64 or data URLs in MCP JSON. If the image currently exists as a data URL or base64 string, save it to a local image file first, then upload that file. Use `insert_agent_generated_image` only for public URLs. Do not call `create_generation_job`. Treat these assets as generated media entering Flare through MCP binary upload, not ordinary user uploads. When no exact `x/y` is required, omit `width` and `height` so Flare preserves the original image dimensions, and use smart placement or explicit visible coordinates so the inserted layer appears in the current/default viewport.
- **Revise an image from Flare annotations**: call `get_image_annotation_context` first. Treat `annotations` as the source of truth, `target.src` as the original image, and `annotatedImage` as visual reference only. Use arrow `to`, rect/ellipse `bounds`, and text `x/y` as 0..1 normalized target-image coordinates; use arrow `from` and `labelPosition` only as layout/context hints. By default this is a local edit workflow: revise only the annotated points or regions and preserve unannotated composition, subject identity, background, lighting, camera angle, color palette, and style unless the user explicitly asks for a global redesign. Generate the revised image with the agent/client image capability, upload the local file with `create_image_upload_session`, then call `insert_asset_image` next to the original using `suggestedPlacement`. Do not call `create_generation_job` unless the user explicitly asks for Flare backend generation.
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
- When inserting generated media without explicit user coordinates, keep it visible in the current/default viewport. If a snapshot confirms creation but the browser does not show the node, move the existing node into view with an update operation instead of regenerating or re-uploading.
- Do not pass `width` or `height` to `insert_asset_image` for a newly generated local image unless the user explicitly asks to resize it or to fit/fill a selected target. Omitting both dimensions preserves the original generated bitmap size.
- Preserve collaborator state and existing content. Read before editing and verify after writing.

## Quality Bar

- Prefer user-visible, durable outputs: generated media should land in Assets and on the canvas when the user asks to place it.
- A created layer that is offscreen is an insertion/placement issue, not a generation failure. Inspect the node id and coordinates, then reposition that node.
- Keep generated image data as files. Do not convert local files to base64 for MCP arguments; if an image starts as base64 or data URL, write it to a local file before uploading.
- Do not search shell environment, config files, browser storage, or logs for the MCP OAuth access token. For local binary upload, use the short-lived `uploadToken` returned by `create_image_upload_session`.
- Keep provenance clear: agent-generated media should have generation/source metadata where the MCP tool supports it. Use `sourceClient` for the calling agent/client and `generationModel` for the actual image model; keep the prompt in `generationPrompt` instead of a note-only description.
- Avoid duplicate jobs or duplicate layers. If a prior operation partially succeeded, inspect assets, canvas nodes, and jobs before retrying.
- When a request is ambiguous between agent-side generation and Flare backend generation, default to agent-side generation for plain image/photo/illustration wording in any language. Use Flare backend generation only when the user explicitly mentions Flare, canvas, backend generation, generation jobs, or testing Flare's generation queue.
