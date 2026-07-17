---
name: "flat-imagegen"
description: "Default image generation skill for natural-language image requests. Use when the user asks to generate, create, draw, design, make, or edit images, posters, covers, banners, stickers, icons, product shots, illustrations, concept art, or reference-based image edits. Prefer this skill for plain Chinese/English prompts like 生图, 生成图片, 创作图片, 做海报, 画一张, create an image, generate a poster, or edit this image via Flat/gateway OpenAI-compatible APIs and unofficial models."
---

# Flat Imagegen

Generates or edits images by calling the project's Flat OpenAI-compatible image endpoint.

## When to use

Trigger on plain natural-language image requests, even if the user does not name this skill:

- 生图 / 生成图片 / 创作图片 / 做图 / 画一张 / 出一张图
- 做海报 / 做封面 / 做 banner / 做 sticker / 做 icon / 产品图 / 插画
- generate/create/draw/design/make an image, poster, cover, banner, sticker, icon, product shot, illustration
- edit/modify/restyle an existing image with a reference file
- any raster image request that should go through Flat / gateway OpenAI-compatible APIs, especially unofficial models such as `gpt-image-2`

Default behavior for these prompts:

1. Prefer this skill automatically.
2. Do not ask the user to name `flat-imagegen` first.
3. Run `scripts/flat_image_gen.mjs` in default `images` mode unless the user explicitly asks for another provider.

## When not to use

- The task is better handled by editing repo-native SVG, HTML/CSS, or canvas assets directly
- The user explicitly asks for a different provider/skill path
- The request is pure text, code, or layout work with no raster image output

## Top-level mode

This skill uses one execution path only:

- `scripts/flat_image_gen.mjs`

The script:

- defaults to `POST /v1/images/generations`
- supports both text-to-image and reference-image editing
- keeps optional `--api-mode responses` for gateways that support Responses + `image_generation`
- writes final images to disk

## Defaults

- Base URL: `https://gateway.aimsg.uk/v1`
- Default API mode: `images`
- Default image model: `gpt-image-2`
- Default responses model (compat mode only): `gpt-5.5`
- Default output directory: `data/generated-images`

Authentication is required:

- The script reads the API key from `auth.json` in the skill root directory (preferred), falling back to the `FLAT_API_KEY` environment variable.

**Before running the script, check if `auth.json` exists in the skill root.** If it does not exist or the key inside is empty, ask the user for their API key, then create or update the file:

```json
{
  "FLAT_API_KEY": "the_key_from_user"
}
```

Write the file using the Write tool. Do not commit `auth.json` to version control — it is listed in `.gitignore`.

Never hardcode secrets into any other committed files.

## Workflow

1. Treat plain image language as enough signal to use this skill. Do not require the user to say `flat-imagegen`.
2. Decide whether the request is generate or edit.
3. Collect the prompt, output size, output format, count, and any reference images. If the user only gives a short natural-language request, expand it into a clear generation prompt and proceed.
4. Check `auth.json` without displaying or logging its secret. If credentials are absent, request the key and store it only in the skill root or environment.
5. If the user names a destination path, pass `--output-dir` so the final asset lands in the workspace where they want it. Otherwise use the default output directory.
6. Run the bundled script in default `images` mode. Only use `--api-mode responses` when the gateway explicitly supports it.
7. Inspect the saved result if the task depends on visual correctness.
8. Report the final file path(s), image model used, and any revised prompt returned by the API.

## Privacy and provenance

- Reference images are Base64-encoded and uploaded to the third-party Flat endpoint. Do not send private, confidential, or unpublished assets without making that data transfer clear to the user.
- The source repository may not declare an open-source license. Treat the code as usable only within the permissions actually granted by its owner; do not assume unrestricted redistribution rights.

## Reference-image editing

- For one reference image, pass one `--image <path>`
- For multiple references, repeat `--image <path>` for each file
- In default images mode, local files are encoded and sent as image-to-image input
- In responses mode, local files are converted to `data:` URLs and sent as `input_image`

## Transparency

- Default image generation uses `gpt-image-2`
- If the user explicitly asks for true native transparency, switch to:
  - `--image-model gpt-image-1.5`
  - `--background transparent`
  - `--output-format png` or `webp`
- Do not silently switch models for transparency-sensitive work; mention the model change

## Command patterns

Generate:

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "A friendly robot mascot" `
  --size 1024x1024 `
  --output-format png
```

Edit with one reference:

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "Keep the subject, replace the background with a bright studio setup" `
  --image .\reference.png `
  --size 1024x1536
```

Optional responses-compatible mode:

```powershell
node scripts/flat_image_gen.mjs `
  --api-mode responses `
  --prompt "A red sticker icon on a transparent background" `
  --image-model gpt-image-1.5 `
  --background transparent `
  --output-format png
```

## Prompt guidance

Use a short structured spec when the user prompt is vague:

```text
Asset type: <icon/poster/hero/product image>
Primary request: <what to generate>
Reference images: <what each image contributes>
Style: <photo/illustration/3D/vector-like>
Composition: <framing and placement>
Lighting/mood: <lighting and mood>
Constraints: <must keep / must avoid>
```

Preserve detailed user prompts as-is when they are already specific.

## Files

- `scripts/flat_image_gen.mjs`: execution script
- `references/api.md`: request/response notes and environment variables
- `auth.json`: local API key store (gitignored)
- `.gitignore`: ignores secrets and generated assets