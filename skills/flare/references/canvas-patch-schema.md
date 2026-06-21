# Canvas Patch Schema

Use this when batching canvas edits with `apply_canvas_patch` or when no specialized tool exists.

Dynamic source of truth:

```text
list_canvas_patch_operations
```

Prefer specialized tools for single operations. Use raw patch for ordered batches, atomic edits, or unsupported combinations.

## Geometry

For `insert_*` and `create_frame`, `x/y` are center-origin canvas scene coordinates. `width/height` are unscaled layer size.

For raw image patches, do not shrink source `geometry.width/height` to resize a bitmap unless the user wants crop. Keep source dimensions and use scale when direct geometry is involved. Specialized image insertion handles this when asset dimensions are known.

## Operations

- `insert_text`: create text. Fields include `text`, `name`, `parentId`, `x/y/width/height`, `fill`, `fontFamily`, `fontSize`.
- `insert_rect`: create a rectangle. Fields include `fill`, `stroke`, `strokeWidth`, `rx`, `ry`.
- `insert_image`: create an image from account image asset. Requires `asset`; supports `mediaFit`.
- `insert_video`: create a video from account video asset. Requires `asset`; supports `autoplay`, `loop`, `muted`, `mediaFit`.
- `insert_shader`: create a procedural shader layer. Use `presetId`, optional `blendMode`, `animated`, `timeMs`, `params`.
- `create_frame`: create frame/artboard/scene. Fields include `frameName`, `headerHeight`, `fill`, `stroke`, `rx`, `ry`.
- `replace_media`: replace an image/video node with a new asset. Requires `id` and `asset`.
- `update_text`: update text, fill, font family/size/weight, and alignment.
- `set_layer_style`: merge a style object into an existing node.
- `reorder_node`: move a node before/after another node or into/out of a parent.
- `group_nodes`: create a group containing existing nodes.
- `add_audio_track`: add an audio asset to the timeline.
- `add_caption_track`: add manual or transcript caption cues.
- `apply_motion_design`: update frame composition and layer motion.
- `update_node`: generic node patch for `name`, `visible`, `locked`, `geometry`, `style`, `data`, `timeline`, `motionClips`, `animation`, or `textAnimation`.
- `delete_node`: delete a node by id.
- `set_canvas_background`: set canvas background color.

## Batching Rules

- Limit batches to 50 operations.
- Use stable ids when later operations refer to nodes created earlier in the same batch.
- Verify created, updated, and deleted node ids with `get_canvas_snapshot`.
- Avoid raw document rewrites when a specialized tool exists.
