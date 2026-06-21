---
name: flare
description: Operate Flare projects through MCP for AI agents and MCP clients, including project discovery, live canvas edits, asset library management, agent-provided generated media insertion, Flare backend AI generation jobs, motion design, audio/captions, and render jobs. Use when the user mentions Flare, flare.design, a Flare project URL, canvas/artboard/frame/layer edits, generating or adding images/photos to a Flare canvas, saving media to Assets, designing motion for a selected board, or rendering/exporting a Flare project.
metadata:
  version: "0.1.0"
---

# Flare

## Overview

Use this skill as the operating guide for Flare from any capable AI agent or MCP client. Route user intent to the correct Flare MCP tool, preserve live canvas state, and verify changes in the project after each meaningful edit.

Flare MCP is the source of truth for project, canvas, asset, generation, motion, and render operations. Agent-side media generation is a separate capability: when an agent or another MCP client has already generated or obtained an image, insert it with Flare's generated-image import tools instead of starting a Flare generation job.

## Reference Map

Read only the references needed for the current request:

- `references/mcp-setup.md`: MCP installation, auth, scopes, environments, and tool discovery.
- `references/tool-routing.md`: Which MCP tool to use for each user intent.
- `references/canvas-geometry.md`: Coordinate system, parenting, frames, placement, and live collaboration rules.
- `references/workflows.md`: End-to-end flows for generated images, assets, motion, render, and canvas edits.
- `references/codex.md`: Codex-specific image generation and in-app browser workflow.
- `references/validation.md`: Verification checklist, failure recovery, and debugging signals.

## Operating Loop

1. Identify the environment and project. If the user is already on a Flare project URL, use that project id. Otherwise list or ask for the target project.
2. Confirm Flare MCP is connected. Use tool discovery when available; if the Flare MCP server is missing, follow `references/mcp-setup.md`.
3. Read state before writing. For canvas work, prefer `get_live_canvas_context` and `get_canvas_snapshot`; for broader context, use `export_project_snapshot`.
4. Choose the highest-level safe tool. Prefer specialized tools like `insert_agent_generated_image` for agent-provided images, `insert_asset_image`, `apply_motion_design`, or `create_render_job` before raw `apply_canvas_patch`.
5. Apply the change with center-origin scene coordinates. Avoid unintended parenting.
6. Verify with a snapshot, job status, asset list, or browser view. Report concrete ids and any remaining uncertainty.

## Core Routing Rules

- **Plain "generate an image/photo" requests default to agent-side generation**: if the user asks to generate a picture/photo/illustration for Flare and does not explicitly say to use Flare backend generation, use the agent/client image generation capability first, then insert the result with `insert_agent_generated_image`.
- **The agent generated or obtained an image**: first obtain image bytes, a data URL, or a public HTTPS image URL; then call `insert_agent_generated_image`. If the server is older, fall back to `insert_codex_generated_image` or `insert_generated_image`. Do not call `create_generation_job`.
- **Flare should generate media**: call `create_generation_job`, then poll `get_generation_job` or inspect `list_generation_jobs`, only when the user explicitly asks for Flare/canvas/backend generation or is testing that queue.
- **Save user or agent media to Assets only**: call `save_image_asset` for base64/data URL or `save_image_asset_from_url` for public HTTPS URLs.
- **Insert an existing asset**: call `insert_asset_image`, `insert_asset_video`, or `add_audio_track` as appropriate.
- **Edit the live canvas**: use specialized insert/update/group/reorder/delete tools when possible; use `apply_canvas_patch` for controlled batches.
- **Design motion for a selected board/frame**: call `get_live_canvas_context`, resolve the target frame, then use `apply_motion_design`.
- **Render/export**: call `create_render_job`, then poll with `get_render_job`.

## Canvas Defaults

- Treat canvas geometry as center-origin scene coordinates: `x/y` are the layer center, and `width/height` are unscaled layer size.
- Insert generated images as root canvas layers by default, not inside the selected frame/artboard.
- Pass `parentId` only when the user explicitly asks to place the layer inside a frame, group, or artboard.
- Use `anchorNodeId` and `placement` for relative positioning; do not assume this implies hierarchy.
- Preserve collaborator state and existing content. Read before editing and verify after writing.

## Quality Bar

- Prefer user-visible, durable outputs: generated media should land in Assets and on the canvas when the user asks to place it.
- Keep provenance clear: agent-generated media should have generation/source metadata where the MCP tool supports it.
- Avoid duplicate jobs or duplicate layers. If a prior operation partially succeeded, inspect assets, canvas nodes, and jobs before retrying.
- When a request is ambiguous between agent-side generation and Flare backend generation, default to agent-side generation for plain "生成图片/照片/插图" wording. Use Flare backend generation only for explicit wording like "用画布生成", "Flare 后端生成", or "create_generation_job".
