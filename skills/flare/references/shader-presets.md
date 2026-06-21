# Shader Presets

Use this when inserting or editing procedural shader layers.

Dynamic source of truth:

```text
list_shader_presets
```

Use `insert_shader_layer` for normal insertion. Use raw `insert_shader` only inside a controlled `apply_canvas_patch` batch.

## Insert Shape

```json
{
  "projectId": "project-id",
  "presetId": "meshAurora",
  "anchorNodeId": "frame-id",
  "placement": "fill",
  "blendMode": "screen",
  "params": {
    "uSpeed": 0.35,
    "grain": 0.08
  }
}
```

The renderer normalizes missing `view` and `params` from the selected preset. `insert_shader_layer` can use `x/y/width/height` or smart placement fields such as `anchorNodeId`, `placement`, `fitSelection`, and `margin`.

## Blend Modes

`normal`, `screen`, `multiply`, `overlay`, `lighter`.

## Categories And Presets

- Plane: `halo`, `meshBloom`, `lightLeak`, `watercolorWash`
- Orb: `pensive`, `interstella`, `violaOrientalis`, `sunset`
- Flow: `mint`, `nightyNight`, `universe`, `mandarin`, `cottonCandy`, `meshAurora`, `meshPearl`, `meshLime`, `ribbonFlow`
- Texture: `grainPeach`, `grainInk`, `grainLava`, `halftoneSwirl`, `grainient`, `silk`
- Fluid: `ferrofluid`, `liquidChrome`, `plasma`
- Aurora: `aurora`, `softAurora`
- Light: `lightRays`, `lightfall`
- Prismatic: `prism`, `prismaticBurst`
- Graphic: `gridScan`, `lineWaves`

## Parameter Families

Shader Gradient presets expose parameter definitions per preset. Plane and water-plane presets commonly support:

- `color1`, `color2`, `color3`
- `uSpeed`, `uStrength`, `uDensity`
- `brightness`, `grain`

Sphere shader-gradient presets additionally support:

- `uFrequency`
- `uAmplitude`

Fragment shader presets use a preset-specific subset of:

- `color1` through `color4`
- `speed`, `scale`, `intensity`, `density`, `frequency`, `amplitude`
- `brightness`, `distortion`, `softness`, `glow`, `grain`, `rotation`, `count`, `thickness`

Call `list_shader_presets` for the exact `parameters` and `parameterKeys` per preset before setting params. Use the returned type/range/step definitions when tuning numeric values.

## Usage Guidance

- Use `placement: "fill"` with a frame when the shader should be a background.
- Use `screen`, `overlay`, or `lighter` for light/aurora effects over imagery.
- Use `multiply` for texture or grain overlays.
- Put shader backgrounds behind text/media with `reorder_nodes` when layering matters.
- Keep shader params sparse unless the user requested precise procedural tuning.
