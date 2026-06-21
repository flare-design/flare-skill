# Generation Models

Use this when calling Flare backend generation through `create_generation_job`.

Dynamic source of truth:

```text
list_generation_models
```

Call it before choosing a backend model or model-specific parameter. Treat its response as authoritative when it is available.

## Request Shape

`create_generation_job` takes:

```json
{
  "request": {
    "kind": "image",
    "mode": "text-to-image",
    "model": "nano-banana-2",
    "prompt": "..."
  },
  "visibility": "visible"
}
```

Common request fields:

- `kind`: `image` or `video`
- `mode`: image uses `text-to-image` or `image-to-image`; video uses `text-to-video` or `image-to-video`
- `model`: one supported model id
- `prompt`: required
- `negativePrompt`: optional
- `promptStyle`: `none`, `photoreal`, `cinematic`, `product`, `illustration`, `anime`, or `3d`
- `enhancePrompt`: optional boolean
- `aspectRatio`: model-specific
- `referenceAssets`: saved account image assets, each with `assetId` and matching `src`
- `endReferenceAsset`: saved account image asset for video end frame when supported

Image-only fields:

- `outputFormat`: `png`, `jpg`, or `webp`
- `width` / `height`: up to 4096 px per edge
- `outputCount`: model-specific
- `matchInputAspectRatio`: requires at least one reference image and model support
- `googleSearch` / `imageSearch`: model-specific grounding/search controls

Video-only fields:

- `outputFormat`: `mp4`
- `durationSeconds`: model-specific
- `resolution`: model-specific
- `referenceMode`: `frame`, `frames`, or `references`
- `generateAudio`: model-specific

## Defaults

- Default image model: `nano-banana-2`
- Default video model: `seedance-2.0`

## Image Models

- `nano-banana`: image, text/image input, standard ratios, up to 14 references, match input aspect ratio.
- `nano-banana-pro`: image, text/image input, 1024/2048/4096, up to 14 references, match input aspect ratio.
- `nano-banana-2`: image, text/image input, expanded ratios including `21:9`, 1024/2048/4096, up to 14 references, Google/Image Search.
- `gpt-image-2`: image, text/image input, ratios `1:1`, `3:2`, `2:3`, up to 10 references, output counts 1-10.
- `seedream-4`: image, text/image input, 1024/2048/4096, up to 10 references, output counts 1/2/4.
- `seedream-4.5`: image, text/image input, 2048/4096, up to 14 references, output counts 1/2/4.
- `seedream-5-lite`: image, text/image input, 2048/3072, up to 14 references, output counts 1/2/4.

## Video Models

- `seedance-1-pro`: video, text/image input, ratios `16:9`, `4:3`, `1:1`, `3:4`, `9:16`, `21:9`, `9:21`, resolutions 1080p/720p/480p, durations 5s/10s, one start reference and optional end reference.
- `seedance-1.5-pro`: video, like Seedance 1 Pro, supports native audio.
- `seedance-2.0`: video, ratios `16:9`, `4:3`, `1:1`, `3:4`, `9:16`, `21:9`, `adaptive`, resolutions 720p/480p, durations 5s/10s/15s, up to 9 references, native audio.
- `veo-3.1`: video, ratios `16:9`, `9:16`, resolutions 1080p/720p, durations 4s/6s/8s, up to 3 references, optional end reference, native audio. Reference-image mode only supports 16:9 and 8s, without end frame.
- `kling-3`: video, ratios `16:9`, `9:16`, `1:1`, resolutions 720p/1080p, durations 3s/5s/10s/15s, one start reference and optional end reference, native audio.

## Validation Limits

- Prompt limit: 2000 characters.
- Negative prompt limit: 500 characters.
- Maximum generated image dimension: 4096 px per edge.
- Active generation jobs per account: 4.
- Daily generation jobs per account: 50.
- Reference inputs must be saved account image assets; public URLs or local files must be saved/uploaded first.

## Routing Reminder

Do not call `create_generation_job` for plain agent-side image requests such as "Codex 生成图片放到画布". Use agent image generation plus upload session plus `insert_asset_image`.
