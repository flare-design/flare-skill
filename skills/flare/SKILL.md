---
name: flare
description: Operate Flare projects through MCP for AI agents and MCP clients, including project discovery, live canvas edits, asset library management, agent-provided generated media insertion, HTML-to-frame import, Flare backend AI generation jobs, motion design, audio/captions, and render jobs. Use when the user mentions Flare, flare.design, a Flare project URL, canvas/artboard/frame/layer edits, generating or adding images/photos to a Flare canvas, importing HTML into a canvas, saving media to Assets, designing motion for a selected board, or rendering/exporting a Flare project.
metadata:
  version: "0.1.11"
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

1. Identify the environment and project. If the user is already on a Flare project URL, use that project id. If the user gave a project URL or id, use it. If the target is unclear, call `list_projects` with `limit: 5` and `status: "active"`, show those recent projects in the conversation, and wait for the user to choose before editing. Do not guess the project.
2. Confirm Flare MCP is connected. Use tool discovery when available; if the Flare MCP server is missing, follow `references/mcp-setup.md`.
3. Before the first visible canvas write in a conversation, call `check_client_setup` when the tool is exposed. Pass `skillName: "flare"` and `installedSkillVersion: "0.1.11"`; include the client name when known. If `updateRequired` is true, ask the user to run the returned `updateCommand` before continuing. If only `updateAvailable` is true, mention the returned `updateCommand` once and continue when the current workflow is still compatible.
4. Read state before writing. For canvas work, prefer `get_live_canvas_context` and `get_canvas_snapshot`; for broader context, use `export_project_snapshot`.
5. Query dynamic capability tools before using option-heavy features: `list_generation_models`, `list_motion_presets`, `list_shader_presets`, `list_canvas_patch_operations`, `list_media_capabilities`, or `list_render_presets`.
6. Choose the highest-level safe tool. Prefer specialized tools like `get_image_annotation_context` for annotated image revision, `create_image_upload_session` plus binary upload plus `insert_asset_image` for local agent-generated files, `insert_agent_generated_image` for public URLs, `insert_html` for HTML-to-frame imports, `insert_shader_layer`, `apply_motion_design`, or `create_render_job` before raw `apply_canvas_patch`.
7. Apply the change with center-origin scene coordinates. Avoid unintended parenting.
8. Verify with a snapshot, job status, asset list, or browser view. Report concrete ids and any remaining uncertainty.

## Core Routing Rules

- **Plain "generate an image/photo" requests default to agent-side generation**: if the user asks to generate a picture/photo/illustration for Flare and does not explicitly say to use Flare backend generation, use the agent/client image generation capability first. If the result is a local file, call `create_image_upload_session` with provenance (`sourceClient`, `generationPrompt`, `generationModel`, `generationTool` when known), binary-upload it to Assets with the returned upload token, then insert it with `insert_asset_image`. If the current client only exposes `get_image_upload_endpoint`, use its returned `uploadToken`.
- **The agent generated or obtained an image**: prefer a local file or public HTTPS URL. For local files, create an MCP upload session or use the generic upload token returned by `get_image_upload_endpoint`, upload raw bytes, and then `insert_asset_image`; do not put base64 or data URLs in MCP JSON. If the image currently exists as a data URL or base64 string, save it to a local image file first, then upload that file. Use `insert_agent_generated_image` only for public URLs. Do not call `create_generation_job`. Treat these assets as generated media entering Flare through MCP binary upload, not ordinary user uploads.
- **Revise an image from Flare annotations**: call `get_image_annotation_context` first. Use the returned target image URL, structured text/arrow annotations with 0..1 normalized target points, `annotatedImage` composite preview when returned, and suggested placement. Generate the revised image with the agent/client image capability, upload the local file with `create_image_upload_session`, then call `insert_asset_image` next to the original. Do not call `create_generation_job` unless the user explicitly asks for Flare backend generation.
- **Flare should generate media**: call `create_generation_job`, then poll `get_generation_job` or inspect `list_generation_jobs`, only when the user explicitly asks for Flare/canvas/backend generation or is testing that queue.
- **Need supported model/preset/options**: call the relevant `list_*` capability tool first. Treat these dynamic MCP responses as the source of truth, and use the reference files as fallback guidance.
- **Save user or agent media to Assets only**: create an upload session and binary-upload local files; call `save_image_asset_from_url` for public HTTPS URLs.
- **Insert an existing asset**: call `insert_asset_image`, `insert_asset_video`, or `add_audio_track` as appropriate.
- **Insert a shader/procedural background**: call `list_shader_presets`, then `insert_shader_layer`; use raw `insert_shader` only inside a controlled `apply_canvas_patch` batch.
- **Import HTML into the canvas**: call `insert_html` when the user provides or asks you to create HTML for the canvas. One HTML snippet or document becomes one editable root frame; supported text, simple boxes, and public HTTPS images become child layers. Do not use Playwright or screenshots unless the user explicitly needs pixel-perfect browser rendering.
- **Edit the live canvas**: use specialized insert/update/group/reorder/delete tools when possible; use `apply_canvas_patch` for controlled batches.
- **Design motion for a selected board/frame**: call `get_live_canvas_context`, resolve the target frame, then use `apply_motion_design`.
- **Render/export**: call `create_render_job`, then poll with `get_render_job`.

## Canvas Defaults

- Treat canvas geometry as center-origin scene coordinates: `x/y` are the layer center, and `width/height` are unscaled layer size.
- For raw image-node patches, do not shrink `geometry.width/height` to resize a bitmap unless the user wants a crop. Preserve the image's intrinsic source `width/height` and use `scaleX/scaleY` for display sizing. Specialized insert tools should handle this when asset dimensions are known.
- Insert generated images as root canvas layers by default, not inside the selected frame/artboard.
- Pass `parentId` only when the user explicitly asks to place the layer inside a frame, group, or artboard.
- Use `anchorNodeId` and `placement` for relative positioning; do not assume this implies hierarchy.
- Preserve collaborator state and existing content. Read before editing and verify after writing.

## Quality Bar

- Prefer user-visible, durable outputs: generated media should land in Assets and on the canvas when the user asks to place it.
- Keep generated image data as files. Do not convert local files to base64 for MCP arguments; if an image starts as base64 or data URL, write it to a local file before uploading.
- Do not search shell environment, config files, browser storage, or logs for the MCP OAuth access token. For local binary upload, use the short-lived `uploadToken` returned by `create_image_upload_session`.
- Keep provenance clear: agent-generated media should have generation/source metadata where the MCP tool supports it. Use `sourceClient` for the calling agent/client and `generationModel` for the actual image model; keep the prompt in `generationPrompt` instead of a note-only description.
- Avoid duplicate jobs or duplicate layers. If a prior operation partially succeeded, inspect assets, canvas nodes, and jobs before retrying.
- When a request is ambiguous between agent-side generation and Flare backend generation, default to agent-side generation for plain image/photo/illustration wording. Treat Chinese phrases like `生成图片`, `生成照片`, and `生成插图` as agent-side generation unless the user explicitly asks for Flare backend generation with wording like `用画布生成`, `Flare 后端生成`, or `create_generation_job`.
