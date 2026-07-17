---
name: "flat-imagegen"
description: "Generate or edit raster images through the Flat Responses API. Use when the user wants image generation or reference-image editing through the project's API endpoint."
---

# Flat Imagegen

Generates or edits images by calling the project's Flat `POST /v1/responses` endpoint with the `image_generation` tool.

## When to use

- The user wants image generation routed through the Flat API
- The user wants reference-image editing through the same API path
- The user asks for a scriptable, repo-local image generation path

## When not to use

- The task is better handled by editing repo-native SVG, HTML/CSS, or canvas assets directly
- The user needs a different provider path that is unrelated to this API

## Top-level mode

This skill uses one execution path only:

- `scripts/flat_image_gen.mjs`

The script:
- calls the API endpoint directly
- sends `Responses + image_generation tool` payloads
- supports both text-to-image and reference-image editing
- parses SSE output and writes final images to disk

## Defaults

- Base URL: `https://gateway.aimsg.uk/v1`
- Default responses model: `gpt-5.5`
- Default image model: `gpt-image-2`
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

1. Decide whether the request is generate or edit.
2. Collect the prompt, output size, output format, count, and any reference images.
3. Check `auth.json` without displaying or logging its secret. If credentials are absent, request the key and store it only in the skill root or environment.
4. If the user names a destination path, pass `--output-dir` so the final asset lands in the workspace where they want it.
5. Run the bundled script.
6. Inspect the saved result if the task depends on visual correctness.
7. Report the final file path(s), image model used, and any revised prompt returned by the API.

## Privacy and provenance

- Reference images are Base64-encoded and uploaded to the third-party Flat endpoint. Do not send private, confidential, or unpublished assets without making that data transfer clear to the user.
- The source repository may not declare an open-source license. Treat the code as usable only within the permissions actually granted by its owner; do not assume unrestricted redistribution rights.

## Reference-image editing

- For one reference image, pass one `--image <path>`
- For multiple references, repeat `--image <path>` for each file
- The script converts local files to `data:` URLs and sends them as `input_image`

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

Transparent output:

```powershell
node scripts/flat_image_gen.mjs `
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
