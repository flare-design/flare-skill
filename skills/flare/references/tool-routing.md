# Tool Routing

## Decision Table

| User intent | Preferred tool path | Avoid |
| --- | --- | --- |
| "/flare 生成一张图片/照片/插图" | Use client/agent image generation, save the result as a local file, call `create_image_upload_session` or fallback `get_image_upload_endpoint`, binary-upload it, then `insert_asset_image` | `create_generation_job` |
| "Agent/Claude/Codex 生成图片放到画布" | Use the client or agent's image generation capability, save a local file, call `create_image_upload_session` or fallback `get_image_upload_endpoint`, binary-upload it, then `insert_asset_image` | `create_generation_job` |
| "把这张本地图片存到素材" | Call `create_image_upload_session` or fallback `get_image_upload_endpoint`, then binary-upload the local file with the returned `uploadToken` | Raw canvas patch without asset |
| "把这个图片 URL 存到素材" | `save_image_asset_from_url` | Raw canvas patch without asset |
| "把已有素材放到画布" | `insert_asset_image` or `insert_asset_video` | Re-uploading the same asset |
| "用 Flare/画布 AI 生图" | `create_generation_job`, then `get_generation_job` | Client-side generation unless asked |
| "看当前选中了什么" | `get_live_canvas_context` | Guessing from screenshot only |
| "读整个项目/画布状态" | `export_project_snapshot` or `get_canvas_snapshot` | Writing before reading |
| "添加文字/矩形/frame" | `insert_text`, `insert_rect`, `create_frame` | Freeform patch unless batching |
| "改文字/样式" | `update_text`, `set_layer_style` | Delete and recreate unless requested |
| "调整层级/组合/删除" | `reorder_nodes`, `group_nodes`, `delete_nodes` | Blind raw document rewrites |
| "给选中画板加动效" | `get_live_canvas_context`, then `apply_motion_design` | Applying motion to an unspecified frame |
| "加音频/字幕" | `add_audio_track`, `add_caption_track` | Treating audio as canvas image/video |
| "渲染/导出视频" | `create_render_job`, then `get_render_job` | Browser screenshot as final render |

## Agent-Side vs Flare Generation

Default to agent-side/client-side generation when the user asks to generate an image/photo/illustration and does not explicitly request Flare backend generation. This includes:

- "/flare 生成一张照片"
- "/flare 生成图片放到画布"
- "生成小猪佩奇上幼儿园的照片"
- "Codex 内生图"
- "Claude 生成一张图"
- "agent 自己生成图片"
- "你生成一张图再放到画布"
- "不要调用画布生图"
- "用你自己的生图能力"

Use Flare generation job only when the user says:

- "用画布的生图"
- "用 Flare 后端生成"
- "创建一个 generation job"
- "测试 Flare generation queue"
- They are testing the app's built-in generation pipeline

If wording is mixed, ask one concise clarification before spending generation credits. Do not choose `create_generation_job` just because the Chinese verb is "生成".

## Asset Provenance

For agent-generated images:

- Prefer local-file `create_image_upload_session` plus binary upload plus `insert_asset_image`; use `get_image_upload_endpoint` as a fallback when the current client has stale tool metadata. Use `insert_agent_generated_image` only for public HTTPS `imageUrl` or `url`.
- If an agent has image data as a data URL or base64 string, first save it to a local image file; do not pass it in MCP JSON.
- Do not search for or expose MCP OAuth tokens. Use the short-lived `uploadToken` from `create_image_upload_session` for local file bytes.
- Set `sourceModel` to the client or model name when known; otherwise let the MCP server use its default provenance.
- Include a useful `fileName` and `name` when the user gave a subject.

For user-provided or external images:

- Use asset save tools first when the user asks to keep the file.
- Use insert tools after saving when the user asks to place it on canvas.

## Raw Patch Use

Use `apply_canvas_patch` when:

- Multiple canvas operations must be atomic or ordered together.
- No specialized tool exists for the intended operation.
- You already inspected the current document and can target stable node ids.

Do not use raw patch as the first choice for generated image insertion, existing asset insertion, motion design, or render jobs.
