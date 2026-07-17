# Flat Image API

This skill defaults to the classic Images API:

- `POST https://gateway.aimsg.uk/v1/images/generations`

Default request style:

- `model`: usually `gpt-image-2`
- `prompt`
- `size`
- `n`
- `response_format: "b64_json"`
- optional image-to-image fields: `image` / `extra_body.image`

Authentication:

- `Authorization: Bearer <FLAT_API_KEY>`

Optional compatibility mode:

- `POST https://gateway.aimsg.uk/v1/responses`
- outer model: usually `gpt-5.5`
- `tools`: one `image_generation` tool entry
- `tool_choice: { type: "image_generation" }`
- `stream: true`
- `store: false`

Use responses mode only when the gateway actually supports Responses + image tools. Many unofficial gateways only enable Images API for `gpt-image-2`.

Preferred environment variables:

- `FLAT_API_KEY`
- `FLAT_IMAGEGEN_BASE_URL`
- `FLAT_IMAGEGEN_API_MODE`
- `FLAT_IMAGEGEN_RESPONSES_MODEL`
- `FLAT_IMAGEGEN_IMAGE_MODEL`
- `FLAT_IMAGEGEN_IMAGE_PROMPT`
- `FLAT_IMAGEGEN_IMAGE_SIZE`
- `FLAT_IMAGEGEN_IMAGE_OUTPUT_FORMAT`
- `FLAT_IMAGEGEN_IMAGE_BACKGROUND`
- `FLAT_IMAGEGEN_IMAGE_QUALITY`
- `FLAT_IMAGEGEN_IMAGE_OUTPUT_COMPRESSION`
- `FLAT_IMAGEGEN_IMAGE_COUNT`
- `FLAT_IMAGEGEN_TIMEOUT_MS`
- `FLAT_IMAGEGEN_OUTPUT_DIR`

Output handling:

- Images mode parses JSON `data[].b64_json`
- Responses mode parses SSE events (`response.output_item.done` / `response.completed`)
- Returned image payloads are base64-decoded and saved locally