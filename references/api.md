# Flat Image API

This skill calls the API endpoint:

- `POST https://gateway.aimsg.uk/v1/responses`

Request style:

- outer model: usually `gpt-5.5`
- `input`: user message with `input_text` and optional `input_image`
- `tools`: one `image_generation` tool entry
- `tool_choice: { type: "image_generation" }`
- `stream: true`
- `store: false`

Authentication:

- `Authorization: Bearer <FLAT_API_KEY>`

Preferred environment variables:

- `FLAT_API_KEY`
- `FLAT_IMAGEGEN_BASE_URL`
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

- The script parses SSE events
- It accepts both `response.output_item.done` and `response.completed`
- Returned image payloads are base64-decoded and saved locally
