# Canvas Geometry

## Coordinate System

Flare canvas geometry uses center-origin scene coordinates:

- `x`: layer center x in canvas scene coordinates
- `y`: layer center y in canvas scene coordinates
- `width`: unscaled layer width
- `height`: unscaled layer height

This remains true even when a node has a parent. `parentId` changes hierarchy/clipping; it does not change the meaning of `x/y`.

## Root Layers vs Parent Layers

Default generated images to root canvas layers.

Pass `parentId` only when the user explicitly asks to put the layer inside a frame, group, or artboard. Selection alone is not enough evidence that the generated image should become a child of the selected frame.

Bad default:

```json
{ "parentId": "selected_frame_1" }
```

Good default:

```json
{ "anchorNodeId": "selected_frame_1", "placement": "fill" }
```

The second form can use a frame for placement and size while still inserting the image as a root canvas layer.

## Placement

Use `anchorNodeId` and `placement` for relative layout:

- `center`: align centers
- `fill`: match anchor bounds
- `left`, `right`, `above`, `below`: place near the anchor with optional margin

Use explicit `x/y/width/height` when the user gives exact geometry or when validating a known target.

When exact user coordinates are not required, keep inserted media inside the current/default visible viewport. If smart placement creates a node but the browser does not show it, inspect the snapshot coordinates and move that existing node into view instead of duplicating the asset.

## Live Collaboration

Before writing to a live project:

1. Open the MCP-returned `projectUrl` exactly as returned. It may include `?agentSession=...`; this binds the browser tab's live selection and viewport to the current agent operation.
2. Preserve the returned `agentSessionId` and pass it to `get_live_canvas_context`, annotation context tools, placement tools, and `create_image_upload_session.autoInsert` when available.
3. Read `get_live_canvas_context` to understand active selection, viewport, and text editing state. With `agentSessionId`, this should resolve the bound browser tab instead of another collaborator's tab.
4. Read `get_canvas_snapshot` or `export_project_snapshot` when node ids or hierarchy matter.
5. Write through MCP so the collab room syncs to active clients.
6. Verify the created or updated node ids unless the workflow explicitly says to stop after upload auto-insert.

Avoid editing stale snapshots directly. If the browser shows different content from the snapshot, reload state before retrying.

Do not infer live selection from another collaborator just because they share the same account or project. Use `agentSessionId` first, explicit `clientId` second, and only fall back to generic active presence when no bound browser context exists.

## Sizing Defaults

When no size is specified:

- Use intrinsic image dimensions if the MCP tool can detect them.
- Otherwise choose a conservative visible size near the current viewport or selected frame.
- Keep aspect ratio unless the user requests cropping, fill, or distortion.

For `insert_asset_image`, omit `width` and `height` when placing a newly generated image unless the user explicitly requests a display size, fill, crop, or target match. Preserve visibility through `x/y`, `anchorNodeId`, `placement`, and browser viewport control instead of shrinking the bitmap.

For raw image-node edits, distinguish source geometry from display size. A Fabric image node's `geometry.width/height` can become the crop frame when smaller than the source bitmap. To resize without cropping, keep `geometry.width/height` at the intrinsic image dimensions and set `scaleX/scaleY` so the rendered bounds match the desired display size.

For generated images placed on a board/frame, prefer `placement: "fill"` with `anchorNodeId` and no `parentId`.
