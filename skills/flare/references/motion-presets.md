# Motion Presets

Use this when calling `apply_motion_design` or writing motion data through `apply_canvas_patch`.

Dynamic source of truth:

```text
list_motion_presets
```

Call it before using preset ids or motion-track fields.

## Layer Presets

- Basic: `none`, `fade`, `flash`
- Move: `slide-left`, `slide-right`, `slide-up`, `slide-down`, `push-left`, `push-right`, `push-up`, `push-down`, `drift-left`, `drift-right`, `drift-up`, `drift-down`, `shake`, `jitter`, `bounce`, `float`
- Scale & Rotate: `scale`, `zoom-out`, `pop`, `elastic-pop`, `pulse`, `breathe`, `spin-left`, `spin-right`, `tilt-left`, `tilt-right`, `swing`
- Reveal: `wipe-left`, `wipe-right`, `wipe-up`, `wipe-down`, `clip-slide-left`, `clip-slide-right`, `clip-slide-up`, `clip-slide-down`
- Path & Stroke: `draw-on`, `draw-off`, `morph`, `draw-loop`, `marching-ants`
- Media: `crop-left`, `crop-right`, `crop-up`, `crop-down`, `focus`, `defocus`, `depth-pulse`, `pan-left`, `pan-right`, `pan-up`, `pan-down`, `ken-burns`, `parallax-drift`

## Text Presets

- Basic: `none`, `fade`, `show-hide`, `typewriter`
- Move: `slide`, `rise`, `drop`, `drift`
- Scale & Rotate: `grow`, `shrink`, `pop`, `bounce`, `spin`, `swing`
- Focus & Style: `blur`, `blur-slide`, `highlight`, `focus-pull`, `tracking`

## Payload Shape

`apply_motion_design` takes a frame and layer patches:

```json
{
  "projectId": "project-id",
  "frameId": "frame-id",
  "durationMs": 5200,
  "fps": 30,
  "posterTimeMs": 1200,
  "replaceExisting": true,
  "layers": [
    {
      "nodeId": "title_1",
      "timeline": { "startMs": 200, "endMs": 3600 },
      "textAnimation": {
        "in": { "preset": "rise", "durationMs": 700 }
      },
      "motionClips": [
        {
          "id": "clip_title_in",
          "label": "Title rise",
          "kind": "in",
          "mode": "preset",
          "startMs": 200,
          "endMs": 900,
          "tracks": [],
          "enabled": true
        }
      ]
    }
  ]
}
```

Use `appendMotionClips: true` when adding clips without replacing existing clip data. Otherwise the default behavior replaces existing motion for the patched fields.

## Track Fields

Phases: `in`, `out`, `loop`, `emphasis`.

Easing: `ease-out`, `ease-in`, `ease-in-out`, `linear`, `hold`, `bezier`, `spring`.

Loop modes: `none`, `loop`, `ping-pong`.

Common numeric track properties include `translateX`, `translateY`, `scaleX`, `scaleY`, `rotation`, `opacity`, `blur`, crop insets, shape radii, `morphProgress`, `revealProgress`, `trim`, stroke dash fields, `pathProgress`, and `textCharSpacing`.

Color track properties: `fill`, `stroke`.

## Design Guidance

- Resolve the target frame first with `get_live_canvas_context` or explicit `frameId`.
- Use visual hierarchy: title first, supporting media next, decorative/background layers last.
- Stagger timing instead of starting every layer at 0.
- Use media presets only for image/video layers.
- Use path/stroke presets only for path or stroke-like layers.
- Do not apply generic animations to every layer unless the user asks for a uniform effect.
