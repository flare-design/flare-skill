# Render Export

Use this when creating render/export jobs.

Dynamic source of truth:

```text
list_render_presets
```

Call it with `projectId` before choosing `presetId` or `variantId`.

## Create Render Job

Simple project render:

```json
{
  "projectId": "project-id",
  "presetId": "optional-project-preset-id",
  "variantId": "optional-template-variant-id"
}
```

Equivalent nested form:

```json
{
  "project": {
    "projectId": "project-id",
    "presetId": "optional-project-preset-id",
    "variantId": "optional-template-variant-id"
  }
}
```

Advanced render:

```json
{
  "snapshot": {
    "schemaVersion": "...",
    "documentId": "...",
    "documentName": "...",
    "exportedAt": "...",
    "editorSchema": {},
    "assets": [],
    "preset": {
      "id": "preset_1280x720_30fps",
      "format": "mp4",
      "width": 1280,
      "height": 720,
      "fps": 30,
      "includeAudio": true,
      "includeCaptions": true
    }
  }
}
```

## Preset Rules

Render presets are project-specific, not global.

Preset fields:

- `id`
- `format`: `mp4` or `png`
- `width`
- `height`
- `fps`
- `includeAudio`
- `includeCaptions`

If no `presetId` is provided, Flare derives a default preset from the current sequence frame size and uses 30fps. Audio and captions are included only when present and enabled in the project state.

## Workflow

1. Read project state if render target is ambiguous.
2. Call `list_render_presets` with `projectId`.
3. Choose `presetId` and `variantId` only if the project exposes matching ids.
4. Call `create_render_job`.
5. Poll `get_render_job` until `completed` or `failed`.
6. Report output URL or failure message.
