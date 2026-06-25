# Tool Routing

## Decision Table

| User intent pattern                                     | Preferred tool path                                                                                                                                                                                                  | Avoid                                                                                                                              |
| ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Generate/create an image, photo, or illustration        | Use client/agent image generation, save the result as a local file, call `create_image_upload_session` with `autoInsert`, then binary-upload it and stop when autoInsert succeeds                                    | `create_generation_job`, final snapshot verification                                                                               |
| The current agent/Claude/Codex should generate it       | Use the client or agent's image generation capability, save a local file, call `create_image_upload_session` with `autoInsert`, then binary-upload it and stop when autoInsert succeeds                              | `create_generation_job`, final snapshot verification                                                                               |
| Revise an image from Flare canvas annotations           | Call `get_image_annotation_context`, use agent/client image generation or image-edit capability to create a fresh revised bitmap, then upload with `autoInsert` from `suggestedAutoInsert` and stop when it succeeds | `create_generation_job`, platform annotation generation, canvas shapes/text, local deterministic image scripts, final verification |
| Save a local image file to Assets                       | Call `create_image_upload_session` or fallback `get_image_upload_endpoint`, then binary-upload the local file with the returned `uploadToken`                                                                        | Raw canvas patch without asset                                                                                                     |
| Save a public image URL to Assets                       | `save_image_asset_from_url`                                                                                                                                                                                          | Raw canvas patch without asset                                                                                                     |
| Place an existing asset on the canvas                   | `insert_asset_image` or `insert_asset_video`                                                                                                                                                                         | Re-uploading the same asset                                                                                                        |
| Ask for supported models, parameters, presets, or tools | `list_generation_models`, `list_motion_presets`, `list_shader_presets`, `list_canvas_patch_operations`, `list_media_capabilities`, or `list_render_presets` depending on the feature                                 | Guessing from old skill text                                                                                                       |
| Explicitly use Flare/canvas/backend generation          | `list_generation_models`, then `create_generation_job`, then `get_generation_job`                                                                                                                                    | Client-side generation unless asked                                                                                                |
| Inspect current selection                               | `get_live_canvas_context`                                                                                                                                                                                            | Guessing from screenshot only                                                                                                      |
| Read full project or canvas state                       | `export_project_snapshot` or `get_canvas_snapshot`                                                                                                                                                                   | Writing before reading                                                                                                             |
| Add text, rectangle, or frame                           | `insert_text`, `insert_rect`, `create_frame`                                                                                                                                                                         | Freeform patch unless batching                                                                                                     |
| Import an HTML snippet/document into the canvas         | `insert_html`; one HTML snippet or document becomes one editable root frame                                                                                                                                          | Playwright/browser screenshot unless the user needs pixel-perfect rendering                                                        |
| Add shader/procedural background                        | `list_shader_presets`, then `insert_shader_layer`                                                                                                                                                                    | Raw shader patch without checking supported preset ids                                                                             |
| Edit text or visual style                               | `update_text`, `set_layer_style`                                                                                                                                                                                     | Delete and recreate unless requested                                                                                               |
| Reorder, group, or delete layers                        | `reorder_nodes`, `group_nodes`, `delete_nodes`                                                                                                                                                                       | Blind raw document rewrites                                                                                                        |
| Add motion to a selected board/frame                    | `get_live_canvas_context`, `list_motion_presets`, then `apply_motion_design`                                                                                                                                         | Applying motion to an unspecified frame                                                                                            |
| Add audio or captions                                   | `add_audio_track`, `add_caption_track`                                                                                                                                                                               | Treating audio as canvas image/video                                                                                               |
| Render/export video                                     | `list_render_presets`, then `create_render_job`, then `get_render_job`                                                                                                                                               | Browser screenshot as final render                                                                                                 |

## Agent-Side vs Flare Generation

Default to agent-side/client-side generation when the user asks to generate an image/photo/illustration and does not explicitly request Flare backend generation. Interpret equivalent wording across languages instead of enumerating locale-specific trigger phrases. This includes intents such as:

- Generate/create a picture, photo, illustration, or revised image.
- Place an agent-generated image on the Flare canvas.
- Use the current agent/client's own image generation capability.
- Do not use Flare/canvas/backend generation.
- Revise an image from Flare annotations.

Use Flare generation job only when the user says:

- Use Flare, canvas, or backend generation.
- Create a generation job.
- Test the Flare generation queue or the app's built-in generation pipeline.

If wording is mixed, ask one concise clarification before spending generation credits. Do not choose `create_generation_job` just because the user used a generic word for "generate" in any language.

Before calling `create_generation_job`, call `list_generation_models` and use its defaults, model-specific fields, output limits, and policy limits as the source of truth. Skill reference files are fallback guidance only.

## Asset Provenance

For agent-generated images:

- Prefer local-file `create_image_upload_session` with `autoInsert` plus binary upload; use `insert_asset_image` only when autoInsert is unavailable or failed. Use `get_image_upload_endpoint` as a fallback when the current client has stale tool metadata. Use `insert_agent_generated_image` only for public HTTPS `imageUrl` or `url`.
- Use the exact path returned by the generation/edit tool. Do not shell-search for the newest generated file or copy it into a workspace assets folder before upload.
- If an agent has image data as a data URL or base64 string, first save it to a local image file; do not pass it in MCP JSON.
- Do not search for or expose MCP OAuth tokens. Use the short-lived `uploadToken` from `create_image_upload_session` for local file bytes.
- Set `sourceClient` to the calling agent/client when known, such as `codex` or `claude`.
- Set `generationModel` to the actual image generation model when known.
- Keep `sourceModel` only as a legacy source hint when the tool accepts it; do not use it as the primary place for the real image model.
- Include a useful `fileName` and `name` when the user gave a subject.

For user-provided or external images:

- Use asset save tools first when the user asks to keep the file.
- Use insert tools after saving when the user asks to place it on canvas.

## Raw Patch Use

Use `apply_canvas_patch` when:

- Multiple canvas operations must be atomic or ordered together.
- No specialized tool exists for the intended operation.
- You already inspected the current document and can target stable node ids.

Call `list_canvas_patch_operations` before writing unfamiliar raw operations. Prefer `insert_shader_layer` over raw `insert_shader` unless the shader must be part of an atomic multi-operation batch.

Do not use raw patch as the first choice for generated image insertion, existing asset insertion, HTML import, shader insertion, motion design, or render jobs.
