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

## Live Collaboration

Before writing to a live project:

1. Read `get_live_canvas_context` to understand active selection, viewport, and text editing state.
2. Read `get_canvas_snapshot` or `export_project_snapshot` when node ids or hierarchy matter.
3. Write through MCP so the collab room syncs to active clients.
4. Verify the created or updated node ids.

Avoid editing stale snapshots directly. If the browser shows different content from the snapshot, reload state before retrying.

## Sizing Defaults

When no size is specified:

- Use intrinsic image dimensions if the MCP tool can detect them.
- Otherwise choose a conservative visible size near the current viewport or selected frame.
- Keep aspect ratio unless the user requests cropping, fill, or distortion.

For generated images placed on a board/frame, prefer `placement: "fill"` with `anchorNodeId` and no `parentId`.
