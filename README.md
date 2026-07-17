# flat-imagegen

适用于 **非官方模型网关** 的 Codex 生图 skill。  
通过 OpenAI 兼容的 `Responses + image_generation` 接口，把文生图 / 参考图编辑接到 `https://gateway.aimsg.uk/v1`。

适合这些场景：

- 你在用 Codex / 本地 agent，但图片能力不走官方 OpenAI 直连
- 你的模型服务是第三方 / 自建 / 网关转发的 OpenAI 兼容接口
- 你想要一个可脚本化、可落盘、可重复调用的本地生图 skill

## 简介

`flat-imagegen` 是一个最小可运行的 skill 包：

| 文件 | 作用 |
| --- | --- |
| `SKILL.md` | skill 入口：何时触发、默认参数、工作流 |
| `scripts/flat_image_gen.mjs` | 真正发请求、解析 SSE、保存图片的脚本 |
| `references/api.md` | API 请求形态与环境变量说明 |
| `auth.json` | 本地密钥（不提交） |
| `.gitignore` | 忽略密钥和生成结果 |

默认行为：

- Base URL：`https://gateway.aimsg.uk/v1`
- Responses 模型：`gpt-5.5`
- 图片模型：`gpt-image-2`
- 输出目录：`data/generated-images`
- 鉴权：`auth.json` 里的 `FLAT_API_KEY`，或环境变量 `FLAT_API_KEY`

## 安装

把整个目录放进 Codex / Hermes 的 skills 目录，例如：

```text
<skills-root>/flat-imagegen/
  SKILL.md
  scripts/flat_image_gen.mjs
  references/api.md
```

然后创建本地密钥：

```json
{
  "FLAT_API_KEY": "your_gateway_token"
}
```

也可以：

```powershell
$env:FLAT_API_KEY = "your_gateway_token"
```

验证脚本语法：

```powershell
node --check scripts/flat_image_gen.mjs
```

## 快速使用

### 文生图

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "A friendly robot mascot" `
  --size 1024x1024 `
  --output-format png
```

### 参考图编辑

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "Keep the subject, replace the background with a bright studio setup" `
  --image .\reference.png `
  --size 1024x1536
```

### 透明背景

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "A red sticker icon on a transparent background" `
  --image-model gpt-image-1.5 `
  --background transparent `
  --output-format png
```

### 指定输出目录

```powershell
node scripts/flat_image_gen.mjs `
  --prompt "poster concept" `
  --output-dir .\out\posters
```

## 为什么适合“非官方模型”

官方 Codex 内置生图路径通常绑定自家 provider。  
这个 skill 把生图能力拆成：

1. 本地 skill 触发
2. 脚本调用你配置的 OpenAI 兼容网关
3. 网关再转发到可用的图片模型

因此你可以：

- 换 base URL
- 换 responses / image model id
- 继续用 Codex 的 skill 工作流
- 不把业务绑死在单一官方账号路径上

常见自定义方式：

```powershell
$env:FLAT_IMAGEGEN_BASE_URL = "https://gateway.aimsg.uk/v1"
$env:FLAT_IMAGEGEN_RESPONSES_MODEL = "gpt-5.5"
$env:FLAT_IMAGEGEN_IMAGE_MODEL = "gpt-image-2"
```

或 CLI：

```powershell
node scripts/flat_image_gen.mjs `
  --base-url "https://gateway.aimsg.uk/v1" `
  --responses-model "gpt-5.5" `
  --image-model "gpt-image-2" `
  --prompt "..."
```

## 请求形态

脚本调用：

```http
POST /v1/responses
Authorization: Bearer <FLAT_API_KEY>
```

核心 payload 风格：

- `input`: `input_text` + 可选 `input_image`
- `tools`: `[{ "type": "image_generation", ... }]`
- `tool_choice`: `{ "type": "image_generation" }`
- `stream: true`
- `store: false`

返回的 SSE 图片结果会解码并写到本地。

## 环境变量

| 变量 | 含义 |
| --- | --- |
| `FLAT_API_KEY` | API key |
| `FLAT_IMAGEGEN_BASE_URL` | Responses base URL |
| `FLAT_IMAGEGEN_RESPONSES_MODEL` | 外层 responses 模型 |
| `FLAT_IMAGEGEN_IMAGE_MODEL` | 图片工具模型 |
| `FLAT_IMAGEGEN_IMAGE_PROMPT` | 默认 prompt |
| `FLAT_IMAGEGEN_IMAGE_SIZE` | 尺寸，如 `1024x1024` |
| `FLAT_IMAGEGEN_IMAGE_OUTPUT_FORMAT` | `png` / `jpeg` / `webp` |
| `FLAT_IMAGEGEN_IMAGE_BACKGROUND` | `transparent` / `opaque` / `auto` |
| `FLAT_IMAGEGEN_IMAGE_QUALITY` | 质量参数 |
| `FLAT_IMAGEGEN_IMAGE_COUNT` | 次数 |
| `FLAT_IMAGEGEN_TIMEOUT_MS` | 超时毫秒 |
| `FLAT_IMAGEGEN_OUTPUT_DIR` | 输出目录 |

## 安全注意

- 不要提交 `auth.json`
- 不要把 token 写进 skill 文档或脚本
- 参考图会 Base64 上传到第三方网关；私密素材请先确认可外传
- 生图结果默认落在本地 `data/generated-images/`

## 仓库状态

当前仓库已按 Flat / gateway 路径整理：

- skill 名：`flat-imagegen`
- 执行脚本：`scripts/flat_image_gen.mjs`
- 默认网关：`https://gateway.aimsg.uk/v1`

如果你只是想在 Codex 里用“非官方模型也能稳定生图”的 skill，放进 skills 目录、配好 `FLAT_API_KEY` 即可。